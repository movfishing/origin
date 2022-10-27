---
title: HNU database system exp2
date: 2022-10-27 13:25:28
tags: experiments
categories: Database System
---

# 数据库安全性定义与检查

安全性实验包含两个实验项目，其中 1 个为必修，1 个为选修。自主存取控制实验为设计型实验项目，审计实验为验证型实验项目。 

### 实验 2.1 自主存取控制实验 

* 实验目的 

    掌握自主存取控制权限的定义和维护方法。 

* 实验内容和要求 

    定义用户、角色，分配权限给用户、角色，回收权限，以相应的用户名登录数据库验证权限分配是否正确。选择一个应用场景，使用自主存取控制机制设计权限分配。可以采用两种方案。

    * 方案一：采用 SYSTEM 超级用户登录数据库，完成所有权限分配工作，然后用相应用户名登录数据库以验证权限分配正确性；

    * 方案二：采用 SYSTEM 用户登录数据库创建三个部门经理用户，并分配相应的权限，然后分别用三个经理用户名登录数据库，创建相应部门的 USER, ROLE，并分配相应权限。

    实验指导书的实验报告示例，采用实验方案一。验证权限分配之前，请备份好数据库；针对不同用户所具有的权限，分别设计相应的 SQL 语句加以验证。 

* 实验重点和难点 
  
    实验重点：定义角色，分配权限和回收权限。

    实验难点：实验方案二实现权限的再分配和回收。 

 

### *实验 2.2 审计实验 

* 实验目的 
  
    掌握数据库审计的设置和管理方法，以便监控数据库操作，维护数据库安全。 

* 实验内容和要求 
  
    打开数据库审计开关。以具有审计权限的用户登陆数据库，设置审计权限，然后以普通用户登录数据库，执行相应的数据操纵 SQL 语句，验证相应审计设置是否生效，最后再以具有审计权限的用户登录数据库，查看是否存在相应的审计信息。 

* 实验重点和难点 
  
    实验重点：数据库对象级审计，数据库语句级审计 

    实验难点：合理地设置各种审计信息。一方面，为了保护系统重要的敏感数据，需要系统地设置各种审计信息，不能留有漏洞，以便随时监督系统使用情况，一旦出现问题，也便于追查；另一方面，审计信息设置过多，会严重影响数据库的使用性能，因此需要合理设置。

---

## 写在实验前

本次实验和实验一的索引实验相同，使用的是MySQL官方的示例库`employees`，关于此库的导入请自行百度。

另外由于MMySQL中`CREATE USER`与`CREATE ROLE`这块的语法与教材以及实验书上的示例代码的语法稍有不同，可以参考[官方文档](https://dev.mysql.com/doc/)配合食用。

由于VSCode的MySQL插件更换用户登录数据库比较麻烦，所以相关操作我在cmd中进行。

---

***开始实验吧！***

---

## 实验操作以及代码

#### 1.自主存取控制实验

* 方案一：由root用户创建全部的主管以及员工用户并分配权限

    ```sql
    CREATE USER 'Adam'@'localhost' IDENTIFIED BY 'jiarandiana';

    CREATE USER 'Betty'@'localhost' IDENTIFIED BY 'jiarandiana';

    CREATE USER 'Castle'@'localhost' IDENTIFIED BY 'jiarandiana';

    /*创建主管用户*/

    CREATE USER 'Diana'@'localhost' IDENTIFIED BY 'jiarandiana';

    CREATE USER 'Elieen'@'localhost' IDENTIFIED BY 'jiarandiana';

    CREATE USER 'Fork'@'localhost' IDENTIFIED BY 'jiarandiana';

    /*创建员工用户*/

    CREATE ROLE 'EmpManager'@'localhost';

    GRANT ALL PRIVILEGES ON TABLE employees TO 'EmpManager'@'localhost' WITH GRANT OPTION;

    CREATE ROLE 'SalManager'@'localhost';

    GRANT ALL PRIVILEGES ON TABLE salaries TO 'SalManager'@'localhost' WITH GRANT OPTION;

    CREATE ROLE 'TitManager'@'localhost';

    GRANT ALL PRIVILEGES ON TABLE titles TO 'TitManager'@'localhost' WITH GRANT OPTION;

    /*创建主管权限角色*/

    CREATE ROLE 'EmpEmployee'@'localhost';

    GRANT SELECT,INSERT ON TABLE employees TO 'EmpEmployee'@'localhost';

    CREATE ROLE 'SalEmployee'@'localhost';

    GRANT SELECT,INSERT ON TABLE salaries TO 'SalEmployee'@'localhost';

    CREATE ROLE 'TitEmployee'@'localhost';

    GRANT SELECT,INSERT ON TABLE titles TO 'TitEmployee'@'localhost';

    /*创建员工权限角色*/

    GRANT 'EmpManager'@'localhost' TO 'Adam'@'localhost' WITH ADMIN OPTION;

    GRANT 'SalManager'@'localhost' TO 'Betty'@'localhost' WITH ADMIN OPTION;

    GRANT 'TitManager'@'localhost' TO 'Castle'@'localhost' WITH ADMIN OPTION;

    /*分配主管权限*/

    GRANT 'EmpEmployee'@'localhost' TO 'Diana'@'localhost';

    GRANT 'SalEmployee'@'localhost' TO 'Elieen'@'localhost';

    GRANT 'TitEmployee'@'localhost' TO 'Fork'@'localhost';

    /*分配员工权限*/

    SET DEFAULT ROLE 'EmpManager'@'localhost' TO 'Adam'@'localhost';

    SET DEFAULT ROLE 'SalManager'@'localhost' TO 'Betty'@'localhost';

    SET DEFAULT ROLE 'TitManager'@'localhost' TO 'Castle'@'localhost';

    SET DEFAULT ROLE 'EmpEmployee'@'localhost' TO 'Diana'@'localhost';

    SET DEFAULT ROLE 'SalEmployee'@'localhost' TO 'Elieen'@'localhost';

    SET DEFAULT ROLE 'TitEmployee'@'localhost' TO 'Fork'@'localhost';

    /*设置默认激活角色*/
    ```

    (1)创建角色与用户时名字后面的@'localhost'指定主机为本机才能使用。

    (2)MySQL的ROLE创建后处于inactive状态，我们可以通过`SET DEFAULT ROLE {NONE | ALL | role [, role ] ...} TO user [, user ]`来将对应的role在登录用户user时自动变为active。

    验证是否成功创建以及正确分配权限：

    主管Adam：

    ![](/img/databaseexp2/1adam.png)

    能正常登录，可以对employees中的employees表进行查询、删除等操作。

    员工Diana：

    ![](/img/databaseexp2/1diana.png)

    能正常登录，仅能对employees中的employees表进行查询、插入操作。

    用户创建成功，权限分配正确。

* 方案二：先创建主管用户分配权限后，再登录主管用户创建对应员工账号分配权限

    流程如下：

    (1)创建主管用户Adam、Betty、Castle，分配权限给这三个用户

    ```sql
    CREATE USER 'Adam'@'localhost' IDENTIFIED BY 'jiarandiana';

    CREATE USER 'Betty'@'localhost' IDENTIFIED BY 'jiarandiana';

    CREATE USER 'Castle'@'localhost' IDENTIFIED BY 'jiarandiana';

    /*创建主管用户*/

    GRANT ALL PRIVILEGES ON TABLE employees TO 'Adam'@'localhost' WITH GRANT OPTION;

    GRANT CREATE USER ON *.* TO 'Adam'@'localhost' WITH GRANT OPTION;

    GRANT ALL PRIVILEGES ON TABLE salaries TO 'Betty'@'localhost' WITH GRANT OPTION;

    GRANT CREATE USER ON *.* TO 'Betty'@'localhost' WITH GRANT OPTION;

    GRANT ALL PRIVILEGES ON TABLE titles TO 'Castle'@'localhost' WITH GRANT OPTION;

    GRANT CREATE USER ON *.* TO 'Castle'@'localhost' WITH GRANT OPTION;

    /*分配主管权限*/
    ```

    (2)分别登录这三个用户账号，创建其对应的员工用户，并分配权限

    可以在cmd中使用语句`mysql -u[user] -p[password]`登录对应用户。

    ```sql
    CREATE USER 'Diana'@'localhost' IDENTIFIED BY 'jiarandiana';

    CREATE USER 'Elieen'@'localhost' IDENTIFIED BY 'jiarandiana';

    CREATE USER 'Fork'@'localhost' IDENTIFIED BY 'jiarandiana';

    /*创建员工用户*/

    GRANT SELECT,INSERT ON TABLE employees TO 'Diana'@'localhost';

    GRANT SELECT,INSERT ON TABLE salaries TO 'Elieen'@'localhost';

    GRANT SELECT,INSERT ON TABLE titles TO 'Fork'@'localhost';

    /*分配员工权限*/
    ```

    测试员工权限：

    ![](/img/databaseexp2/2diana.png)


测试权限收回：

登录Adam用户，收回Diana的INSERT权限

![](/img/databaseexp2/return1.png)

然后回root用户，查看Diana的权限，发现INSERT权限已经消失，权限成功收回。

![](/img/databaseexp2/return2.png)

#### 2.审计实验

由于MySQL的社区版并不支持Audit功能，得企业版才行，所以我采用了general log进行审计(小负载的数据库)。

```sql
show variables like '%general%';/*查看general log是否开启*/

set global general_log=on;/*开启general log*/

set global log_output='table';/*设置将日志结果存入表中(默认存入文件)*/

select * from general_log;/*查询日志内容*/
```

![](/img/databaseexp2/logs.png)

可以看到，表中有执行操作的记录，并且查询日志内容的操作也统计了进去。

