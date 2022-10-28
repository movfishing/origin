---
title: HNU database system exp1
date: 2022-10-26 15:09:24
tags: experiments
categories: Database System
---
# **数据库定义与操作语言实验**

数据库定义与操作语言实验包含 6 个实验项目，其中 5 个必修实验项目， 1 个选修实验项目。其中实验项目 1.1 至 1.5 为设计型实验，实验项目 1.6 为 验证型实验。 

### **实验 1.1 数据库定义实验** 

* 实验目的

  理解和掌握数据库 DDL 语言，能够熟练地使用 SQL DDL 语句创建、修改和 删除数据库、模式和基本表。  
  
* 实验内容和要求
  
    理解和掌握 SQL DDL 语句的语法，特别是各种参数的具体含义和使用方法； 使用 SQL 语句创建、修改和删除数据库、模式和基本表。掌握 SQL 语句常见语 法错误的调试方法。  

* 实验重点和难点  
  
  实验重点：创建数据库、基本表。  
  实验难点：创建基本表时，为不同的列选择合适的数据类型，正确创建表 级和列级完整性约束，如列值是否允许为空、主码和外码等。注意：数据完整 性约束，可以在创建基本表时定义，也可以先创建表然后定义完整性约束。由于完整性约束的限制，被引用的表要先创建。 

 

### **实验 1.2 数据基本查询实验** 

* 实验目的  
  
  掌握 SQL 程序设计基本规范，熟练运用 SQL 语言实现数据基本查询，包括 单表查询、分组统计查询和连接查询。  
  
* 实验内容和要求  
  
  针对某个数据库设计各种单表查询 SQL 语句、分组统计查询语句；设计单 个表针对自身的连接查询，设计多个表的连接查询。理解和掌握 SQL 查询语句 各个子句的特点和作用，按照 SQL 程序设计规范写出具体的 SQL 查询语句，并调试通过。 说明：简单地说，SQL 程序设计规范包含 SQL 关键字大写、表名、属性名、 存储过程名等标识符大小写混合、SQL 程序书写缩进排列等编程规范。

* 实验重点和难点  
  
  实验重点：分组统计查询、单表自身连接查询、多表连接查询。  
  实验难点：区分元组过滤条件和分组过滤条件；确定连接属性，正确设计 连接条件。 

 

### **实验 1.3 数据高级查询实验** 

* 实验目的  
  
  掌握 SQL 嵌套查询和集合查询等各种高级查询的设计方法等。  
  
* 实验内容和要求  
  
  针对自定义数据库，正确分析用户查询要求，设计各种嵌套查询和集合查 询。  
  
* 实验重点和难点  
  
  实验重点：嵌套查询。  
  实验难点：相关子查询、多层 EXIST 嵌套查询。 

 

### **实验 1.4 数据更新实验** 

* 实验目的  
  
  熟悉数据库的数据更新操作，能够使用 SQL 语句对数据库进行数据的插入、 修改、删除操作。  
  
* 实验内容和要求  
  
  针对自定义数据库设计单元组插入、批量数据插入、修改数据和删除数据 等 SQL 语句。理解和掌握 INSERT、UPDATE 和 DELETE 语法结构的各个组成成分， 结合嵌套SQL子查询，分别设计几种不同形式的插入、修改和删除数据的语句， 并调试成功。  
  
* 实验重点和难点 
  
  实验重点：插入、修改和删除数据的 SQL。  
  实验难点：与嵌套SQL子查询相结合的插入、修改和删除数据的SQL 语句； 利用一个表的数据来插入、修改和删除另外一个表的数据。 

 

### **实验 1.5 视图** 

* 实验目的  
  
  熟悉SQL语言有关视图的操作，能够熟练使用 SQL语句来创建需要的视图， 定义数据库外模式，并能使用所创建的视图实现数据管理。  
  
* 实验内容和要求  

  针对给定的数据库模式，以及相应的应用需求，创建视图和带 WITH CHECK  OPTION 的视图，并验证视图 WITH CHECK OPTION 选项的有效性。理解和掌握视 图消解执行原理，掌握可更新视图和不可更新视图的区别。  
  
* 实验重点和难点  
  
  实验重点：创建视图。  
  实验难点：可更新的视图和不可更新的视图的区别， WITH CHECK OPTION 的验证。 

 

### ***实验 1.6 索引实验** 

* 实验目的  
  
  掌握索引设计原则和技巧，能够创建合适的索引以提高数据库查询、统计 分析效率。  
  
* 实验内容和要求  
  
  针对给定的数据库模式和具体应用需求，创建唯一索引、函数索引、复合 索引等；修改索引；删除索引。设计相应的 SQL 查询验证索引有效性。学习利 用 EXPLAIN 命令分析 SQL 查询是否使用了所创建的索引，并能够分析其原因， 执行 SQL 查询并估算索引提高查询效率的百分比。要求实验数据集达到 10 万 条记录以上的数据量，以便验证索引效果。  

* 实验重点和难点 

  实验重点：创建索引。  
  实验难点：设计 SQL 查询验证索引有效性。

  ---

  ***开始实验吧！***

  ---

## 一、软件下载以及环境配置

本人此次实验使用的是MySQL 8.0，直接前往 [MySQL官网](https://dev.mysql.com/downloads/mysql/) 下载即可。关于安装的步骤可以参考课程网站上的相关文档。

需要注意的是，如果在安装过程中没有选择开机自启，那么需要手动启动MySQL。可以在命令行中使用指令`net start mysql80`以启动(mysql80是在安装过程中设置的用户名，若自己无更改，则默认是mysql80)。若提示拒绝访问，可以尝试先获取管理员权限。

MySQL安装好后，你会发现其仅提供了一个命令行工具，并没有图形化界面。想要更加直观的操作管理数据库的话，可以安装MySQL配套的MySQL WorkBench。

本人选择的是在VSCode上安装插件`MySQL`以进行数据库管理。安装好插件后，在左侧边栏会出现一个`Database`的选项，点击后进入配置界面：

![](/img/databaseexp1/connect.png)

填写图中*号标记的项目，其中密码是在安装MySQL时自己设置的管理密码。完成后点击连接即可。

## 二、实验使用的数据库

使用的教材配套的学生-课程数据库S-T，~~主要原因是用例比较少，好建~~

有三个基本表：
* 学生表：Student(Sno,Sname,Ssex,Sage,Sdept) 

* 课程表：Course(Cno,Cname,Cpno,Ccredit) 

* 学生选课表：SC(Sno,Cno,Grade)

## 三、实验代码(按本人实际操作顺序，并未按实验编号顺序)

#### 1.数据库的定义

```sql
CREATE SCHEMA ST;

CREATE TABLE
    ST.Student(
        Sno CHAR(9) PRIMARY KEY,
        Sname CHAR(20),
        Ssex CHAR(2),
        Sage SMALLINT,
        Sdept CHAR(20)
    );

CREATE TABLE
    ST.Course(
        Cno CHAR(4) PRIMARY KEY,
        Cname CHAR(40) NOT NULL,
        Cpno CHAR(4),
        Ccredit SMALLINT,
        FOREIGN KEY (Cpno) REFERENCES Course(Cno)
    );

CREATE TABLE
    ST.SC(
        Sno CHAR(9),
        Cno CHAR(4),
        Grade SMALLINT,
        PRIMARY KEY(Sno, Cno),
        FOREIGN KEY (Sno) REFERENCES Student(Sno),
        FOREIGN KEY (Cno) REFERENCES Course(Cno)
    );
```

值得注意的是，在MySQL中，`schema`与`database`并没有明确的区别，在其官方文档中提到，`CREATE SCHEMA`与`CREATE DATABASE`是等效的。

![](/img/databaseexp1/reminds.png)

使用`CREATE SCHEMA`语句实际上也是建立了一个数据库。

#### 2.数据更新

```sql
INSERT INTO Student VALUES('201215121','李勇','男','20','CS');

INSERT INTO Student VALUES('201215122','刘晨','女','19','CS');

INSERT INTO Student VALUES('201215123','王敏','女','18','MA');

INSERT INTO Student VALUES('201215125','张立','男','19','IS');

INSERT INTO Course VALUES('1','数据库',NULL,'4');

INSERT INTO Course VALUES('2','数学',NULL,'2');

INSERT INTO Course VALUES('3','信息系统',NULL,'4');

INSERT INTO Course VALUES('4','操作系统',NULL,'3');

INSERT INTO Course VALUES('5','数据结构',NULL,'4');

INSERT INTO Course VALUES('6','数据处理',NULL,'2');

INSERT INTO Course VALUES('7','PASCAL语言',NULL,'4');

UPDATE Course SET Cpno=5 WHERE Cno='1';

UPDATE Course SET Cpno=1 WHERE Cno='3';

UPDATE Course SET Cpno=6 WHERE Cno='4';

UPDATE Course SET Cpno=7 WHERE Cno='5';

UPDATE Course SET Cpno=6 WHERE Cno='7';

INSERT INTO SC VALUES('201215121','1','92');

INSERT INTO SC VALUES('201215121','2','85');

INSERT INTO SC VALUES('201215121','3','88');

INSERT INTO SC VALUES('201215122','2','90');

INSERT INTO SC VALUES('201215122','3','80');
```

在Course表建立时，由于存在Cpno参照表是自身，所以需要先建立Cpno为空值，在去修改Cpno的值。

#### 3.数据查询

```sql
SELECT * FROM student;

/*查询student中的所有元组*/

SELECT Sname FROM Student WHERE Ssex='男';

/*查询所有男生的姓名*/

SELECT Cno,Sno,MAX(Grade) FROM SC GROUP BY Cno;

/*查询每门课的最高成绩*/

SELECT
    FIRST.Cname,
    SECOND.Cname
FROM
    Course FIRST,
    Course SECOND
WHERE FIRST.Cpno = SECOND.Cno;

/*查询某课程以及其先修课程的名字*/

SELECT Sname, Cname, Grade
FROM Student, Course, SC
WHERE
    Student.Sdept = 'CS'
    AND Student.Sno = SC.Sno
    AND SC.Cno = Course.Cno;

/*查询所有CS专业的学生的所有课程成绩*/

SELECT Sno, Cno
FROM SC x
WHERE Grade > (
        SELECT AVG(Grade)
        FROM SC y
        WHERE y.Cno = x.Cno
    );

/*查询每个学生的课程成绩比课程平均成绩高的课程*/

SELECT Sname
FROM Student
WHERE EXISTS(
        SELECT *
        FROM SC
        WHERE
            Sno = Student.Sno
            AND Cno = '2'
    )
    AND NOT EXISTS(
        SELECT *
        FROM SC
        WHERE
            Sno = Student.Sno
            AND Cno = '1'
    );

/*查询选修了2号课程但是没有选修1号课程的学生名字*/

SELECT *
FROM Student
WHERE Sdept = 'CS'
EXCEPT
SELECT *
FROM Student
WHERE Sage <= 19;

/*查询CS专业中大于19岁的学生*/
```

#### 4.视图

```sql
CREATE VIEW CS_STUDENT AS 
	SELECT 
      Sno,
      Sname,
      Sage,
      Sdept 
  FROM Student 
  WHERE Sdept = 'CS'; 

/*建立CS专业学生的视图(not WITH CHECK OPTION)*/

CREATE VIEW IS_STUDENT AS 
	SELECT
	    Sno,
	    Sname,
	    Sage,
	    Sdept
	FROM Student
	WHERE Sdept = 'IS'
	WITH CHECK OPTION; 

/*建立IS专业学生的视图(WITH CHECK OPTION)*/
```

建立了两个视图，用于验证`WITH CHECK OPTION`的功能。
建立好后，我们先查看CS专业的视图：

![](/img/databaseexp1/view1.png)

先向该视图添加一个Sdept为`CS`的元组，可以成功添加，再查看视图：

![](/img/databaseexp1/view2.png)

可以在视图中看到刚添加的元组。

然后再添加一个Sdept为`IS`的元组，可以发现能成功添加，查看视图：

![](/img/databaseexp1/view3.png)

在视图中并没有发现刚刚添加的元组，但是我们可以在IS专业的视图中看到刚刚添加的元组：

![](/img/databaseexp1/view4.png)

再向IS专业的视图添加一个Sdept为`CS`的元素，可以发现提示我们添加失败：

![](/img/databaseexp1/view5.png)

然后我们到Student表中也没有看到刚刚尝试添加的元素，这就是`WITH CHECK OPTION`在起作用。

```sql
CREATE VIEW S_G(SNO, GAVG) AS 
	SELECT Sno,AVG(Grade) FROM SC GROUP BY SNO; 
```

建立了一个不可更新的视图，因为其中有GAVG项，而此项是通过计算SC表中对应项的平均值得来的，而系统无法去更改这个计算得出的平均成绩，故该视图不可更新。我们可以尝试验证：

![](/img/databaseexp1/view_update.png)

很明显，当我尝试更新时，出现了报错。

#### 5.索引

由于本部分需要测试的数据集达到10万组以上，所以我导入使用了MySQL的示例库`employees`。

可以查看employee的结构：

![](/img/databaseexp1/struct.png)

按emp_no升序建立唯一索引：

```sql
CREATE UNIQUE INDEX Empno ON employees(emp_no ASC);
```

使用`EXPLAIN`方法检测是否成功建立索引且有效：

```sql
EXPLAIN SELECT * FROM employees WHERE emp_no>=15000;
```

![](/img/databaseexp1/empno1.png)

可以看到，表中的key项为primary，未使用索引。但是possible_keys项中有Empno，表示索引成功建立且有效。

这种情况是MySQL认为全表扫描比走索引更快，就不会走索引了。

但是我们可以通过force index来强制其走索引：

```sql
EXPLAIN SELECT * FROM employees force INDEX(`Empno`) WHERE emp_no>=15000;
```

![](/img/databaseexp1/empno2.png)

可以看到key项变为了Empno，成功使用索引。

---

按birth_date的月份建立函数索引：

```sql
ALTER Table employees ADD INDEX emp_func((month(birth_date)));
```

使用`EXPLAIN`方法检测是否成功建立索引且能使用：

```sql
EXPLAIN SELECT * FROM employees WHERE month(birth_date)=4;
```

![](/img/databaseexp1/emp_func.png)

可以看到，表中的key项为emp_func，即使用了emp_func索引。

接下来测试使用索引与不使用索引时的时间消耗：

建立的索引以及执行的查询：

```sql
ALTER Table employees ADD INDEX emp_func((month(birth_date)) ASC);

SELECT * FROM employees WHERE month(birth_date)=1;
```

* 使用索引时

  ![](/img/databaseexp1/with.png)

* 不使用索引时

  ![](/img/databaseexp1/without.png)

两次测试均测试多次，时间消耗在图所示耗时上下小幅度摇动。

我们可以看到，对于employees的299600条数据，使用索引约有42%的提升。




