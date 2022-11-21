---
title: Compile Principle Exp2
date: 2022-11-18 20:52:06
tags: experiments
categories: CompilePrinciple
---

# 利用 BISON 构造 C-Minus-f 语法分析器

### 一、实验要求

本次实验需要各位同学首先将自己的 lab1 的词法部分复制到 `/src/parser` 目录的 [lexical\_analyzer.l](../../src/parser/lexical\_analyzer.l)并合理修改相应部分，然后根据 `cminus-f` 的语法补全 [syntax\_analyer.y](../../src/parser/syntax_analyzer.y) 文件，完成语法分析器，要求最终能够输出解析树。如：

输入：

```c
int bar;
float foo(void) { return 1.0; }
```

则 `parser` 将输出如下解析树：

```
>--+ program
|  >--+ declaration-list
|  |  >--+ declaration-list
|  |  |  >--+ declaration
|  |  |  |  >--+ var-declaration
|  |  |  |  |  >--+ type-specifier
|  |  |  |  |  |  >--* int
|  |  |  |  |  >--* bar
|  |  |  |  |  >--* ;
|  |  >--+ declaration
|  |  |  >--+ fun-declaration
|  |  |  |  >--+ type-specifier
|  |  |  |  |  >--* float
|  |  |  |  >--* foo
|  |  |  |  >--* (
|  |  |  |  >--+ params
|  |  |  |  |  >--* void
|  |  |  |  >--* )
|  |  |  |  >--+ compound-stmt
|  |  |  |  |  >--* {
|  |  |  |  |  >--+ local-declarations
|  |  |  |  |  |  >--* epsilon
|  |  |  |  |  >--+ statement-list
|  |  |  |  |  |  >--+ statement-list
|  |  |  |  |  |  |  >--* epsilon
|  |  |  |  |  |  >--+ statement
|  |  |  |  |  |  |  >--+ return-stmt
|  |  |  |  |  |  |  |  >--* return
|  |  |  |  |  |  |  |  >--+ expression
|  |  |  |  |  |  |  |  |  >--+ simple-expression
|  |  |  |  |  |  |  |  |  |  >--+ additive-expression
|  |  |  |  |  |  |  |  |  |  |  >--+ term
|  |  |  |  |  |  |  |  |  |  |  |  >--+ factor
|  |  |  |  |  |  |  |  |  |  |  |  |  >--+ float
|  |  |  |  |  |  |  |  |  |  |  |  |  |  >--* 1.0
|  |  |  |  |  |  |  |  >--* ;
|  |  |  |  |  >--* }
```

* 请注意，上述解析树含有每个解析规则的所有子成分，包括诸如 `;` `{` `}` 这样的符号，请在编写规则时务必不要忘了它们。

### 二、实验难点

* yylval的作用

* 树的构建以及源码理解

### 三、实验设计

bison的.y文件结构与flex的.l文件结构类似，分为一下部分：

```
%{
  /*这部分中的代码会照抄到.c文件中，可以用于写一些头文件、全局变量、函数定义等*/
%}

  /*这部分用于定义token、symbol、union、start等值，也可以在这里定义算符优先级*/

  %union{
    type member;
  }
  %token <type> token1 token2 ...
  %symbol <type> symbol1 symbol2 ...

  %left sign1 sign2 ... /*左结合*/
  %right sign1 sign2 ... /*右结合*/
  /*越靠下优先级越高*/

%%
  /*这部分用于定义rule*/
  symbol:
  tokens symbols   { action; }
| tokens symbols   { action; }
;
%%

  /*这部分用于定义函数*/

```

关于%union：是token、symbol取值类型的集合，可能不同的token、symbol会使用不同的类型，那么就可以在%token \<type> 设置其类型。
在编译时，其会作为.c文件中的YYSTYPE，在本实验中，只用输出代码的语法分析树，所以类型只有一种`syntax_tree_node*`,是语法树的节点。关于语法树的分析放在文末思考题中。

实验指导文档中已经给出了 C-Minus-f 的语法：

1. $`\text{program} \rightarrow \text{declaration-list}`$
2. $`\text{declaration-list} \rightarrow \text{declaration-list}\ \text{declaration}\ |\ \text{declaration}`$
3. $`\text{declaration} \rightarrow \text{var-declaration}\ |\ \text{fun-declaration}`$
4. $`\text{var-declaration}\ \rightarrow \text{type-specifier}\ \textbf{ID}\ \textbf{;}\ |\ \text{type-specifier}\ \textbf{ID}\ \textbf{[}\ \textbf{INTEGER}\ \textbf{]}\ \textbf{;}`$
5. $`\text{type-specifier} \rightarrow \textbf{int}\ |\ \textbf{float}\ |\ \textbf{void}`$
6. $`\text{fun-declaration} \rightarrow \text{type-specifier}\ \textbf{ID}\ \textbf{(}\ \text{params}\ \textbf{)}\ \text{compound-stmt}`$
7. $`\text{params} \rightarrow \text{param-list}\ |\ \textbf{void}`$
8. $`\text{param-list} \rightarrow \text{param-list}\ ,\ \text{param}\ |\ \text{param}`$
9. $`\text{param} \rightarrow \text{type-specifier}\ \textbf{ID}\ |\ \text{type-specifier}\ \textbf{ID}\ \textbf{[]}`$
10. $`\text{compound-stmt} \rightarrow \textbf{\{}\ \text{local-declarations}\ \text{statement-list} \textbf{\}}`$
11. $`\text{local-declarations} \rightarrow \text{local-declarations var-declaration}\ |\ \text{empty}`$
12. $`\text{statement-list} \rightarrow \text{statement-list}\ \text{statement}\ |\ \text{empty}`$
13. $`\begin{aligned}\text{statement} \rightarrow\ &\text{expression-stmt}\\ &|\ \text{compound-stmt}\\ &|\ \text{selection-stmt}\\ &|\ \text{iteration-stmt}\\ &|\ \text{return-stmt}\end{aligned}`$
14. $`\text{expression-stmt} \rightarrow \text{expression}\ \textbf{;}\ |\ \textbf{;}`$
15. $`\begin{aligned}\text{selection-stmt} \rightarrow\ &\textbf{if}\ \textbf{(}\ \text{expression}\ \textbf{)}\ \text{statement}\\ &|\ \textbf{if}\ \textbf{(}\ \text{expression}\ \textbf{)}\ \text{statement}\ \textbf{else}\ \text{statement}\end{aligned}`$
16. $`\text{iteration-stmt} \rightarrow \textbf{while}\ \textbf{(}\ \text{expression}\ \textbf{)}\ \text{statement}`$
17. $`\text{return-stmt} \rightarrow \textbf{return}\ \textbf{;}\ |\ \textbf{return}\ \text{expression}\ \textbf{;}`$
18. $`\text{expression} \rightarrow \text{var}\ \textbf{=}\ \text{expression}\ |\ \text{simple-expression}`$
19. $`\text{var} \rightarrow \textbf{ID}\ |\ \textbf{ID}\ \textbf{[}\ \text{expression} \textbf{]}`$
20. $`\text{simple-expression} \rightarrow \text{additive-expression}\ \text{relop}\ \text{additive-expression}\ |\ \text{additive-expression}`$
21. $`\text{relop}\ \rightarrow \textbf{<=}\ |\ \textbf{<}\ |\ \textbf{>}\ |\ \textbf{>=}\ |\ \textbf{==}\ |\ \textbf{!=}`$
22. $`\text{additive-expression} \rightarrow \text{additive-expression}\ \text{addop}\ \text{term}\ |\ \text{term}`$
23. $`\text{addop} \rightarrow \textbf{+}\ |\ \textbf{-}`$
24. $`\text{term} \rightarrow \text{term}\ \text{mulop}\ \text{factor}\ |\ \text{factor}`$
25. $`\text{mulop} \rightarrow \textbf{*}\ |\ \textbf{/}`$
26. $`\text{factor} \rightarrow \textbf{(}\ \text{expression}\ \textbf{)}\ |\ \text{var}\ |\ \text{call}\ |\ \text{integer}\ |\ \text{float}`$
27. $`\text{integer} \rightarrow \textbf{INTEGER}`$
28. $`\text{float} \rightarrow \textbf{FLOATPOINT}`$
29. $`\text{call} \rightarrow \textbf{ID}\ \textbf{(}\ \text{args} \textbf{)}`$
30. $`\text{args} \rightarrow \text{arg-list}\ |\ \text{empty}`$
31. $`\text{arg-list} \rightarrow \text{arg-list}\ \textbf{,}\ \text{expression}\ |\ \text{expression}`$

有个地方需要注意：

* 在《编译原理与实践》第九章附录给出的 C-Minus-f 的语法与上文有些许区别，除了float的加入之外，定义函数的规则也不同，本实验必须要有函数的实现内容(即花括号以及之间的内容)，而书上则允许不用该部分。

还是以本次实验的指导文档为准。

根据语法规则，定义了如下token,symbol,rule:

```c
%token <node> IDENTIFIER SEMICOLON LBRACKET INTEGER FLOATPOINT RBRACKET LPARENTHESE RPARENTHESE VOID COMMA LBRACE RBRACE IF ELSE WHILE RETURN ASSIN LTE LT GT GTE EQ NEQ ADD SUB MUL DIV ARRAY INT FLOAT
%type <node> program declaration-list declaration var-declaration fun-declaration params param-list param compound-stmt local-declarations statement-list statement expression-stmt selection-stmt iteration-stmt return-stmt expression var simple-expression relop additive-expression addop term mulop factor integer float call args arg-list type-specifier
%start program

%%

program : declaration-list { $$ = node("program", 1, $1); gt->root = $$; };

declaration-list : declaration-list declaration { $$ = node("declaration-list",2,$1,$2); } | declaration { $$ = node("declaration-list",1,$1); };

declaration : var-declaration { $$ = node("declaration",1,$1); } | fun-declaration { $$ = node("declaration",1,$1); };

var-declaration : type-specifier IDENTIFIER SEMICOLON { $$ = node("var-declaration",3,$1,$2,$3); } 
| type-specifier IDENTIFIER LBRACKET INTEGER RBRACKET SEMICOLON { $$ = node("var-declaration",6,$1,$2,$3,$4,$5,$6); };

type-specifier : INT { $$ = node("type-specifier",1,$1); } | FLOAT { $$ = node("type-specifier",1,$1); } | VOID { $$ = node("type-specifier",1,$1); };

fun-declaration : type-specifier IDENTIFIER LPARENTHESE params RPARENTHESE compound-stmt {  $$ = node("fun-declaration",6,$1,$2,$3,$4,$5,$6);  };

params : param-list { $$ = node("params", 1, $1); } | VOID { $$ = node("params", 1, $1); };

param-list : param-list COMMA param { $$ = node("param-list",3,$1,$2,$3); } | param { $$ = node("param-list", 1, $1); };

param : type-specifier IDENTIFIER { $$ = node("param",2,$1,$2); } | type-specifier IDENTIFIER ARRAY { $$ = node("param",3,$1,$2,$3); };

compound-stmt : LBRACE local-declarations statement-list RBRACE { $$ = node("compound-stmt",4,$1,$2,$3,$4); };

local-declarations : local-declarations var-declaration { $$ = node("local-declarations",2,$1,$2); } |  { $$ = node("local-declarations",0); };

statement-list : statement-list statement { $$ = node("statement-list",2,$1,$2); } |  { $$ = node("statement-list",0); };

statement : expression-stmt { $$ = node("statement", 1, $1); }
| compound-stmt { $$ = node("statement", 1, $1); }
| selection-stmt { $$ = node("statement", 1, $1); }
| iteration-stmt { $$ = node("statement", 1, $1); }
| return-stmt { $$ = node("statement", 1, $1); };

expression-stmt : expression SEMICOLON { $$ = node("expression-stmt",2,$1,$2); } | SEMICOLON { $$ = node("expression-stmt", 1, $1); };

selection-stmt : IF LPARENTHESE expression RPARENTHESE statement { $$ = node("selection-stmt",5,$1,$2,$3,$4,$5); } | IF LPARENTHESE expression RPARENTHESE statement ELSE statement { $$ = node("selection-stmt",7,$1,$2,$3,$4,$5,$6,$7); };

iteration-stmt : WHILE LPARENTHESE expression RPARENTHESE statement { $$ = node("iteration-stmt",5,$1,$2,$3,$4,$5); };

return-stmt : RETURN SEMICOLON { $$ = node("return-stmt",2,$1,$2); } | RETURN expression SEMICOLON { $$ = node("return-stmt",3,$1,$2,$3); };

expression : var ASSIN expression { $$ = node("expression",3,$1,$2,$3); } 
| simple-expression { $$ = node("expression", 1, $1); };

var : IDENTIFIER { $$ = node("var", 1, $1); } | IDENTIFIER LBRACKET expression RBRACKET { $$ = node("var",4,$1,$2,$3,$4); };

simple-expression : additive-expression relop additive-expression { $$ = node("simple-expression",3,$1,$2,$3); } | additive-expression { $$ = node("simple-expression", 1, $1); };

relop : LTE { $$ = node("relop", 1, $1); } | LT { $$ = node("relop", 1, $1); } | GT { $$ = node("relop", 1, $1); } | GTE { $$ = node("relop", 1, $1); } | EQ { $$ = node("relop", 1, $1); } | NEQ { $$ = node("relop", 1, $1); };

additive-expression : additive-expression addop term { $$ = node("additive-expression",3,$1,$2,$3); } | term { $$ = node("additive-expression", 1, $1); };

addop : ADD { $$ = node("addop", 1, $1); } | SUB { $$ = node("addop", 1, $1); };

term : term mulop factor { $$ = node("term",3,$1,$2,$3); } | factor { $$ = node("term", 1, $1); };

mulop : MUL { $$ = node("mulop", 1, $1); } | DIV { $$ = node("mulop", 1, $1); };

factor : LPARENTHESE expression RPARENTHESE { $$ = node("factor",3,$1,$2,$3); } | var { $$ = node("factor", 1, $1); } | call { $$ = node("factor", 1, $1); } | integer { $$ = node("factor", 1, $1); } | float { $$ = node("factor", 1, $1); };

integer : INTEGER { $$ = node("integer", 1, $1); };

float : FLOATPOINT { $$ = node("float", 1, $1); };

call : IDENTIFIER LPARENTHESE args RPARENTHESE { $$ = node("call",4,$1,$2,$3,$4); };

args : arg-list { $$ = node("args", 1, $1); } |  { $$ = node("args",0); };

arg-list : arg-list COMMA expression { $$ = node("arg-list",3,$1,$2,$3); } | expression { $$ = node("arg-list", 1, $1); };
```

其中，文法的开始符号`program`放在`%start`后，token一般大写，symbol一般小写。

同时，`lexical_analyzer.l`的词法也需进行修改，对于在`syntax_analyzer.y`中说明了的token，需要返回枚举值，并用pass_node(yytext)入栈(关于pass_node的一会再说)；而对于未说明的如`COMMENT`,`BLANK`,`EOF`,`.`则无需返回枚举值，也不需要入栈，只需进行pos_start,pos_end与line值的更改。例：

```c
\+ {pos_start=pos_end; pos_end++; pass_node(yytext); return ADD;}

[ \f\r\t\v] {pos_end++; pos_start=pos_end;}//BLANK
```

### 四、实验结果验证

#### 实验测试用例的结果：

![easy](/img/CPexp2/easy.png)

![normal](/img/CPexp2/normal.png)

#### 自己编写的测试用例：

```c
int a[10];
int b[10];
int func(int a,int b){
return a+b;
}


int main(void){

int i;

i=10;

while(i!=0){
	i=i-1;
	if(a[i]==0)
		a[i]=func(a[i],b[i]);
	if(b[i]!=0)
		b[i]=0;
	else
		b[i]=1;
	
	if(a[i]>=b[i]){
		a[i]=a[i]+b[i]*a[i];
		if(a[i]<=a[i+1])
			a[i]=a[i+1];
	}
	else{
		a[i]=b[i];
	}
	
}

}
```

bison采用的是LALR()文法，可能会导致悬挂else的问题，即有多个if时，else不知道与哪个if配对。解决方法是else与其最近的if配对，我编写的测试用例主要验证了这个问题。

看语法分析树的相关部分：(if中间的分析树省略)

```
|  |  |  |  |  |  |  |  |  |  |  |  |  >--+ statement
|  |  |  |  |  |  |  |  |  |  |  |  |  |  >--+ selection-stmt
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  >--* if
.
.
.
|  |  |  |  |  |  |  |  |  |  |  |  >--+ statement
|  |  |  |  |  |  |  |  |  |  |  |  |  >--+ selection-stmt
|  |  |  |  |  |  |  |  |  |  |  |  |  |  >--* if
.
.
.
|  |  |  |  |  |  |  |  |  |  |  |  |  |  >--* else
|  |  |  |  |  |  |  |  |  |  |  |  |  |  >--+ statement
```

可以看到第一部分的else是与第二个if配对的。

```
|  |  |  |  |  |  |  |  |  |  |  >--+ statement
|  |  |  |  |  |  |  |  |  |  |  |  >--+ selection-stmt
|  |  |  |  |  |  |  |  |  |  |  |  |  >--* if
.
.
.
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  >--+ statement
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  >--+ selection-stmt
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  >--* if
.
.
.
|  |  |  |  |  |  |  |  |  |  |  |  |  >--* else
|  |  |  |  |  |  |  |  |  |  |  |  |  >--+ statement
```

可以看到第二部分的else是与第一个if配对的。

#### 思考题
1.在1.3样例代码中存在左递归文法，为什么 `bison` 可以处理？（提示：不用研究`bison`内部运作机制，在下面知识介绍中有提到 `bison` 的一种属性，请结合课内知识思考）

因为bison使用的是LALR(1)文法，可以处理左递归文法。

2.请在代码层面上简述下 `yylval` 是怎么完成协同工作的。（提示：无需研究原理，只分析维护了什么数据结构，该数据结构是怎么和`$1`、`$2`等联系起来？）

* `yylval`是一个全局变量，在多个文件中都能找到其声明，其类型为`YYSTYPE`，就是`%union`。其实其存放的就是yylex读取到的token的语义值，在本实验中只有node这一种。

* 首先，`yylex`获取下一个token，通过`pass_node`函数将其传给`yylval`，这时，parser就可以使用该值。

* 在`yybackup`函数中可以找到`yylval`的踪影，此函数用于判断获取到一个token时该执行的动作---是移入又或是规约。其中有一段关键代码：

```c
/* Shift the lookahead token.  */
  YY_SYMBOL_PRINT ("Shifting", yytoken, &yylval, &yylloc);
  yystate = yyn;
  YY_IGNORE_MAYBE_UNINITIALIZED_BEGIN
  *++yyvsp = yylval;
  YY_IGNORE_MAYBE_UNINITIALIZED_END
```

`yyn`是规则对应的枚举值，而`yyvsp`则是一个指针，指向当前使用的token值栈`yyvsa`的尾端，`*++yyvsp = yylval;`则意味着将`yylval`入栈(移入)。

随后再看其是如何与`$1,$2`等联系起来的：

在处理规约的函数`yyreduce`中可以看到我们编写的规则：

```c
yylen = yyr2[yyn];

yyval = yyvsp[1-yylen];

YY_REDUCE_PRINT (yyn);
switch (yyn)
    {
  case 2:
#line 44 "syntax_analyzer.y"
                           { (yyval.node) = node("program", 1, (yyvsp[0].node)); gt->root = (yyval.node); }
#line 1438 "/home/liaonux/CompileExps/cminus_compiler-2022-fall/build/src/parser/syntax_analyzer.c"
    break;

  case 3:
#line 46 "syntax_analyzer.y"
                                                { (yyval.node) = node("declaration-list",2,(yyvsp[-1].node),(yyvsp[0].node)); }
#line 1444 "/home/liaonux/CompileExps/cminus_compiler-2022-fall/build/src/parser/syntax_analyzer.c"
	.
	.
	.
	}
.
.
.
YYPOPSTACK (yylen);
yylen = 0;
YY_STACK_PRINT (yyss, yyssp);

*++yyvsp = yyval;
```

`yylen`是需要规约的规则长度，`yyval`是规约后放入规则左值的位置，`yyvsp`是当前token值栈的尾端。我们可以看到，若选择一个右值长度为2 token的规则规约，那么`$1,$2`对应的就是`yyvsp[-1].node,yyvsp[0].node`，随后将左值保存在`yyval`。最后将栈内`yylen`长度个数的tokens弹出，将`yyval`放入。

3.请尝试使用1.3样例代码运行除法运算除数为0的例子（测试case中有）看下是否可以通过，如果不，为什么我们在case中把该例子认为是合法的？（请从语法与语义上简单思考）

可以通过。因为是上下文无关文法，而乘除规则中我们是直接使用的`INTEGER [0-9]+`，所以是合法的。

4.能否尝试修改下1.3计算器文法，使得它支持除数0规避功能。

修改部分内容如下：

```c
#include <limits.h>

line
: expr RET
{
	if($1 != INT_MAX)
    printf(" = %f\n", $1);
}

term
: factor
{
    $$ = $1;
}
| term MULOP factor
{
    switch ($2) {
    case '*': $$ = $1 * $3; break;
    case '/': if($3 == 0) {yyerror("divisor can't be zero."); $$ = INT_MAX; break;} $$ = $1 / $3; break;
    }
}
```

添加`limits.h`头文件，在匹配`term MULOP factor`规则时，若符号为除号，则将`$$`设为`INT_MAX`并报错，最后在匹配`expr RET`规则时若`expr`值为`INT_MAX`则不输出，成功规避了除数0。

![](/img/CPexp2/calc.png)

### 六、实验反馈：

本次实验进行了关于语法规则的编写，本人也通过阅读源码了解了LALR(1)文法的代码实现的一些细节，以及词法分析器与语法分析器的联动，巩固了课程知识。

源码的阅读还是很有意思的！但是有一个小小的问题：在`syntax_analyzer.y`能`make`通过之前都是无法在build文件夹中找到`syntax_analyzer.c`的，`README.md`中说先编译1.3的代码就能找到，其实是在1.3代码的文件夹中的`calc.tab.c`文件...



附：一些`syntax_analyzer.c`中变量的意义
```
yyssa->state stack   yyss,yyssp->pointers(start,end of stack used)
yyvsa->value stack   yyvs,yyvsp->pointers(start,end of stack used)

yylval->token semantic value
yychar->a token from lexer

yyn->rule enum
yylen->规约的规则的右值的token数。

yytranslate->yylex返回的token的对应值。

终结符与非终结符均放在了一个数组中：yytname，从0-YYNTOKENS为终结符，之后的是非终结符。

yyrline->rules对应的代码位置。

yypact->yytable的索引。

yytable->在对应状态应该做什么。若为正数，将该token移进；若为负数，按-yytable值对应的规则规约。

yyr1[yyn]->yyn对应规则的左值的token对应的值

yyr2[yyn]->yyn对应规则的右值的token个数
```