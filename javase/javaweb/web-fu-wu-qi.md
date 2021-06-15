# Web服务器

## Web架构

C/S架构：客户端/服务器架构，通过客户端连接服务器获取资源

B/S架构：浏览器/服务器机构，通过浏览器获取服务器资源

资源分类为**静态资源**和**动态资源**：

* 静态资源：html,css,javascript等代码文件，不同用户解析同一个资源，结果永远相同，可以直接被浏览器解析
* 动态资源：servelet,jsp,php,asp等文件，不同用户访问相同动态资源，得到的结果可能不同。需要转换为静态资源之后再被浏览器解析

网络通信三要素：

1. IP：电子设备在网络中的唯一表示
2. 端口：应用程序在电子设备中的唯一表示
3. 传输协议：规定了数据传输的规则，包括tcp协议（安全协议，三次握手，慢）,udp协议（不安全协议，快）

## Tomcat

一个Web服务器软件，支持少量的JavaEE规范

### 启动与配置

根目录下，bin/startup.bat可以启动tomcat，默认端口占用端口为8080

配置文件在conf/server.xml，可以修改占用的端口等配置

### 部署项目的方式

分为三种方式部署

1. 直接部署：将项目文件直接放进根目录下webapps目录即可，也可以选择将项目打包成war包，直接放在webapps下
2. 配置conf/server.xml部署：在&lt;host&gt;标签体中可以配置&lt;Context docBase=xxx path=xxx&gt;标签体即可
3. 在conf/Catalina/localhost创建**对应路径名称**的xml文件，编写Context标签体

### 目录结构

通常会有WEB-INFO目录，其内部包含了该项目的基本配置信息，例如web.xml，classes目录以及lib目录

* web.xml：web项目的核心配置文件
* classes目录：存放对应字节码的陆慕
* lib目录：存放依赖的jar包

### IDEA配置Tomcat

IDEA会为每一个Tomcat部署的项目单独设置一个配置文件，可以在启动服务器时查看对应的`Using CATALINA_BASE`查看配置文件的路径

**tomcat部署的web项目**和实际的**工作空间项目**是不同的，在IDEA中操作是对实际的工作空间操作，但是对应的操作也会改变tomcat部署的webapps下的项目。

## Servlet

运行在服务端的小程序，定义了Java类被浏览器识别的规则

通常一个类实现了Servlet接口并复写方法即可

### 配置

配置web.xml文件可以生成对应的虚拟路径

```markup
<servlet>
    <servlet-name>demo</servlet-name>
    <servlet-class>xxxxxx</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>demo</servlet-name>
    <url-pattern>/demo</url-pattern>
</servlet-mapping>
```

* servlet下的`servlet-name`为servlet小程序的名字
* `servlet-class`是servlet对应的实现类，需要写全类名
* servlet-mapping下的`servlet-name`需要找到对应servlet小程序的名字，用名字做索引
* `url-pattern`则是虚拟路径，决定了访问的路径

### 执行原理

1. 当服务器接收到客户端浏览器的请求后，会解析请求的URL地址，获取访问的servlet资源路径
2. 查找web.xml，寻找是否有对应的url-pattern标签
3. 查找对应的servlet-class标签，找到对应的类
4. tomcat会将对应的字节码文件加载进内存，并创建对象
5. 调用类中的方法

### 生命周期

1. 被创建时：执行init方法，整个生命周期只执行一次
   * 默认是在**第一次被访问时**才算被创建
   * 可以通过给web.xml下的Servlet标签的**&lt;load-on-startup&gt;标签**赋值来确定合适创建，负整数为访问时创建，0或正整数为服务器启动时创建
   * init方法只执行一次，说明Servlet是**单例**的，多个用户同时访问时可能会出现线程安全问题。此时不要在Servlet中定义成员变量，或是定义了不修改即可解决，通过volatile也可以解决
2. 被访问/提供服务时：执行service方法
   * 每次访问servlet都会被执行
3. 被销毁时：执行destroy方法
   * Servlet被销毁时执行，服务器关闭时Servlet被销毁
   * 只有正常关闭时才会调用destroy
   * 在服务器关闭前调用，通常用于释放资源

### 快速配置

Servlet3.0提供了注解配置的方式，便于快速配置

在实现了Servlet接口的类上加**@WebServlet**注解，并配置对应的属性即可

@WebServlet\(“虚拟路径”\)，具有其它的属性，可以快捷设置虚拟路径，初始化场景，Servlet-name等

### 体系结构

Servlet接口下有两个抽象类：

* GenericServlet
  * 继承此类时只需要复写service方法即可，其他方法默认复写为空
* HTTPServlet
  * 是对http协议的封装和描述
  * 可以复写doGet\(\)和doPost\(\)方法

## Http

超文本传输协议，定义了客户端和服务器通信时，发送数据的格式

特点：

1. 基于TCP/IP的高级协议
2. 默认端口为80
3. 基于请求/响应模型的：一次请求对应一次响应
4. 无状态的：每次请求之间相互独立，数据不能交互

http1.0版本是一次请求建立一次新连接，得到响应后马上释放

http1.1版本是一次请求建立一次新连接，一段时间内没有新的请求才释放

### 请求消息

请求消息的数据格式：

* 请求行
  * 按照：请求方式    请求URL    请求协议/版本的格式
  * 请求方式有7种，常用2种
    * get：请求参数在请求行中，也跟在URL后，有长度限制且不安全
    * post：请求参数在请求体中，无长度限制，相对安全
* 请求头
  * ```http
    GET /URL HTTP/1.1 
    Host: localhost:8080 
    Connection: keep-alive 
    Cache-Control: max-age=0 
    Upgrade-Insecure-Requests: 1 
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.182 Safari/537.36 
    Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9 
    Sec-Fetch-Site: none Sec-Fetch-Mode: navigate Sec-Fetch-User: ?1
    Sec-Fetch-Dest: document Accept-Encoding: gzip, deflate, br Accept-Language: zh-CN,zh;q=0.9
    Cookie:SID=s%3APREbzhQCU5wDWdyfIHCRnnch28MmfNo5.fQ8366nYZo%2FT8Zep93EpOcgZO95YsdZiioQX4oxdASQ; Hm_lvt_eaa57ca47dacb4ad4f5a257001a3457c=1609894854,1610611629,1611913885; Idea-9dbc10d5=617ae407-698d-47ad-9656-f2d4190b6a4a; _xsrf=2|0c91c625|9e0ae29df4ba4dc06daa83257b2666a7|1619536664; username-localhost-8888="2|1:0|10:1619628940|23:username-localhost-8888|44:Y2E0NjdlMmJhODI0NGQ2MmJkMTBiODM0NWMzMzNlZmU=|775750b22c0d16787d9463d37e8c140a4060785fe9b250b7f7c2480cecaaf4a0"; JSESSIONID=2AB3E8F103B9FDC073F3FF2021E1A979
    ```
  * 常见的请求头：
    * `User-Agent`浏览器版本信息，可以通过获取版本信息来执行不同的操作
    * `Referer`根据获取浏览器从哪来，可以**防盗链**，也可以用作统计
* 请求空行
* 请求体
  * get方式没有请求体
  * post方式的请求体有一些表单的数据

#### Request

Request对象和Response对象是服务器处理请求消息和响应消息必备的两个对象：

1. 当浏览器根据URL请求服务器对应的资源时，tomcat会自动生成Request和Response对象
2. Request对象中封装了请求消息的数据，Response对象中封装了服务器处理好的响应消息的数据

#### Request对象继承体系结构

ServletRequest接口→HTTPServletRequest接口→org.apache.catalina.connector.RequestFacade实现类

#### Request的基本功能

1. 获取请求行数据
   * `String getMethod()`获取请求方式
   * `String getContextPath()`获取虚拟目录
   * `String getServletPath()`获取Servlet路径
   * `String getQueryString()`获取请求参数
   * `String getRequestURI()`获取请求URI
   * `String getRequestURL()`获取请求URL
   * `String getProtocol()`获取HTTP协议版本
   * `String getRemoteAddr()`获取客户端ip
2. 获取请求头数据
   * `String getHeader()`根据名字获取Header
   * `getHeaderName()`获取Header索引的集合
3. 获取请求体数据
   * 只有post方法才有请求体，在请求体中会封装post的请求参数
   * `BufferedReader getReader()`获取字符输入流，只能操作**字符数据**
   * `ServletInputStream getInputStream()`获取字节输入流，可以操作所有类型的数据

#### 其他功能

获取请求参数的通用方法（不论get/post）：

* `String getParameter(String name)`根据参数名称获取参数值
* `String[] getParameterValues(String name)`根据参数名称获取参数值的数组
* `Enumeration<String> getParameterNames()`获取所有请求的参数名称
* `Map<String,String[]> getParameterMap()`获取所有参数的map集合

> 中文乱码会出现在post方式中，在tomcat8之后get方式已经不会出现中文乱码了
>
> post方式在获取参数前需要修改获取的编码`request.setCharacterEndocing(“utf-8”)`将其修改为对应的中文编码

**请求转发**：在服务器内部进行资源跳转的方式

* 通过`RequestDispatcher getRequestDispatcher(String path)`获取转发器的对象
* 通过`RequestDispatcher.forward(request,response)`对象来进行转发

特点：请求转发是服务器**内部**进行资源跳转的方式，浏览器路径不会随着请求转发的情况而改变，实际上一次请求转发只有一次请求

共享数据：通过域对象进行数据共享

* 域对象：一个有作用范围的对象，可以在范围内共享数据
* Request域：代表**一次请求**的范围，请求转发到达的范围都是Request域的范围
  * `void setAttribute(String name,Object obj)`设置共享键值对
  * `void removeAttribute(String name)`删除共享键值对
  * `Object getAttribute(String name)`根据键获取值

共享数据仅限于**一次请求**范围内的共享，因此只能在请求转发时使用

> 用户登录案例
>
> * 编写login.html，需要有username和password两个属性
> * 使用Druid数据库连接池技术，操作mysql中的user表
> * 使用JDBCTemplate封装JDBC
> * 登陆成功跳转到SuccessServlet，失败跳转到FailServlet

步骤：

1. 写好login.html用于登陆，写好SuccessServlet和FailServlet用于判断之后跳转的界面
2. 创建JDBCUtils工具类用于快速获取DataSource，并用JDBCTemplate封装
3. 创建User和UserDao类，用于连接数据库，获取数据库中对应的信息
4. 创建LoginServlet，利用UserDao类访问数据库，对比数据库中的数据并封装进User对象中，再使用请求转发将User转发到Success或Fail界面 

:exclamation:Attention：出现过错误`ClassNotFound`，解决此问题需要正确规划项目结构，同时将Tomcat的libs包改名为lib，否则无法正确获取依赖

### 响应消息

请求消息的数据格式：

* 响应行
  * 组成：协议/版本    状态响应码    状态码描述\(HTTP/1.1    200    OK\)
  * 响应状态码：服务器告诉客户端本次请求和响应的状态
    * 通常为三位数字
    * 1xx：服务端持续接受客户端消息，发送1xx询问是否发送完毕
    * 2xx：成功，通常为200
    * 3xx：重定向，302（浏览器向服务器请求资源，服务器说明资源在其他地方并发送302），304（访问缓存）
    * 4xx：客户端错误，404（客户端错误，请求路径无对应资源），405（请求方式无对应的方法）
    * 5xx：服务端错误，500（服务端内资源查找失败）
* 响应头
  * 通常格式为“头名称：值”
  * Content-Type：服务器告诉客户端本次响应体数据的格式以及编码（`Context-Type:text/html;charset=utf-8`）
  * Content-disposition：服务器告诉客户端以什么格式打开响应体（in-line：在当前页面打开，attachment;filename=xxx：以附件的形式打开，名为xxx）
* 响应空行
* 响应体

#### Response

设置相应消息

1. 设置响应行
   * 格式：HTTP/1.1    200    OK
   * 设置状态码：`setStatus(int sc)`
2. 设置响应头：`setHeader(String name,String value)`
3. 设置响应体：
   * 获取输出流：`PrintWriter getWriter()`字符输出流，`ServletOutputStream getOutputStream()`字节输出流
   * 使用输出流：将数据输出到客户端浏览器
   * 使用输出流之前一定要先设置浏览器的编码`response.setContentType(“text/html;charset=utf-8”)`，或者通过`setCharactorEncoding()`的方式设置编码

#### 验证码操作

本质上是图片，目的是为了防止恶意注册

### ServeletContext

代表整个web应用，可以和程序的容器进行通讯，可以通过`request.getServletContext()`或`this.getServletContext()`方式获取

功能：

* 获取MIME类型
  * 在互联网通信过程中定义的一种文件数据类型
  * `String getMimeType(String file)`获取MIME类型，大类型/小类型（text/html）
* 域对象
  * `getAttribute()`
  * `setAttribute()`
  * `removeAttribute()`可以通过这三种设置Attribute数据
  * 范围是所有用户所有请求
* 获取文件在服务器的真实路径
  * `getRealPath()`，可以根据web目录下的资源路径获得在资源在服务器的真实路径

### JavaBean

BeanUtils工具类，用于简化数据的封装。减少使用apache的BeanUtils，通常使用Spring的代替。

要求：

* 类必须被public修饰
* 必须提供空参的构造器
* 成员变量必须使用private修饰
* 提供公共的getter和setter方法

概念：

* 成员变量和属性是不同的，成员变量是手动定义的，而属性是getxxx和setxxx后面的xxx
* 即属性是getter和setter方法截取后的名字
* `BeanUtils.setProperty(Object,name,value)`，传入一个Object，此方法可以根据name查找对应的**属性**，并为属性赋值
* `BeanUtils.getProperty(Object,name)`传入一个Object，通过name获取对应的属性
* `BeanUtils.populate(Object obj,Map map)`将map集合的键值对信息直接封装到对应的JavaBean对象中

## 会话技术

会话：一次会话中包含多次请求和响应，浏览器第一次给服务器资源发送请求，会话建立，直到某一方断开

在一次会话的范围内，多次请求之间数据会共享

### Cookie

客户端会话技术：会将数据存储到客户端

使用流程：

1. 创建Cookie对象，绑定数据：`new Cookie(String name, String value)`
2. 发送Cookie对象：`response.addCookie(Cookie cookie)`将Cookie添加进response对象中，数据会出现在请求头
3. 获取Cookie对象：`Cookie[] request.getCookies()`，通过遍历获取Cookies集合

原理：基于响应头set-cookie和请求头cookie实现

#### 细节

* 一次可以发送多个Cookie，可以通过多次调用addCookie\(\)方法来添加cookie进response
* cookie默认当浏览器关闭后自动销毁，可以通过设置**持久化存储**来进行保存
  * `setMaxAge(int seconds)`：正数以秒为单位，零是删除，负数是默认值
* 在tomcat 8之前，cookie需要**转码**才能传输中文，通常使用URL转码`URLEndocing()`和`URLDecoding()`，tomcat 8之后就支持中文数据的传输了
* cookie的共享问题
  1. 同一个tomcat服务器中部署了多个web项目：默认情况下cookie**不能共享**，但是可以通过`setPath(String path)`设置cookie的获取范围，只需将path设置为“/”即可实现共享
  2. 不同的tomcat服务器：通过设置一级域名也可以共享cookie`setDomain(String path)`，如果一级域名相同，则多个服务器之间也可以共享cookie
* cookie存储数据在客户端本地，对于单个cookie的大小限制一般为4kb，同一个域名下的cookie也有20个的数量限制
* 通常cookie可以在不登录的情况下完成服务器对客户端的身份识别，或用于存储少量的不太敏感的数据

### Session

服务端会话技术：会将数据存储到服务端

原理：Session依赖于**Cookie**

1. 当浏览器第一次获取Session时，会在内存中创建一个新的Session对象并赋予对应的id。
2. 此时服务器返回的响应头中会携带一个`set-cookie:JSESSIONID=id`用于记录Session对应的id，将其保存在本地的Cookie中
3. 此后访问网页都会根据Cookie中携带的SessionID，在服务器的内存中查找对应的Session对象，只要确保ID相同，就能共享Session。

使用流程：

1. 通过HttpSession获取session：`request.getSession()`
2. 设置Session的Name以及Value：`getAttribute(String name)`&`setAttribute(String name, Object value)`&`removeAttribute(String name)`

#### 细节

1. 若客户端关闭，服务器不关闭，则默认两次获取的Session不相同
   * 如果需要相同，则可以通过创建Cookie`Name=JSESSIONID，Value=Session.getid()`来获取对应的SessionID，让Session持久化保存
2. 若客户端不关闭，服务器关闭，则默认两次获取的Session也不相同
   * 重启服务器之后重新申请Session的地址空间也会不相同，但是需要确保**Session保存的数据相同**
   * Session的钝化：在服务器正常关闭之前，将Session对象序列化到硬盘上进行保存
   * Session的活化：在服务器启动之后，自动将Session文件转化为硬盘上的Session对象，并销毁Session文件
   * 钝化和活化Tomcat服务器可以帮我们自动完成，idea中无法完成活化
3. Session的销毁
   * 服务器正常关闭时自动销毁
   * Session调用`invalidate()`时会自动销毁
   * Session默认失效时间为30分钟，可以在Web.xml中修改`session-config`

Session是用于存储**一次会话**的**多次请求**的数据，且保存在客户端，且**没有大小限制**

## JSP

Java Server Pages 简称Jsp，可以同时在一个文件中定义html标签内容以及java代码，用于简化书写，本质上还是Servlet

### 脚本

脚本即是定义Java代码的方式，在Jsp中有三种标签可以用于定义脚本：

1. `<% code %>`：在此标签中定义的Java代码，会出现在**Service**方法中，格式要求同Service中的要求
2. `<%! code %>`：在此标签中定义的Java代码，会出现在Jsp编译后的Java类的成员变量的位置
3. `<%= code %>`：在此标签中定义的Java代码，会输出到页面上，格式要求同Html

### 内置对象

在Jsp页面中不需要创建和获取就可以直接使用的对象共有9个，被称为Jsp的内置对象

* pageContext：类型为`pageContext`，可在**当前页面共享数据**
* request：类型为`HttpServletRequest`，**一次请求访问的多个资源可以共享**，可以在资源转发范围内进行共享
* session：类型为`HttpSession`，**一次会话的多个请求间共享**
* application：类型为`ServletContext`，**所有用户间共享数据**
* 以上四个为域对象，以下五个为普通对象
* response：类型为`HttpServletResponse`，**响应对象**，可以通过`response.getWriter()`输出数据到页面
* page：类型为`Object`，相当于this
* out：类型为`JspWriter`，**字符输出流对象**，可以将数据通过`out.write()`输出到页面上。`out`和`response`的区别在于，response对象的处理永远优先于out对象，因此无论代码结构如何，response的输出永远先于out
* config：类型为`ServletConfig`，Servlet的配置对象
* exception：类型为`Throwable`，仅在`isErrorPage`为true时可使用，用于打印出现的错误

### 指令

指令是用于配置Jsp的配置信息，用于导入资源文件，类似Html的标签体属性

```text
<%@ 指令名称 属性名1=属性值1 属性名2=属性值2 ... %>
```

通常有三种指令：

1. Page：用于配置Jsp的页面
   * `<%@ page ContentType="text/html,charset=utf-8" %>`
   * `ContentType`，类似`response.setContentType()`可以设置文件的编码，设置响应体的mime类型以及字符集
   * `buffer`，缓冲区，jsp中out对象的缓冲区大小，通常为8kb，可以修改
   * `import`，导入语言包，例如java.util，导入的包会出现在Servlet生成的类中
   * `errorPage`，当前页面发现错误之后会跳转到的页面，同款还有一个`isErrorPage`，用来标记本页面是否为错误页面，标注之后可以使用**exception**对象
2. include：页面包含的，**导入页面的资源文件**
   * `<%@ include file="xxx.jsp" %>`
   * 可以用于导入其他Jsp页面文件，减少重复工作
3. taglib：用于导入资源
   * `<%@ taglib prefix="缩写" uri="xxxxx" %>`
   * 可以根据uri导入外部的标签库，使用缩写来代指导入的标签库

