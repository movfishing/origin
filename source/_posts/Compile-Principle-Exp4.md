---
title: Compile Principle Exp4
date: 2022-12-19 21:06:10
tags: experiments
categories: CompilePrinciple
---

# Lab4 实验报告

## 实验要求

本次实验要使用提供的Light IR接口编写C++程序实现一个cminus-f的编译器。

## 实验难点

首先就是万事开头难，难以找到着手点，给的示例比较少，一开始不知道该干什么。

其次就是写代码的时候，写了两个感觉已经掌握了步骤，但是到后面关于if else与var的部分还是比较困难，耗费了我大量精力。

## 实验设计

总体的设计还是按照语义规则来，部分比较简单，但也还是有几个比较困难的部分。

首先，关于全局变量的设计，所有的函数都没有返回值，所有我引入了全局变量去存储一些在调用函数时需要返回的值，比如说四则运算中，就需要存右边表达式的值。

另外，有很多高度重复使用的函数或组合函数，我也进行了相关的宏定义。

可以看一些比较困难，关键的部分：

```c++
void CminusfBuilder::visit(ASTVar &node)
{
    auto var = scope.find(node.id);              /* 从域中取出对应变量 */
    bool should_return_lvalue = need_as_address; /* 判断是否需要返回地址（即是否由赋值语句调用） */
    need_as_address = false;                     /* 重置全局变量need_as_address */
    Value *index = CONST_INT(0);                 /* 初始化index */
    if (node.expression != nullptr)
    {                                                     /* 若有expression */
        node.expression->accept(*this);                   /* 处理expression，得到结果Res */
        auto res = Res;                                   /* 存储结果 */
        if (checkFloat(res))                              /* 判断结果是否为浮点数 */
            res = builder->create_fptosi(res, Int32Type); /* 若是，则矫正为整数 */
        index = res;                                      /* 赋值给index，表示数组下标 */
        /* 判断下标是否为负数。若是，则调用neg_idx_except函数进行处理 */
        auto function = builder->get_insert_block()->get_parent();                       /* 获取当前函数 */
        auto indexTest = builder->create_icmp_lt(index, CONST_ZERO(Int32Type));          /* test是否为负数 */
        auto failBB = BasicBlock::create(module.get(), node.id + "_failTest", function); /* fail块 */
        auto passBB = BasicBlock::create(module.get(), node.id + "_passTest", function); /* pass块 */
        builder->create_cond_br(indexTest, failBB, passBB);                              /* 设置跳转语句 */

        builder->set_insert_point(failBB);                       /* fail块，即下标为负数 */
        auto fail = scope.find("neg_idx_except");                /* 取出neg_idx_except函数 */
        builder->create_call(static_cast<Function *>(fail), {}); /* 调用neg_idx_except函数进行处理 */
        builder->create_br(passBB);                              /* 跳转到pass块 */

        builder->set_insert_point(passBB);                                /* pass块 */
        if (var->get_type()->get_pointer_element_type()->is_array_type()) /* 若为指向数组的指针 */
            var = builder->create_gep(var, {CONST_INT(0), index});        /* 则进行两层寻址（原因在上一实验中已说明） */
        else
        {
            if (var->get_type()->get_pointer_element_type()->is_pointer_type()) /* 若为指针 */
                var = builder->create_load(var);                                /* 则取出指针指向的元素 */
            var = builder->create_gep(var, {index});                            /* 进行一层寻址（因为此时并非指向数组） */
        }
        if (should_return_lvalue)
        {                            /* 若要返回值 */
            Res = var;               /* 则返回var对应的地址 */
            need_as_address = false; /* 并重置全局变量need_as_address */
        }
        else
            Res = builder->create_load(var); /* 否则则进行load */
        return;
    }
    /* 处理无expression的情况 */
    if (should_return_lvalue)
    {                            /* 若要返回值 */
        Res = var;               /* 则返回var对应的地址 */
        need_as_address = false; /* 并重置全局变量need_as_address */
    }
    else
    {                                                                     /* 否则 */
        if (var->get_type()->get_pointer_element_type()->is_array_type()) /* 若指向数组 */
            Res = builder->create_gep(var, {CONST_INT(0), CONST_INT(0)}); /* 则寻址 */
        else
            Res = builder->create_load(var); /* 否则则进行load */
    }
}
```

首先就是var的处理，由于有多种变量的引用，有的需要返回其地址修改值，所以我定义了一个bool型全局变量need_as_address来判断这两种情况。其次，对于无expression的情况，有一种情况是`a[]`这样的形式，需要我们判断是否指向数组。

```c++
Type *retType; /* 返回类型 */
    /* 根据不同的返回类型，设置retType */
    if (node.type == TYPE_INT)
    {
        retType = Int32Type;
    }
    if (node.type == TYPE_FLOAT)
    {
        retType = FloatType;
    }
    if (node.type == TYPE_VOID)
    {
        retType = Type::get_void_type(module.get());
    }
    /* 根据函数声明，构造形参列表（此处的形参即参数的类型） */
    std::vector<Type *> paramsType; /* 参数类型列表 */
    for (auto param : node.params)
    {
        if (param->isarray)
        {                                /* 若参数为数组形式，则存入首地址指针 */
            if (param->type == TYPE_INT) /* 若为整型 */
                paramsType.push_back(Type::get_int32_ptr_type(module.get()));
            else if (param->type == TYPE_FLOAT) /* 若为浮点型 */
                paramsType.push_back(Type::get_float_ptr_type(module.get()));
        }
        else
        {                                /* 若为单个变量形式，则存入对应类型 */
            if (param->type == TYPE_INT) /* 若为整型 */
                paramsType.push_back(Int32Type);
            else if (param->type == TYPE_FLOAT) /* 若为浮点型 */
                paramsType.push_back(FloatType);
        }
    }
    auto funType = FunctionType::get(retType, paramsType);                    /* 函数结构 */
    auto function = Function::create(funType, node.id, module.get());         /* 创建函数 */
    scope.push(node.id, function);                                            /* 将函数加入到域 */
    scope.enter();                                                            /* 进入此函数作用域 */
    auto bb = BasicBlock::create(module.get(), node.id + "_entry", function); /* 创建基本块 */
    builder->set_insert_point(bb);                                            /* 将基本块加入到builder中 */
    /* 将实参和形参进行匹配 */
    std::vector<Value *> args; /* 创建vector存储实参 */
    for (auto arg = function->arg_begin(); arg != function->arg_end(); arg++)
    {                         /* 遍历实参列表 */
        args.push_back(*arg); /* 将实参加入vector */
    }
    for (int i = 0; i < node.params.size(); i++)
    {                                /* 遍历形参列表（=遍历实参列表） */
        auto param = node.params[i]; /* 取出对应形参 */
        arg = args[i];               /* 取出对应实参 */
        param->accept(*this);        /* 调用param的accept进行处理 */
    }
    node.compound_stmt->accept(*this); /* 处理函数体内语句compound-stmt */
    if (builder->get_insert_block()->get_terminator() == nullptr)
    {
        if (function->get_return_type()->is_void_type())
            builder->create_void_ret();
        else if (function->get_return_type()->is_float_type())
            builder->create_ret(CONST_FP(0.));
        else
            builder->create_ret(CONST_INT(0));
    }
    scope.exit(); /* 退出此函数作用域 */
```

然后就是函数声明，涉及到了关于作用域的操作。每个作用域都有其单独的空间，变量等，需要注意存入作用域中的变量。函数的参数也是该函数作用域中的变量，这里我设置了一个全局变量arg用于获取实参并存入作用域。

```c++
void CminusfBuilder::visit(ASTSelectionStmt &node)
{
    auto function = builder->get_insert_block()->get_parent(); /* 获得当前所对应的函数 */
    node.expression->accept(*this);                            /* 处理条件判断对应的表达式，得到返回值存到expression中 */
    auto resType = Res->get_type();                            /* 获取表达式得到的结果类型 */
    Value *TrueFalse;                                          /* 存储if判断的结果 */
    if (resType->is_integer_type())
    { /* 若结果为整型，则针对整型进行处理(bool类型视为整型) */
        auto intType = Int32Type;
        TrueFalse = builder->create_icmp_gt(Res, CONST_ZERO(intType)); /* 大于0视为true */
    }
    else if (resType->is_float_type())
    { /* 若结果为浮点型，则针对浮点数进行处理 */
        auto floatType = FloatType;
        TrueFalse = builder->create_fcmp_gt(Res, CONST_ZERO(floatType)); /* 大于0视为true */
    }
    if (node.else_statement != nullptr)
    {                                                                       /* 若存在else语句 */
        auto trueBB = BasicBlock::create(module.get(), "true", function);   /* 创建符合条件块 */
        auto falseBB = BasicBlock::create(module.get(), "false", function); /* 创建else块 */
        builder->create_cond_br(TrueFalse, trueBB, falseBB);                /* 设置跳转语句 */

        builder->set_insert_point(trueBB);            /* 符合if条件的块 */
        node.if_statement->accept(*this);             /* 处理符合条件后要执行的语句 */
        auto curTrueBB = builder->get_insert_block(); /* 将块加入 */

        builder->set_insert_point(falseBB);            /* else的块 */
        node.else_statement->accept(*this);            /* 处理else语句 */
        auto curFalseBB = builder->get_insert_block(); /* 将块加入 */

        /* 处理返回，避免跳转到对应块后无return */
        auto trueTerm = builder->get_insert_block()->get_terminator();  /* 判断true语句中是否存在ret语句 */
        auto falseTerm = builder->get_insert_block()->get_terminator(); /* else语句中是否存在ret语句 */
        BasicBlock *retBB;
        if (trueTerm == nullptr || falseTerm == nullptr)
        {                                                              /* 若有一方不存在return语句，则需要创建返回块 */
            retBB = BasicBlock::create(module.get(), "ret", function); /* 创建return块 */
            builder->set_insert_point(retBB);                          /* return块（即后续语句） */
        }
        if (trueTerm == nullptr)
        {                                         /* 若符号条件后要执行的语句中不存在return */
            builder->set_insert_point(curTrueBB); /* 则设置跳转 */
            builder->create_br(retBB);            /* 跳转到刚刚设置的return块 */
        }
        if (falseTerm == nullptr)
        {                                          /* 若else语句中不存在return */
            builder->set_insert_point(curFalseBB); /* 则设置跳转 */
            builder->create_br(retBB);             /* 跳转到刚刚设置的return块 */
        }
    }
    else
    {                                                                     /* 若不存在else语句，则只需要设置true语句块和后续语句块即可 */
        auto trueBB = BasicBlock::create(module.get(), "true", function); /* true语句块 */
        auto retBB = BasicBlock::create(module.get(), "ret", function);   /* 后续语句块 */
        builder->create_cond_br(TrueFalse, trueBB, retBB);                /* 根据条件设置跳转指令 */

        builder->set_insert_point(trueBB);                            /* true语句块 */
        node.if_statement->accept(*this);                             /* 执行条件符合后要执行的语句 */
        if (builder->get_insert_block()->get_terminator() == nullptr) /* 补充return（同上） */
            builder->create_br(retBB);                                /* 跳转到刚刚设置的return块 */
        builder->set_insert_point(retBB);                             /* return块（即后续语句） */
    }
}
```

在编写时想到了在if，else的基本块中，可能是没有ret语句的，我们需要针对这种情况创建一个ret的基本块去实现正常的跳出。

测试一下助教提供的测试用例：

![](/img/CPexp4/1.png)

成功通过了测试用例。

自己设置了一个测试用例：

```c++
int x[1];
void call(int u[]){
    int a;
    a = u[0];
    return ;
}
void funArray(int u[])
{
    call(u);
    return ;
}
void main(void)
{
    x[0] = 90;
    funArray(x);
    return;
}
```

![](/img/CPexp4/2.png)

能够成功通过。这个测试用例主要看了关于var的几种形式的引用，对于指针型的u[]，数组型的x[]都能通过。

### 实验总结

本次实验是在实验三的基础上完成的一个微型编译器，但是个人做起来感觉十分困难，一开始感觉无从下手，查阅了很多资料才略知皮毛。尤其困难的是很多类型与函数的翻找，一开始翻老半天才能找到想要的函数与类型的定义在哪。但是收获很多，了解了编译器的运行流程。
