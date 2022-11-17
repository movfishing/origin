---
title: Database System exp3
date: 2022-11-17 15:47:37
tags: experiments
categories: Database System
---

# 数据库完整性定义与检查

完整性语言实验包含 3 个实验项目，其中 2 个必修项目，1 个选修项目。
该实验的各个实验项目均为验证型实验项目。 

### 实验内容

#### 实验 3.1 实体完整性实验 

* 实验目的 

掌握实体完整性的定义和维护方法。 

* 实验内容和要求 

定义实体完整性，删除实体完整性。能够写出两种方式定义实体完整性的SQL 语句：创建表时定义实体完整性、创建表后定义实体完整性。设计SQL语句验证完整性约束是否起作用。 

* 实验重点和难点 
  
实验重点：创建表时定义实体完整性。

实验难点：有多个候选码时实体完整性的定义。 

#### 实验 3.2 参照完整性实验 

* 实验目的 

掌握参照完整性的定义和维护方法。 

* 实验内容和要求 
  
定义参照完整性，定义参照完整性的违约处理，删除参照完整性。写出两种方式定义参照完整性的 SQL 语句：创建表时定义参照完整性、创建表后定义参照完整性。 

* 实验重点和难点 

实验重点：创建表时定义参照完整性。 

实验难点：参照完整性的违约处理定义。 

#### 实验 3.3 用户自定义完整性实验 

* 实验目的

掌握用户自定义完整性的定义和维护方法。

* 实验内容和要求 
  
针对具体应用语义，选择 NULL/NOT NULL、DEFAULT、UNIQUE、CHECK 等，定义属性上的约束条件。 

* 实验重点和难点 

实验重点：NULL/NOT NULL, DEFAULT。 

实验难点：CHECK。 

### 实验过程与结果

本实验采用与教科书配套的S-T关系模式数据库。

#### 3.1 实体完整性实验

在列级与表级分别建立实体完整性：

```sql
CREATE TABLE
    ST.Student(
        Sno CHAR(9) PRIMARY KEY,
        Sname CHAR(20),
        Ssex CHAR(2),
        Sage SMALLINT,
        Sdept CHAR(20)
    );
/*列级*/
CREATE TABLE
    ST.SC(
        Sno CHAR(9),
        Cno CHAR(4),
        Grade SMALLINT,
        PRIMARY KEY(Sno, Cno),
        FOREIGN KEY (Sno) REFERENCES Student(Sno),
        FOREIGN KEY (Cno) REFERENCES Course(Cno)
    );
/*表级*/
```

验证实体完整性：

先查看表Student，表中已经有主码为`201215121`的元组，执行操作`INSERT Student VALUES(201215121,NULL,NULL,NULL,NULL);`后，报错：

![](/img/databaseexp3/1.png)

查看表SC，表中已经有主码为`(201215121,1)`的元组，执行操作`INSERT SC VALUES(201215121,1,NULL);`后，报错：

![](/img/databaseexp3/2.png)

尝试删除实体完整性`ALTER TABLE Student DROP PRIMARY KEY;`：

![](/img/databaseexp3/3.png)

删除失败,因为有外码依赖。写了一个简单的测试数据库测试，发现提示删除操作执行成功：

![](/img/databaseexp3/4.png)

删除前表结构：

![](/img/databaseexp3/5.png)

删除后：

![](/img/databaseexp3/6.png)

可以看到Col1与Col2已经不是主码了。

#### 实验 3.2 参照完整性实验

重新创建SC表：

在建立表时定义参照完整性

```sql
CREATE TABLE
    ST.SC(
        Sno CHAR(9),
        Cno CHAR(4),
        Grade SMALLINT,
        PRIMARY KEY(Sno, Cno),
        FOREIGN KEY (Sno) REFERENCES Student(Sno)
        ON DELETE CASCADE /*级联删除SC表中相应的元组*/
        ON UPDATE CASCADE,/*级联更新SC表中相应的元组*/
    );
```

建表后定义参照完整性

```sql
ALTER TABLE SC ADD FOREIGN KEY (Cno) REFERENCES Course(Cno)
                   ON DELETE NO ACTION 
                   /*当删除course 表中的元组造成了与SC表不一致时拒绝删除*/
                   ON UPDATE CASCADE;
                   /*当更新course表中的cno时，级联更新SC表中相应的元组*/
```

验证参照完整性作用：

先查看SC表中的所有元组：

![](/img/databaseexp3/7.png)

随后，先进行操作`UPDATE Student SET Sno='201315121' WHERE Sno='201215121';`将Student表中Sno为`201215121`的元组的Sno改为`201315121`，再查看SC表：

![](/img/databaseexp3/8.png)

发现成功级联修改。

再进行操作`DELETE FROM Student WHERE Sno='201315121';`，将刚刚更改的元组删除,再查看SC表：

![](/img/databaseexp3/9.png)

发现成功级联删除。

随后再修改Course中的Cno`UPDATE Course SET Cno='123' WHERE Cno='2';`,再查看SC表：

![](/img/databaseexp3/10.png)

成功级联修改。

最后，进行操作`DELETE FROM Course WHERE Cno='123';`，将刚刚更改的元组删除,可以看到发生错误，数据库拒绝了我的删除操作：

![](/img/databaseexp3/15.png)

#### 实验 3.3 用户自定义完整性实验

重新创建SC表：

```sql
CREATE TABLE
    SC(
        Sno CHAR(9),
        Cno CHAR(4),
        Grade SMALLINT 
        CONSTRAINT C3 CHECK (
            Grade >= 0
            AND Grade <= 100
        ) DEFAULT 0,
        /*Grade取值范围是0到100*/
        CONSTRAINT nsno CHECK (Sno IS NOT NULL),
        CONSTRAINT ncno CHECK (Cno IS NOT NULL),
        CONSTRAINT SCKey PRIMARY KEY (Sno)
    );
```

MySQL并不支持在同行直接定义`NOT NULL`的约束，需要使用`CHECK`。

检查约束：

```sql
INSERT INTO SC VALUES('201215121',NULL,'92');

INSERT INTO SC VALUES('201215121','1','-1');

INSERT INTO SC (Sno,Cno) VALUES('201215121','1');
```

第一行语句，尝试插入Cno为NULL的元组，报错：

![](/img/databaseexp3/11.png)

第二行语句，尝试插入Grade为-1的元组，报错：

![](/img/databaseexp3/12.png)

第三行语句，插入元组只规定了Sno和Cno，成功将其默认设为了0：

![](/img/databaseexp3/13.png)

同样的，我们可以更改或删除这些约束，先删除C3`ALTER TABLE SC DROP CONSTRAINT C3;`,再尝试插入成绩为-1的元组：

![](/img/databaseexp3/16.png)

可以看到能成功插入。

再重新添加C3`ALTER TABLE SC ADD CONSTRAINT C3 CHECK(Grade BETWEEN 60 AND 100);`，达成修改C3的目的，再尝试插入成绩为10的元组：

![](/img/databaseexp3/14.png)

可以看到拒绝了操作。