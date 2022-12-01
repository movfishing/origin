---
title: Compile Principle exp3
date: 2022-11-30 14:17:39
tags: experiments
categories: CompilePrinciple
---

# lab3 实验报告

### 一、LLVM IR部分

助教提供了四个简单的c程序，分别是tests/lab3/c_cases/目录下的assign.c、fun.c、if.c和while.c.你需要在tests/lab3/stu_ll/目录中，手工完成自己的assign_hand.ll、fun_hand.ll、if_handf.ll和while_hand.ll，以实现与上述四个C程序相同的逻辑功能.你需要添加必要的注释..ll文件的注释是以";"开头的。

与汇编语言类似，有区别的是可以有无限个寄存器，但每个寄存器只能进行一次赋值操作。

`assign_hand.ll`:

```
define dso_local i32 @main() #0 {
  %1 = alloca i32, align 4
  %2 = alloca [10 x i32], align 4						;define int a[10]
  store i32 0, i32* %1, align 4
  %3 = getelementptr inbounds [10 x i32], [10 x i32]* %2, i64 0, i64 0	;pointer of a[0]
  store i32 10, i32* %3
  %4 = load i32, i32* %3, align 4
  %5 = mul nsw i32 %4, 2
  %6 = getelementptr inbounds [10 x i32], [10 x i32]* %2, i64 0, i64 1	;pointer of a[1]
  store i32 %5, i32* %6
  ret i32 %5
}
```

需要注意的是此处的getelementptr语句并不能像`gcd_array.ll`一样嵌入到store语句中，会报错。

inbounds参数是越界检查，若你的指针访问越界了，llvm不会终止程序，而是会返回一个`poison value`。

`fun_hand.ll`:

```
define dso_local i32 @callee(i32 %0) #0 {
  %2 = mul i32 %0, 2
  ret i32 %2
}

define dso_local i32 @main() #0 {
  %1 = call i32 @callee(i32 110)
  ret i32 %1
}
```

`if_hand.ll`:

```
define dso_local i32 @main() #0 {
  %1 = alloca float, align 4
  store float 0x40163851E0000000, float* %1, align 4	;float a = 5.555
  %2 = load float , float* %1, align 4
  %3 = fcmp ugt float %2, 1.0				;if judge
  br i1 %3, label %4, label %5
  
4:							;if true
  ret i32 233

5:
  ret i32 0
}
```

这里面有一个值得注意的问题：float的表示。在llvm中，可以使用三种形式来表示float:

* 基本的形式 如：1.5

* 标准形式 如：1.5e+0

* double类型的IEEE 754标准的十六进制形式(64bits) 如上所示

那么如何判断该用什么形式呢？很简单，二进制不能精确表示的小数，就必须要用最后一种形式。

`while_hand.ll`:

```
define dso_local i32 @main() #0 {
  %1 = alloca i32, align 4		;a
  %2 = alloca i32, align 4		;i	
  store i32 10, i32* %1, align 4
  store i32 0, i32* %2, align 4
  br label %3				
  
3:					;while judge
  %4 = load i32, i32* %2, align 4
  %5 = icmp slt i32 %4, 10
  br i1 %5, label %6, label %11
  
6:					;while body
  %7 = load i32, i32* %2, align 4
  %8 = add i32 %7, 1
  store i32 %8, i32* %2
  %9 = load i32, i32* %1, align 4
  %10 = add i32 %8, %9
  store i32 %10, i32* %1
  br label %3

11:
  %12 = load i32, i32* %1, align 4
  ret i32 %12
}
```

运行结果：

![](/img/CPexp3/result1.png)

### 二、Light IR部分

你需要在tests/lab3/stu_cpp/目录中，编写assign_generator.cpp、fun_generator.cpp、if_generator.cpp和while_generator.cpp，以生成与1.3节的四个C程序相同逻辑功能的.ll文件。

我们需要编写的.cpp代码在make后会生成对应的可执行文件，运行可执行文件就会输出相应的.ll代码，我们要保证与上一部分生成的.ll代码功能相同。我通过输出重定向将运行可执行文件输出的代码写入了对应的.ll文件中，以便运行验证结果。

其实本部分也是依葫芦画瓢，参考试验提供的tests/lab3/ta_gcd/gcd_array_generator.cpp就可以写出需要的代码。

assign_generator.cpp:

```c++
int main() {
  auto module = new Module("assign");
  auto builder = new IRBuilder(nullptr, module);
  Type *Int32Type = Type::get_int32_type(module);
  Type *Int32ArrayType = Type::get_array_type(Int32Type,10); //array type
  // main函数
  auto mainFun = Function::create(FunctionType::get(Int32Type, {}),
                                  "main", module);
  auto bb = BasicBlock::create(module, "entry", mainFun);
  builder->set_insert_point(bb);
  
  auto retAlloca = builder->create_alloca(Int32Type);
  auto arrAlloca = builder->create_alloca(Int32ArrayType);
  builder->create_store(CONST_INT(0), retAlloca);

  auto a0GEP = builder->create_gep(arrAlloca, {CONST_INT(0), CONST_INT(0)});  //get array[0] pointer
  builder->create_store(CONST_INT(10), a0GEP);
  auto a0Load = builder->create_load(Int32Type,a0GEP);
  auto mulans = builder->create_imul(a0Load,CONST_INT(2));
  auto a1GEP = builder->create_gep(arrAlloca, {CONST_INT(0), CONST_INT(1)});  //get array[1] pointer
  builder->create_store(mulans, a1GEP);
  builder->create_ret(mulans);

  std::cout << module->print();
  delete module;
  return 0;
}
```

fun_generator.cpp:

```c++
int main() {
  auto module = new Module("assign");
  auto builder = new IRBuilder(nullptr, module);
  Type *Int32Type = Type::get_int32_type(module);
  // callee
  auto calleeFun = Function::create(FunctionType::get(Int32Type, {Int32Type}),
                                  "callee", module);
  auto bb = BasicBlock::create(module, "entry", calleeFun);
  builder->set_insert_point(bb);
  auto mulans = builder->create_imul(CONST_INT(2),*(calleeFun->arg_begin()));
  builder->create_ret(mulans);
  // main函数
  auto mainFun = Function::create(FunctionType::get(Int32Type, {}),
                                  "main", module);
  bb = BasicBlock::create(module, "entry", mainFun);
  builder->set_insert_point(bb);

  auto call=builder->create_call(calleeFun, {CONST_INT(110)});
  builder->create_ret(call);

  std::cout << module->print();
  delete module;
  return 0;
}
```

if_generator.cpp:

```c++
int main() {
  auto module = new Module("assign");
  auto builder = new IRBuilder(nullptr, module);
  Type *FloatType = Type::get_float_type(module);
  Type *Int32Type = Type::get_int32_type(module);
  // main函数
  auto mainFun = Function::create(FunctionType::get(Int32Type, {}),
                                  "main", module);
  auto bb = BasicBlock::create(module, "entry", mainFun);
  builder->set_insert_point(bb);
  
  auto aAlloca = builder->create_alloca(FloatType);
  builder->create_store(CONST_FP(5.555), aAlloca);
  auto judge1 = builder->create_load(FloatType,aAlloca);
  auto res = builder->create_fcmp_gt(judge1,CONST_FP(1.0));		//judge
  auto truebb = BasicBlock::create(module, "truebb", mainFun);
  auto falsebb = BasicBlock::create(module, "falsebb", mainFun);
  auto br = builder->create_cond_br(res, truebb, falsebb);
  
  builder->set_insert_point(truebb);			//case 'true'
  builder->create_ret(CONST_INT(233));
  
  builder->set_insert_point(falsebb);			//case 'false'
  builder->create_ret(CONST_INT(0));

  std::cout << module->print();
  delete module;
  return 0;
}
```

while_generator.cpp:

```c++
int main() {
  auto module = new Module("assign");
  auto builder = new IRBuilder(nullptr, module);
  Type *Int32Type = Type::get_int32_type(module);
  // main函数
  auto mainFun = Function::create(FunctionType::get(Int32Type, {}),
                                  "main", module);
  auto bb = BasicBlock::create(module, "entry", mainFun);
  builder->set_insert_point(bb);
  
  auto aAlloca = builder->create_alloca(Int32Type);
  auto iAlloca = builder->create_alloca(Int32Type);
  builder->create_store(CONST_INT(10), aAlloca);
  builder->create_store(CONST_INT(0), iAlloca);
  auto truebb = BasicBlock::create(module, "truebb", mainFun);
  auto falsebb = BasicBlock::create(module, "falsebb", mainFun);
  auto judgebb = BasicBlock::create(module, "judgebb", mainFun);
  auto br1 = builder->create_br(judgebb);
  
  builder->set_insert_point(judgebb);					//while judge
  auto judge1 = builder->create_load(Int32Type,iAlloca);
  auto res = builder->create_icmp_lt(judge1,CONST_INT(10));
  auto br2 = builder->create_cond_br(res, truebb, falsebb);
  
  builder->set_insert_point(truebb);					//while body
  auto newi = builder->create_load(Int32Type,iAlloca);
  auto tempi = builder->create_iadd(newi,CONST_INT(1));
  builder->create_store(tempi, iAlloca);
  auto newa = builder->create_load(Int32Type,aAlloca);
  auto tempa = builder->create_iadd(newa,tempi);
  builder->create_store(tempa, aAlloca);
  auto br3 = builder->create_br(judgebb);
  
  builder->set_insert_point(falsebb);
  auto retnum = builder->create_load(Int32Type,aAlloca);
  builder->create_ret(retnum);


  std::cout << module->print();
  delete module;
  return 0;
}
```

结果展示：

![](/img/CPexp3/result2.png)

### 问题部分

#### 问题1: cpp与.ll的对应

请描述你的cpp代码片段和.ll的每个BasicBlock的对应关系。描述中请附上两者代码。

以`while_generator.cpp`与`while_hand.ll`为例：

```c++
auto bb = BasicBlock::create(module, "entry", mainFun);
builder->set_insert_point(bb);

auto aAlloca = builder->create_alloca(Int32Type);
auto iAlloca = builder->create_alloca(Int32Type);
builder->create_store(CONST_INT(10), aAlloca);
builder->create_store(CONST_INT(0), iAlloca);
```

这段代码对应着main函数开头在while之前的块：

```
%1 = alloca i32, align 4		;a
%2 = alloca i32, align 4		;i	
store i32 10, i32* %1, align 4
store i32 0, i32* %2, align 4
br label %3	
```

```c++
auto judgebb = BasicBlock::create(module, "judgebb", mainFun);
builder->set_insert_point(judgebb);					//while judge
auto judge1 = builder->create_load(Int32Type,iAlloca);
auto res = builder->create_icmp_lt(judge1,CONST_INT(10));
auto br2 = builder->create_cond_br(res, truebb, falsebb);
```

我将while循环的判断语句单独设置了一个块，在.ll文件中对应着：

```
3:					;while judge
  %4 = load i32, i32* %2, align 4
  %5 = icmp slt i32 %4, 10
  br i1 %5, label %6, label %11
```

```c++
auto truebb = BasicBlock::create(module, "truebb", mainFun);
builder->set_insert_point(truebb);					//while body
auto newi = builder->create_load(Int32Type,iAlloca);
auto tempi = builder->create_iadd(newi,CONST_INT(1));
builder->create_store(tempi, iAlloca);
auto newa = builder->create_load(Int32Type,aAlloca);
auto tempa = builder->create_iadd(newa,tempi);
builder->create_store(tempa, aAlloca);
auto br3 = builder->create_br(judgebb);
```

这段代码对应着while循环体内部的代码块，在判断语句为真时进入，在.ll文件中对应着：

```
6:					;while body
  %7 = load i32, i32* %2, align 4
  %8 = add i32 %7, 1
  store i32 %8, i32* %2
  %9 = load i32, i32* %1, align 4
  %10 = add i32 %8, %9
  store i32 %10, i32* %1
  br label %3
```

```c++
builder->set_insert_point(falsebb);
auto retnum = builder->create_load(Int32Type,aAlloca);
builder->create_ret(retnum);
```

这段代码对应着跳出while循环后main函数剩余的代码块，在.ll文件中对应着：

```
11:
  %12 = load i32, i32* %1, align 4
  ret i32 %12
```

#### 问题2: Visitor Pattern

请指出visitor.cpp中，treeVisitor.visit(exprRoot)执行时，以下几个Node的遍历序列:numberA、numberB、exprC、exprD、exprE、numberF、exprRoot。
序列请按如下格式指明：

```
exprRoot->numberF->exprE->numberA->exprD
```

可以看到visitor.cpp中的visit函数遍历顺序：

```c++
int visit(AddSubNode& node) override {
    auto right = node.rightNode.accept(*this);
    auto left = node.leftNode.accept(*this);
    .
    .
    .
 int visit(MulDivNode& node) override {
    auto left = node.leftNode.accept(*this);
    auto right = node.rightNode.accept(*this);
    .
    .
    .
```

对于加减号是先右后左，对于乘除号是先左后右。而添加node时都是一样的，参数1为左子树，参数2为右子树。那么遍历顺序如下图:

![](/img/CPexp3/result3.png)

序列：

```
exprRoot->numberF->exprE->exprD->numberB->numberA->exprC->numberA->numberB
```

#### 问题3: getelementptr

请给出IR.md中提到的两种getelementptr用法的区别,并稍加解释:

```
    %2 = getelementptr [10 x i32], [10 x i32]* %1, i32 0, i32 %0
    %2 = getelementptr i32, i32* %1 i32 %0
```

getelementptr是一个很玄乎的操作，这是由LLVM特殊的索引机制导致的。比如说，在c语言中，对于`Type *a; b=&a->col;`，很明显就一个索引`->col`，a仅仅只是一个Type型指针。但是LLVM在外面多包了一层，在LLVM看来，这个句子会是这样子的：`Type *a; b=&a[0]->col;`这样，就形成了两层索引。

那么回到上面的两条语句，对于第一条语句来说，其返回的是 %1 + 0 * len of([10 x i32]) + 0 * len of(i32)，返回值类型为i32*。

对于第二条语句，其返回的是 %1 + 0 * len of(i32)，返回值类型也为i32*。其实这两者返回的值也是一样的。

我们可以使用llvm生成assign.c的.ll文件，而在其中有一个很好的例子：

```c
int main(){
  int a[10];
  a[0] = 10;
  a[1] = a[0] * 2;
  return a[1];
}
```

```
define dso_local i32 @main() #0 {
  %1 = alloca i32, align 4
  %2 = alloca [10 x i32], align 16
  store i32 0, i32* %1, align 4
  %3 = getelementptr inbounds [10 x i32], [10 x i32]* %2, i64 0, i64 0
  store i32 10, i32* %3, align 16
  %4 = getelementptr inbounds [10 x i32], [10 x i32]* %2, i64 0, i64 0
  %5 = load i32, i32* %4, align 16
  %6 = mul nsw i32 %5, 2
  %7 = getelementptr inbounds [10 x i32], [10 x i32]* %2, i64 0, i64 1
  store i32 %6, i32* %7, align 4
  %8 = getelementptr inbounds [10 x i32], [10 x i32]* %2, i64 0, i64 1
  %9 = load i32, i32* %8, align 4
  ret i32 %9
}
```

定义的int a[10]分配了10个int型的空间，在.ll中可以看到，a是一个[10 x i32]类型的指针。在取a[1]的地址时，第一个偏移量为0，这表示着对于a这个基址，偏移量为0*[10 x i32]；而第二个偏移量为1，这表示着对于a[0]，偏移量为1*i32。那么，取的就是a[1]。

### 实验难点

.ll代码编写时的getelementptr函数十分奇怪且反我们平时写c/c++的逻辑，查阅了官方文档和大量资料才稍微弄懂了一些。

其次是float数，不能用二进制精确表示的数要用double型的IEEE 754标准的十六进制表示，关于其转换有点困难。