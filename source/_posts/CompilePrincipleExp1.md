---
title: CompilePrincipleExp1
date: 2022-11-01 22:53:58
tags: experiments
categories: CompilePrinciple
---

# 利用 FLEX 构造 C-Minus-f 词法分析器

### 写在实验前

* 本人本次实验环境为VMware Ubuntu20.04

* 实验项目先fork，再clone到本地

* 需要提前下载FLEX，BISON，LLVM。采用以下指令：

    ```shell
    sudo apt install flex
    sudo apt install bison
    sudo apt install llvm
    ```

* 记得push前删掉临时文件！！！

### 一、实验要求

* 根据 C-Minus-f 的词法补全`lexical_analyer.l`文件，完成词法分析器，能够输出识别出的`token`，`type` ,`line(刚出现的行数)`，`pos_start(该行开始位置)`，`pos_end(结束的位置,不包含)`。

    例如文本输入：

    ```c
    int a;
    ```

    则识别结果应为：

    ```shell
    int     280     1       2       5
    a       285     1       6       7
    ;       270     1       7       8
    ```

### 二、实验难点

* 识别注释的正则表达式编写

* FLEX 的使用

### 三、实验设计

先分析 C-Minus-f 的词法：

* 关键字 

  ```
  else if int return void while float
  ```

* 专用符号 

  ```
  + - * / < <= > >= == != = ; , ( ) [ ] { } /* */
  ```

* 标识符ID和整数NUM，通过下列正则表达式定义:  

  ```
  letter = a|...|z|A|...|Z
  digit = 0|...|9
  ID = letter+
  INTEGER = digit+
  FLOAT = (digit+. | digit*.digit+)
  ```

* 注释用`/*...*/`表示，可以超过一行。注释不能嵌套。 

  ```
  /*...*/
  ```

* 注：`[`,  `]`,  和 `[]` 是三种不同的token。`[]`用于声明数组类型，`[]`中间不得有空格。
  
  * `a[]`应被识别为两个token: `a`、`[]`
  
  * `a[1]`应被识别为四个token: `a`, `[`, `1`, `]`

不难发现与c语言的一些区别：首先是支持的关键字与操作并不全，其次标识符仅支持字母的组合，然后注释只接受`/*...*/`的形式，并不支持`//`。

本次实验只需要完成`lexical_analyzer.l`文件中的部分内容，通过查阅[FLEX的官方文档](https://www.cs.virginia.edu/~cr4bd/flex-manual/)可以了解以下信息：

* 用`%{`与`%}`括起来的部分用于声明变量、头文件，会直接复制到生成的lex.yy.c中；用`%%`与`%%`括起来的部分，用于编写规则；剩下的部分则是用户编写的代码。

* 在`%}`与`%%`之间的部分可以定义模式，~~但是本人实操一直报错，只好作罢~~

* 对于一条规则，应在一行内编写。

再找到其依赖的头文件`lexical_analyzer.h`，看到里面有定义的枚举 ~~(但其实在这个实验用得上的也没几个)~~ ，可以参照进行规则的编写。

设计：

* 对于普通的关键字与操作符，直接识别，然后先将pos_start移至上一个的pos_end，再令pos_end加上token的长度，最后return对应的枚举值即可。

* 对于int数，正则表达式为`[0-9]+`，后续操作同上。

* 对于float数，正则表达式为`[0-9]+\.|[0-9]*\.[0-9]+`，后续操作同上。

* 对于标识符，正则表达式为`[a-zA-Z]+`，后续操作同上。

* 对于换行符，应当重置pos_start与pos_end均为1，然后再使行数+1。

* 对于空白符，应当使pos_end+1，随后使pos_start=pos_end。

* 对于注释，由于不能嵌套，所以正则表达式为`\/\*([*]*(([^*/])+([/])*)*)*\*\/`。由于允许多行，所以需要再进行操作：遍历注释，遇到换行符时lines++,pos_end重置为1；遇到非换行符，仅pos_end++。

代码：

```c
%option noyywrap
%{
/*****************声明和选项设置  begin*****************/
#include <stdio.h>
#include <stdlib.h>

#include "lexical_analyzer.h"

int lines;
int pos_start;
int pos_end;

/*****************声明和选项设置  end*****************/

%}

%%

 /******************TODO*********************/
 /****请在此补全所有flex的模式与动作  start******/
 //STUDENT TO DO
 
\/\*([*]*(([^*/])+([/])*)*)*\*\/ {return COMMENT;}

\+ {pos_start=pos_end; pos_end++; return ADD;}

\- {pos_start=pos_end; pos_end++; return SUB;}

\* {pos_start=pos_end; pos_end++; return MUL;}

\/ {pos_start=pos_end; pos_end++; return DIV;}

\< {pos_start=pos_end; pos_end++; return LT;}

\<\= {pos_start=pos_end; pos_end+=2; return LTE;}

\> {pos_start=pos_end; pos_end++; return GT;}

\>\= {pos_start=pos_end; pos_end+=2; return GTE;}

\=\= {pos_start=pos_end; pos_end+=2; return EQ;}

\!\= {pos_start=pos_end; pos_end+=2; return NEQ;}

\= {pos_start=pos_end; pos_end++; return ASSIN;}

\; {pos_start=pos_end; pos_end++; return SEMICOLON;}

\, {pos_start=pos_end; pos_end++; return COMMA;}

\( {pos_start=pos_end; pos_end++; return LPARENTHESE;}

\) {pos_start=pos_end; pos_end++; return RPARENTHESE;}

\[ {pos_start=pos_end; pos_end++; return LBRACKET;}

\] {pos_start=pos_end; pos_end++; return RBRACKET;}

\{ {pos_start=pos_end; pos_end++; return LBRACE;}

\} {pos_start=pos_end; pos_end++; return RBRACE;}

else {pos_start=pos_end; pos_end+=4; return ELSE;}

if {pos_start=pos_end; pos_end+=2; return IF;}

int {pos_start=pos_end; pos_end+=3; return INT;}

float {pos_start=pos_end; pos_end+=5; return FLOAT;}

return {pos_start=pos_end; pos_end+=6; return RETURN;}

void {pos_start=pos_end; pos_end+=4; return VOID;}

while {pos_start=pos_end; pos_end+=5; return WHILE;}

[ \f\r\t\v] {return BLANK;}

[\n] {return EOL;}

[a-zA-Z]+ {pos_start=pos_end; pos_end+=strlen(yytext); return IDENTIFIER;}

[a-zA-Z] {pos_start=pos_end; pos_end+=strlen(yytext); return LETTER;}

[0-9]+ {pos_start=pos_end; pos_end+=strlen(yytext); return INTEGER;}

\[\] {pos_start=pos_end; pos_end+=2; return ARRAY;}

[0-9]+\.|[0-9]*\.[0-9]+ {pos_start=pos_end; pos_end+=strlen(yytext); return FLOATPOINT;}

. {return ERROR;}

 /****请在此补全所有flex的模式与动作  end******/
%%

/****************C代码 start*************/

/// \brief analysize a *.cminus file
///
/// \param input_file, 需要分析的文件路径
/// \param token stream, Token_Node结构体数组，用于存储分析结果，具体定义参考lexical_analyer.h

void analyzer(char* input_file, Token_Node* token_stream){
    lines = 1;
    pos_start = 1;
    pos_end = 1;
    if(!(yyin = fopen(input_file,"r"))){
        printf("[ERR] No input file\n");
        exit(1);
    }
    printf("[START]: Read from: %s\n", input_file);

    int token;
    int index = 0;

    while(token = yylex()){
        switch(token){
            case COMMENT:
                //STUDENT TO DO
                for(int i=0;i<strlen(yytext);i++){
                	pos_end++;
                	if(yytext[i]=='\n'){
                		lines++;
                		pos_end=1;
                	}
                }
                break;
            case BLANK:
                //STUDENT TO DO
                pos_end++; 
                pos_start=pos_end;
                break;
            case EOL:
                //STUDENT TO DO
                pos_start = 1;
                pos_end = 1;
                lines++;
                break;
            case ERROR:
                printf("[ERR]: unable to analysize %s at %d line, from %d to %d\n", yytext, lines, pos_start, pos_end);
            default :
                if (token == ERROR){
                    sprintf(token_stream[index].text, "[ERR]: unable to analysize %s at %d line, from %d to %d", yytext, lines, pos_start, pos_end);
                } else {
                    strcpy(token_stream[index].text, yytext);
                }
                token_stream[index].token = token;
                token_stream[index].lines = lines;
                token_stream[index].pos_start = pos_start;
                token_stream[index].pos_end = pos_end;
                index++;
                if (index >= MAX_NUM_TOKEN_NODE){
                    printf("%s has too many tokens (> %d)", input_file, MAX_NUM_TOKEN_NODE);
                    exit(1);
                }
        }
    }
    printf("[END]: Analysis completed.\n");
    return;
}

/****************C代码 end*************/
```

### 四、实验结果验证

* 首先验证实验提供的测试用例：

    测试结果：

    ![](/img/CPexp1/test1.png)

* 其次验证自己编写的测试用例：

    ```c
    /*这是一段测试代码
    这是一段注释
    これはテストコードです
    これは注釈です
    테스트 코드입니다
    이것은 주석입니다.*/

    void mytest(int a,float b){
        int sum=0;
        if(a>=(int)b){
        sum+=10;
        }
        else if(a<=(int)b){
        sum+=4;}
        sum/=2;
        return sum;
    }

    int main(){
        int ans=mytest(2,3.132);
        int arr[];
        int brr[6]
        while(ans--){
        if(ans==0){
        brr[ans]=ans;
        }
        else if(ans!=1){
        brr[ans]=ans-1;
        }
        }
        return 0;
    }
    ```

    测试结果：

    ![](/img/CPexp1/test2.png)

    ```
    void	283	8	1	5
    mytest	285	8	6	12
    (	272	8	12	13
    int	280	8	13	16
    a	285	8	17	18
    ,	271	8	18	19
    float	281	8	19	24
    b	285	8	25	26
    )	273	8	26	27
    {	276	8	27	28
    int	280	9	2	5
    sum	285	9	6	9
    =	269	9	9	10
    0	286	9	10	11
    ;	270	9	11	12
    if	279	10	2	4
    (	272	10	4	5
    a	285	10	5	6
    >=	266	10	6	8
    (	272	10	8	9
    int	280	10	9	12
    )	273	10	12	13
    b	285	10	13	14
    )	273	10	14	15
    {	276	10	15	16
    sum	285	11	2	5
    +	259	11	5	6
    =	269	11	6	7
    10	286	11	7	9
    ;	270	11	9	10
    }	277	12	2	3
    else	278	13	2	6
    if	279	13	7	9
    (	272	13	9	10
    a	285	13	10	11
    <=	264	13	11	13
    (	272	13	13	14
    int	280	13	14	17
    )	273	13	17	18
    b	285	13	18	19
    )	273	13	19	20
    {	276	13	20	21
    sum	285	14	2	5
    +	259	14	5	6
    =	269	14	6	7
    4	286	14	7	8
    ;	270	14	8	9
    }	277	14	9	10
    sum	285	15	2	5
    /	262	15	5	6
    =	269	15	6	7
    2	286	15	7	8
    ;	270	15	8	9
    return	282	16	2	8
    sum	285	16	9	12
    ;	270	16	12	13
    }	277	17	1	2
    int	280	19	1	4
    main	285	19	5	9
    (	272	19	9	10
    )	273	19	10	11
    {	276	19	11	12
    int	280	20	2	5
    ans	285	20	6	9
    =	269	20	9	10
    mytest	285	20	10	16
    (	272	20	16	17
    2	286	20	17	18
    ,	271	20	18	19
    3.132	287	20	19	24
    )	273	20	24	25
    ;	270	20	25	26
    int	280	21	2	5
    arr	285	21	6	9
    []	288	21	9	11
    ;	270	21	11	12
    int	280	22	2	5
    brr	285	22	6	9
    [	274	22	9	10
    6	286	22	10	11
    ]	275	22	11	12
    while	284	23	2	7
    (	272	23	7	8
    ans	285	23	8	11
    -	260	23	11	12
    -	260	23	12	13
    )	273	23	13	14
    {	276	23	14	15
    if	279	24	2	4
    (	272	24	4	5
    ans	285	24	5	8
    ==	267	24	8	10
    0	286	24	10	11
    )	273	24	11	12
    {	276	24	12	13
    brr	285	25	9	12
    [	274	25	12	13
    ans	285	25	13	16
    ]	275	25	16	17
    =	269	25	17	18
    ans	285	25	18	21
    ;	270	25	21	22
    }	277	26	9	10
    else	278	27	9	13
    if	279	27	14	16
    (	272	27	16	17
    ans	285	27	17	20
    !=	268	27	20	22
    1	286	27	22	23
    )	273	27	23	24
    {	276	27	24	25
    brr	285	28	9	12
    [	274	28	12	13
    ans	285	28	13	16
    ]	275	28	16	17
    =	269	28	17	18
    ans	285	28	18	21
    -	260	28	21	22
    1	286	28	22	23
    ;	270	28	23	24
    }	277	29	9	10
    }	277	30	2	3
    return	282	31	2	8
    0	286	31	9	10
    ;	270	31	10	11
    }	277	32	1	2
    ```