# MySQL

## MySQL

本机MySQL停用，转用服务器上的MySQL

MySQL登陆：管理员权限下的CMD，输入`mysql -hip -uuser -p`即可，可以访问本机上的数据库，如果其他人的服务器允许远程访问，也可以把主机名替换成对应的IP地址

### 通用语法

#### 数据库的数据类型：

* int：整数 `age int`
* double：小数 `grade double(4,1)`最多四位，取小数点后1位，如100.2
* date：日期，yyyy-mm-dd HH:MM:SS `birthday date`
* timestamp：时间类型，yyyy-mm-dd HH:MM:SS，如果将来不给这个字段赋值，或者赋值为null，则会默认使用**系统当前时间**自动进行赋值 `insert_time timestamp`
* varchar：字符串 `name varchar(20)`

#### 注释

有三种注释方式：

1. `-- 两个横杠带空格`单行
2. `# 井号注释`单行
3. `/* 注释 */`多行注释

#### SQL分类

1. DDL\(Data Definition Language\)数据定义语言
   * 用来定义数据库的对象，包括数据库、表、列等。`create`、`drop`、`alter`等
2. DML\(Data Manipulation Language\)数据操作语言
   * 用于增删改，`insert`、`delete`、`update`等
3. DQL\(Data Query Language\)数据查询语言
   * 查询表中记录，`select`、`where`等
4. DCL\(Data Control Language\)数据控制语言
   * 创造用户、定义访问权限和安全级别，`GRANT`、`REVOKE`等

#### 对表的操作（DDL）

```sql
create table student(
    name varchar(20),
    age int(10),
    birthday date,
    insert_time timestamp);
```

* 复制表的操作：`create table studeng2 like student;`
* 查询表
  * `show tables`查询表名
  * `desc 表明`查询表结构
* 修改表
  * `alter table NAME rename to NEW_NAME`改表名
  * `alter table NAME character set NEW_CHARACTER`改表的字符集
  * `alter table NAME add NEW_LINE TYPE`添加一列
  * `alter table NAME change LINE NEW_LINE NEW_TYPE`修改列名和类型
  * `alter table NAME modify LINE NEW_TYPE`修改列数据类型
  * `alter table NAME drop LINE`删除列
* 删除表
  * `drop table NAME`

#### 对数据的操作（DML）

* 添加数据
  * `insert into NAME(LINE1,LINE2,LINE3...) values(VALUE1,VALUE2,VALUE3...)`
  * 也可直接`insert into NAME values(VALUE1,VALUE2,VALUE3...)`
* 删除数据
  * `delete from NAME [where 条件]`，不添加条件则删除所有记录
  * 如果要删除所有记录
    * `delete from NAME`，不推荐，每条记录都会执行一次删除操作
    * `TRUNCATE table NAME`，推荐，效率高，删除表之后再创建一个一模一样的空表
* 更新数据
  * `update NAME set LINE1=VALUE1，LINE2=VALUE2,...[where 条件]`，不添加条件则该列所有元素都变

#### 查询表中数据（DQL）

**基础查询**

`select xxx,xxx,xxx from NAME`多字段查询，**\***查所有

`distinct`去除完全重复的数据

`ifnull(表达式1，表达式2)`对null值进行处理

`as`起别名

**条件运算**

`where`子句后跟条件

各式运算符：`>`,`<`,`<=`,`>=`,`=`,`<>不等号或!=`

`between ... and`区间范围

`in`集合

`like`模糊查询

`isnull`判断为空

**查询语句**

`order by line1,line2...`排序方式，后接`ASC`或`DESC`进行升序或降序排序，默认`ASC`升序

`count()`,`max()`,`min()`,`sum()`,`avg()`聚合函数，对数据进行纵向计算，需要`ifnull()`对null值进行处理。聚合函数可以起别名，方便后续使用

`group by`分组字段，后可接分组字段或聚合函数，有注意事项：

* 后可接`where`和`having`，前者在分组之前先限定，不满足条件的数据不参与分组，后者在分组后的结果里查询，不满足条件不参与查询
* `where`后不可跟聚合函数，`having`可以

`limit`分页查询，`select * from student limit 0,3;`查询从索引0开始的后3条元素，开始的索引=（当前页码-1）\*每页显示的条数

#### 用户管理（DCL）

管理用户：

1. 添加用户：`create user ‘用户名’@‘主机名’ IDENTIFIED BY ‘密码’`，通配符`%`表示可以在任意主机使用用户登陆数据库
2. 删除用户：`DROP USER ‘用户名’@‘主机名’`
3. 修改用户：
   1. 修改密码：`update user set password = password(‘xxx’) where user = ‘username’`
   2. `set password for ‘用户名’@‘主机名’ = passwod(‘xxxx’)`
   3. 权限相关：
      1. `show grants for ‘用户名’@‘主机名’`
      2. `grant 权限列表 on 数据库表.表名 to ‘用户名’@‘主机名’`，所有权限可用`all`，所有表可用`*`
      3. `revoke 权限列表 on 数据库表.表名 to ‘用户名’@‘主机名’`，同上
4. 查询用户：用root权限切换到mysql，然后查询user表

**忘记root密码**

1. cmd --&gt;net stop mysql
2. 使用无验证方法启动MySQL `mysqld --skip-grant-tables`，此时需要打开另一个cmd进行操作（如修改密码），然后需要手动结束mysqld进程
3. 启动MySQL服务

#### MySQL的约束

概念：对表中数据进行限定，保证数据的正确性，有效性和完整性

**主键约束**

`primary key`元素非空且唯一，表中只能由一个字段为主键

`primary key auto_increment`主键自动增长，添加主键为`null`时会自动根据**上一条记录的值**加1

`alter table student drop primary key`后续删除主键

`alter table student modify id primary key`后续添加主键，前提是主键元素不重复，也可删除自动增长

**非空约束**

`not null`使元素非空

可以后续添加约束`alter table student MODIFY name varchar(10) not null`，也可用同样的方式删除约束

**唯一约束**

`unique`元素唯一，null值也只能有一个

可以后续添加约束`alter table student MODIFY phone int unique`，也可用同样的方式删除约束

**外键约束**

`foreign key`外键约束，让表与表之间产生关系

```sql
create table classes(
    class varchar(10),
    teacher varchar(10)
);

create table stu(
    name varchar(10),
    age int(3),
       class varchar(10),
    constraint class_fk foreign key class references classes(class)
);
```

`constraint 外键名称 foreign key 外键列名称 references 主表名称(主表列名称)`可以通过这种方式创建外键约束

`alter table 表名 drop foreign key 外键名`删除外键

`cascade`级联操作：

* `alter table 表名 add constraint 外键名称 foreign key 外键列名 references 主表名称(主表列名) on update cascade`级联更新
* `on delete cascade`级联删除

### 数据库设计

#### 多表之间的关系

一对一：两者一一对应，例如**人和身份证**，可在任意一方添加唯一外键指向另一方的主键

一对多/多对一：一个对应多个，**员工和部门**，通常在多的一方建立外键，指向一的一方的主键

多对多：多个对应多个，**学生和课程**，多对多关系需要借助**中间表**来建立联系，中间表至少包含两个字段，分别作为外键指向两张表。中间表中的两个外键通常是**一对一**的关系，也被成为**联合主键**

#### 范式

**第一范式（1NF）**

每一列都是不可分割的原子数据项

**第二范式（2NF）**

在1NF的基础上，非码属性必须完全依赖于候选码，在1NF基础上消除非主属性对主码的**部分函数依赖**

函数依赖：如果通过A属性（或属性组）可以确定唯一的B属性，则称B依赖于A A——&gt;B

完全函数依赖：如果A是一个属性组，B属性的确定需要依赖于A属性组中的**所有属性值**，则称B完全依赖于B

部分函数依赖：如果A是一个属性组，B属性的确定只需要依赖于A属性组中的**某一个属性值**，则称B部分依赖于A

传递函数依赖：如果A属性确定B属性，B属性确定C属性，则C传递函数依赖于A

码：一个属性或属性组在一张表中，被其他属性**完全依赖**

**第三范式（3NF）**

在2NF的基础上，任何非主属性不依赖于其他非主属性，在2NF基础上消除**传递依赖**

### 备份与还原

命令行备份：`mysqldump -u用户名 -p密码 数据库名 > “保存的路径”`

命令行还原：登陆MySQL后，创建一个新的数据库，执行`source 路径`即可还原

### 多表查询

#### 内连接查询

隐式内连接：使用`where`消除无用数据

显示内连接：`select * from 表1 [inner] join 表2 on 条件`

注意事项：

* 从哪些表中查询数据
* 如何通过条件筛选无用数据
* 查询哪些字段

#### 外连接

左外连接：`select * from 表1 left [outer] join 表2 on 条件`

右外连接：`select * from 表1 right [outer] join 表2 on 条件`

根据左/右外连接，查询左/右表**所有数据及其交集部分**

#### 子查询

查询中**嵌套查询**，可以根据查询的结果进行嵌套查询

单行单列：结果可以作为判断条件的值

多行单列/单行多列：可以用`in`运算符进行判断

多行多列：结果可以用做**虚拟表**做查询

### 事务

如果一个包含了**多个步骤**的业务操作被事务管理，要么**同时成功**，要么**同时失败**

操作：

* 开启事务：`start transaction`
* 回滚：`rollback`
* 提交：`commit`

MySQL中事务默认自动提交，手动提交事务需要先**开启**然后在**提交**，可以修改默认提交方式

`select @@autocommit`默认为1，设为0时则不会自动提交

#### 事务四大特征

1. 原子性：不可分割，要么同时成功，要么同时失败
2. 持久性：事务一旦提交/回滚，数据库会持久化保存数据
3. 隔离性：多个事务之间相互独立（隔离级别相同的情况下）
4. 一致性：事务操作前后，数据总量不变

#### 事务的隔离级别

多个事物相互隔离，互相独立。

但是如果多个事务操作同一批数据，则会引发问题（并发）。设置隔离级别可以避免出现问题

存在问题：

1. 脏读：一个事物读取了另一个事务中**尚未提交**的数据
2. 不可重复读（虚读）：同一个事务中，两次读取的数据不同
3. 幻读：A事务操作数据表中的所有记录的同时，B事务添加了一条数据，则A事务无法查看到自己修改的结果

隔离级别：

* **read uncommitted**：读未提交，会产生**脏读、不可重复度、幻读**
* **read committed**：读已提交，会产生**不可重复读，幻读**
* **repetable read**：可重复读，会产生**幻读**
* **serializable**：串行化，不会产生问题（相当于同步锁）

从上到下，安全性越来越高，效率越来越低

可以通过`select @@tx_isoloation`查询隔离级别

也可以通过`set global transaction isolation level 隔离级别`来设置隔离级别

