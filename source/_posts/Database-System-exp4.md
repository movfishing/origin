---
title: Database System exp4
date: 2022-11-17 15:47:43
tags: experiments
categories: Database System
---

# 触发器实验

### 实验内容

* 实验目的 

掌握数据库触发器的设计和使用方法。 

* 实验内容和要求 

定义 BEFORE 触发器和 AFTER 触发器。能够理解不同类型触发器的作用和执行原理，验证触发器的有效性。

* 实验重点和难点 
  
实验重点：触发器的定义。

实验难点：利用触发器实现较为复杂的用户自定义完整性。

### 实验过程与结果

本实验采用与教科书配套的S-T关系模式数据库。

注意：

* MySQL创建一个触发器只能监视一个操作，若想监视INSERT OR UPDATE，就要创建两个触发器。

* MySQL不支持:=这种赋值方式，改为使用SET。

* MySQL可以指定同一时间触发的触发器的触发顺序，创建触发器时添加选项`{ FOLLOWS | PRECEDES } other_trigger_name`即可。

创建一个BEFORE触发器：

```sql
CREATE TRIGGER AUTO_TO_SIXTY_INSERT BEFORE INSERT 
ON SC FOR EACH ROW BEGIN 
	IF(NEW.Grade <= 60) THEN SET NEW.Grade = 60;
	END IF;
END; 

CREATE TRIGGER AUTO_TO_SIXTY_UPDATE BEFORE UPDATE 
ON SC FOR EACH ROW BEGIN 
	IF(NEW.Grade <= 60) THEN SET NEW.Grade = 60;
	END IF;
END;
```

同时，我们新建一个表，用于保存SC表的更新和插入记录：

```sql
CREATE TABLE SCLog(Sno CHAR(9),Cno CHAR(4),Grade SMALLINT,Opration CHAR(6));
```

在SC表上建立触发器保存操作日志：

```sql
CREATE TRIGGER UPDATE_LOG BEFORE UPDATE 
ON SC FOR EACH ROW FOLLOWS AUTO_TO_SIXTY_UPDATE BEGIN
    INSERT INTO SCLog VALUES(NEW.Sno,NEW.Cno,NEW.Grade,'UPDATE');
END;

CREATE TRIGGER INSERT_LOG BEFORE INSERT 
ON SC FOR EACH ROW FOLLOWS AUTO_TO_SIXTY_INSERT BEGIN
	INSERT INTO SCLog VALUES(NEW.Sno,NEW.Cno,NEW.Grade,'INSERT');
END;
```

接下来验证触发器，先尝试插入一个分数为10的元组`INSERT INTO SC VALUES('201215122','2','10');`，查看SCLog表：

![](/img/databaseexp4/1.png)

可以看到成功触发触发器修改成绩为60。

再尝试将刚刚插入的元组的成绩修改为20`UPDATE SC SET Grade='20' WHERE Cno = '2';`，查看SCLog表：

![](/img/databaseexp4/2.png)

可以看到成功触发触发器修改成绩为60。

先将触发器UPDATE_LOG与INSERT_LOG删除:

```sql
DROP TRIGGER INSERT_LOG;
DROP TRIGGER INSERT_LOG;
```

然后，同样的，我们创建一个AFTER触发器：

```sql
CREATE TRIGGER UPDATE_LOG AFTER UPDATE 
ON SC FOR EACH ROW BEGIN
    INSERT INTO SCLog VALUES(NEW.Sno,NEW.Cno,NEW.Grade,'UPDATE');
END;

CREATE TRIGGER INSERT_LOG AFTER INSERT 
ON SC FOR EACH ROW BEGIN
	INSERT INTO SCLog VALUES(NEW.Sno,NEW.Cno,NEW.Grade,'INSERT');
END;
```

先执行操作`INSERT INTO SC VALUES('201215121','4','10');`，查看SCLog表：

![](/img/databaseexp4/3.png)

触发器触发，成绩修改为了60。

再尝试将刚刚插入的元组的成绩修改为80`UPDATE SC SET Grade='80' WHERE Cno = '4';`，查看SCLog表：

![](/img/databaseexp4/4.png)

成绩成功修改为了80。