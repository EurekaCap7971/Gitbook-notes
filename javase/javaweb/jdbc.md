# JDBC

## JDBC

Java Database Connectivity，Java数据库连接，定义了操作**所有关系型数据库**的规则（接口），由各大数据库厂商实现接口，方便Java操作数据库，运行时真正执行的代码是驱动jar包内的实现类

### 基本流程

1. 导入jar包
2. 注册驱动：`Class.forName(“com.mysql.jdbc.Driver”)`
3. 获取数据库连接对象：`Connection conn = DriverManager.getConnection(“jdbc:mysql://ip:port/database”,”user”,”password”)`
4. 定义sql语句：`String sql =“xxx”`
5. 获取执行sql的对象：`Statement stmt = conn.getStatement()`
6. 执行sql：`int/result = stmt.excutexxx(sql)`
7. 处理结果
8. 释放资源：`stmt.close`、`conn.close`

### DriverManager

驱动管理：包含**注册驱动**和**获取数据库连接**两项功能

`Class.forName(“com.mysql.jdbc.Driver”)`用于注册驱动

在`DriverManager`类中具有静态的方法`registerDriver(Driver driver)`，当将class文件加载进内存之后，自动执行`Driver`类中的静态代码块，会自动执行`DriverManager`类中的`registerDriver`方法

```java
static{
    try{
        java.sql.DriverManager.registerDriver(new Driver());
    } catch(SQLException e){
        throw new RuntimeException("Can't register driver!");
    }
}
```

在MySQL-JDBC5之后的Jar包会自动注册驱动，具体文件存放在Jar包下的`META_INF/services/java.sql.Driver`中

### Connection

数据库连接对象，具有**获取执行sql的对象**和**管理事务**的功能

获取执行对象：

* `Statement createStatement()`创建sql的对象
* `PreparedStatement prepareStatement(String sql)`

管理事务：

* `setAutoCommit(boolean autoCommit)`调用该方法来开关自动提交，即开启事务
* `commit()`提交
* `rollback()`回滚

### Statement

执行sql语句并返回结果

`boolean execute(String sql)`执行任意语句

`int executeUpdate(String sql)`执行DML语句（insert等），DDL语句（create，alter等），该方法返回值是**影响的行数**

`ResultSet executeQuery(String sql)`执行DQL语句（select等）

### ResultSet

结果对象集，封装查询结果

`ResultSet rs = executeQuery(sql)`ResultSet返回的查询结果是数据库中查询的表格

`next()`游标向下移动一行，默认指向第一行（即标题）

`getXXX(参数)`可以获取对应的数据类型，参数可以是列的序号（从1开始）也可以是列名

### JDBCUtils

JDBC的简化工具类（自己写）

JDBC需要获取连接和释放连接，代码有重复和繁杂，因此可以自己写一个JDBCUtils包进行简化：

1. 抽取注册驱动
2. 抽取方法用于连接对象
   1. 减少传递的参数，灵活修改参数的内容→**配置文件**，通过**静态代码块**读取资源
3. 抽取方法释放资源（ResultSet,Statement,Connection三种资源的释放）

```java
import java.io.FileReader;
import java.io.IOException;
import java.util.Properties;

public class JDBCUtils {
    private static String url;
    private static String user;
    private static String password;
    private static String driver;

    static {
        try {
            Properties pro = new Properties();
            try {
                pro.load(new FileReader("F:\\Desktop\\大三下学习计划\\Project\\Practice\\pro.properties"));
            } catch (IOException e) {
                e.printStackTrace();
            }
            url = pro.getProperty("url");
            user = pro.getProperty("user");
            password = pro.getProperty("password");
            driver = pro.getProperty("driver");

            try {
                Class.forName(driver);
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }

        } catch (Exception e){
            e.printStackTrace();
        }
    }
    ...
}
```

### PrepareStatement

#### SQL注入问题

在拼接sql时，有特殊关键字参与拼接，例如：

* 用户名：lfbifj
* 密码：a’ or ‘a’ = ‘a’

此时传递到sql语句中为：`select * from user where user=‘lfbifj’ and password = ‘a’ or ‘a’ = ‘a’`，密码的后半段存在一个恒等式，则sql语句会直接搜索所有数据

这种漏洞被称为**SQL注入**，通过把SQL命令插入到Web表单提交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令。

此时可以使用`PreparedStatement`对象进行预编译书写：

* 预编sql使用`?`作为占位符，`String sql = "select * from user where user = ? and password = ?"`
* 生成`PreparedStatement`的对象，传入sql：`PreparedStatement pstmt = new PreparedStatement(sql)`
* 通过`PreparedStatement`的`setxxx(value1, value2)`方法给`?`传值
* 直接执行`pstmt.excutexxx()`

使用PreparedStatement可以防止SQL注入，且效率更高。但是PreparedStatement也有不足之处：防止字符类数据的被注入的方法是**将单引号添加一个`\`，让其转义，而`%`不会被转义**，依旧会出现问题。

如果在登陆过程中，用户输入了包含`%`的数据，或是直接输入了"%xx%"，整个查询的意思就变了，变成包含查询。实际上用户什么都不输入，或者只输入一个%，都可以改变原意。

## 数据库连接池

容器技术：复用需要向底层申请的资源，不释放资源而是存储在连接池中，能够有效提升速度和节约资源

### DataSource

标准接口，位于javax.sql包下，使用`getConnection()`获取连接，通常由数据库厂商实现

### C3P0

步骤：

1. 导入两个jar包：c3p0-0.9.5.2.jar & mchange-commons-java-0.2.12.jar
2. 定义配置文件：c3p0.properties 或 c3p0-config.xml，将文件放在src下即可
3. 创建对象：`ComboPooledDataSource`数据库连接池对象
4. 获取连接：`getConnection()`

```java
public class JDBC_Demo {
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        DataSource ds = new ComboPooledDataSource();

        for(int i=1;i<=10;i++){
            Connection conn = ds.getConnection();
            System.out.println(conn);
        }
    }
}
```

创建`ComboPooledDatSource`如果不传值，则会使用c3p0-config.xml的**默认**配置，也可以选择传入配置文件中不同的配置名来进行不同的连接设置

### Druid

阿里巴巴提供的数据库连接技术

步骤：

1. 导入jar包：druid-1.0.9.jar
2. 定义配置文件：任意路径下任意名称的properties形式的文件
3. 获取数据库连接对象：`DriodDataSourceFactory`通过工厂类获取
4. 获取连接：`getConnection()`

```java
public class JDBC_Demo {
    public static void main(String[] args) throws Exception {
        Properties pro = new Properties();
        InputStream is = JDBC_Demo.class.getClassLoader().getResourceAsStream("F:\\Desktop\\大三下学习计划\\Project\\Practice\\JDBC\\src\\druid.properties");
        pro.load(is);
        DataSource ds = DruidDataSourceFactory.createDataSource(pro);
        ...       
    }
}
```

Druid同样可以使用**工具类**来简化操作：读取配置文件，注册驱动，获取DataSource，创建连接，关闭连接

```java
package xyz.eucaplee.Utils;

import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.io.FileReader;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

public class JDBCUtils_Druid {
    private static DataSource ds;

    static{
        try {
            Properties pro = new Properties();
            pro.load(new FileReader("F:\\Desktop\\大三下学习计划\\Project\\Practice\\druid.properties"));
            ds = DruidDataSourceFactory.createDataSource(pro);
        } catch(Exception e) {
            e.printStackTrace();
        }

    }
    public static Connection getConnection() throws SQLException {
        return ds.getConnection();
    }

    public static void close(Connection conn){
        close(null,conn);
    }

    public static void close(Statement stmt, Connection conn){
        if(stmt!=null){
            try{
                stmt.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
        if(conn!=null){
            try{
                conn.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
    }
    public static DataSource getDataSource(){
        return ds;
    }
}
```

### Spring JDBC

Spring框架对JDBC做了简单的封装，提供了一个JDBCTemplate对象用于简化JDBC的开发

步骤：

1. 导入jar包（5个）：`commons-logging-1.2.jar`,`spring-beans-5.0.0.RELEASE.jar`,`spring-core-5.0.0.RELEASE.jar`,`spring-jdbc-5.0.0.RELEASE.jar`,`spring-tx-5.0.0.RELEASE.jar`
2. 创建JdbcTemplate对象，依赖于数据源DataSource，可以通过Druid工具类获取`JdbcTemplate template = new JdbcTemplate(JDBCUtils_Druid.getDataSource())`
3. 调用JDBCTemplate方法来完成CRUD：
   1. `update()`执行DML，增删改
   2. `queryForMap()`，查询结果封装为Map
   3. `queryForList()`，查询结果封装为List
   4. `query()`，查询结果封装为JavaBean
   5. `queryForObject()`，查询结果封装为对象

```java
public class JDBC_Demo {
    public static void main(String[] args) {
        JdbcTemplate template = new JdbcTemplate(JDBCUtils_Druid.getDataSource());
        String sql = "select * from student";

        List<Map<String, Object>> List = template.queryForList(sql);

        System.out.println(List);

    }
}
```

`template.queryForMap(String sql)`会返回一个Map类型的数据，但是只能包含一条记录

`template.queryForList(String sql)`则会返回一个List类型的数据，存储map类型的元素，包含多条记录

`template.query(String sql,RowMapper<> rmp)`会将数据封装成JavaBean对象，**如果是自己写的话需要重写RowMapper接口里的方法**，但是框架提供了它的实现类\*\*`new BeanPropertyRowMapper<Object>(Object.class)`

该实现类需要传入**需要存放数据的类的字节码**，然后会自动识别读取的内容并生成存放有对应数据的对象，然后存放到List集合中

大前提：存放数据的对象，其属性的名字**必须与数据库中的列名完全一致，否则无法正确调用set方法存值**

```java
public class JDBC_Demo {
    public static void main(String[] args) {
        JdbcTemplate template = new JdbcTemplate(JDBCUtils_Druid.getDataSource());
        String sql = "select * from student";

        List<Student> List = template.query(sql, new BeanPropertyRowMapper<Student>(Student.class));

        for (Student stu : List){
            System.out.println(stu);
        }

    }
}
```

`template.queryForObject(String sql , class)`通常用于查询聚合函数，返回的值一般是数值类型，传入的class文件指的是`Long.class`、`Interger.class`等

