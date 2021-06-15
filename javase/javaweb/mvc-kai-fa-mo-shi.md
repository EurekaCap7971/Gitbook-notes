# MVC开发模式

## MVC开发模式

Model，View，Controller，MVC开发模式将一个程序的三个部分分开进行设计，有利于解决开发中不合理的部分

### 组成

* Model模型
  * 完成具体的业务操作，例如查询数据库，封装对象
* View视图
  * 展示控制器传递的数据
* Controller控制器
  * 获取用户的输入，调用模型，将数据交给视图进行展示

### 优缺点

优点：

1. 耦合性低，方便维护，利于分工协作
2. 可重用性高，不同层次之间只有数据进行传输，不需要了解对应的原理

缺点：开发人员要求较高，提升了开发的难度

## EL表达式

Expression Language 表达式语言，用于替换和简化Jsp页面中Java代码的编写

### 基本语法

Jsp默认支持EL表达式，可以在Jsp文件中直接进行书写

* `${Java代码}`会直接执行EL表达式中的Java代码，例如`${3>4}`就会直接返回false
* `\${Java代码}`在表达式前直接使用`\`会直接忽略EL表达式，直接输出结果
* 在`Page`对象中也可以设置`isELIgnored="true"`来忽略**本页面的所有EL表达式**

#### 运算符

* 算术运算符：`+`,`-`,`*`,`/`
* 比较运算符：`>`,`<`,`>=`,`<=`,`==`,`!=`
* 逻辑运算符：`&&`,`||`,`！`
* 空运算符：`empty`，用于判断字符串、集合、数组对象等是否为null或长度是否为0

#### 获取值

EL表达式只能从**域对象**中获取值，`${域名称.键名}`

* 域名称：pageScope...原域对象后加**Scope**即可
* 也可以直接使用`${键名}`来获取值，没有域名称的话会从**最小的域对象**以此寻找是否有符合的对象
* 通过EL表达式获取对象时，会直接返回地址，也可以通过**JavaBean获取属性**的方式直接获取对象的值
* 获取List对象和Map对象时，也可以通过获取属性的方式获取值
* `${List}`会直接将List中的所有元素打印

### 隐式对象

EL表达式中有11个隐式对象

* `pageContext`：可以使用EL表达式根据此对象直接获取其他八个内置对象，最常用的例子为`${pageContext.request.contextPath}`动态获取虚拟目录，可以用在Servlet和Jsp的虚拟目录设置中

## JSTL标签

JavaServer Pages Tag Library 是Apache组织提供的开源的免费的Jsp标签，用于替换和优化Jsp上的Java代码

使用流程：

1. 导入JSTL的Jar包
2. 在Jsp页面中用`taglib`引用uri，通常的缩写为“c”

### 各标签功能

1. `c:if`标签就是简单的if判断标签，根据test属性的布尔值选择是否展示标签体内容，通常与EL表达式一起使用
2. `c:choose`标签类似switch，可以根据标签体内的`c:when`来进行switch的选择，也可以通过`c:otherwise`
3. `c:forEach`标签功能相当于Java中的foreach，可以设置begin和end来确定遍历的起点和终点，不加则默认完整遍历，`var`属性为遍容器属性的临时变量，`varStatus`包含了当前遍历次数的一些基本信息

```text
<c:if test=${boolean}>
    标签体内容
</c:if>

<c:choose>
    <c:when test="${number == x}"> this </c:when>
    <c:when test="${number == y}"> that </c:when>

    <c:otherwise> 其他情况</c:otherwise>
</c:choose>

<c:forEach items="${容器}" begin="1" end="10" var="i" step="1" varStatus="s">
    ${s.index}    ${s.count}    ${i}    <br>
</c:forEach>
```

## 三层架构

三层架构是软件设计的一种架构，将软件的开发设计分为了三个部分：

1. 界面层（表示层/web层）：获取浏览器发送的请求，包含了MVC中的控制器和视图，将请求传递给业务逻辑层，并接受业务逻辑层返回的数据
2. 业务逻辑层（Service层）：相应界面层的控制，组合调用数据访问层的基本方法
3. 数据访问层（Dao层）：定义了对数据库的最基本的CRUD操作

![&#x4E09;&#x5C42;&#x67B6;&#x6784;](http://121.4.249.42/images/Study/MVC.png)

​

