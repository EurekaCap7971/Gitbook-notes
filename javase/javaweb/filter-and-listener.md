# Filter&Listener

## Filter

当访问服务器的资源时，Filter过滤器可以将请求拦截下来，完成一些特殊的功能：

* 登录验证：多个页面需要登录验证才能访问时，可以将登录验证部分设置在过滤器中。
* 统一编码处理：各种请求编码不一，在页面获取请求时有时需要设置编码，也可以在过滤器中完成相关操作。
* ......

### 快速入门

1. 导入Filter包（`javax.servlet`）并实现Filter接口
2. 复写相关方法`doFilter()`。**注意Bug： Filter接口的`init()`方法如果调用了`Filter.super.init()`就会报错了，无法正常启动Filter。**
3. 配置拦截路径：可以通过注解进行配置拦截路径`@WebFilter()`，也可以通过`web.xml`来配置
4. 设置放行：`filterChain.doFilter(servletRequest,servletResponse)`

### 细节

1. web.xml配置
   * ```markup
     <filter>
         <filter-name>xxx</filter-name>
         <filter-class>xxx</filter-class><!-- 类的全路径 -->
     </filter>
     <filter-mapping>
         <filter-name>xxx</filter-name>
         <url-pattern>/*</url-pattern><!-- 拦截路径的设置 -->
     </filter-mapping>
     ```

* 类似于@WebServlet的xml设置，可以通过xml配置拦截器的类以及拦截路径

1. 过滤器执行流程
   * 在执行放行`filterChain.doFilter()`操作之前，可以对`request`对象进行增强
   * 执行`filterChain.doFilter()`之后，请求获取到对应的资源，获取到网页资源之后开始加载，同时`response`对象返回，此时可以对`response`对象执行增强操作
2. 过滤器生命周期
   * `init()`方法会在服务器启动之后自动创建Filter对象，在整个生命周期只会执行一次，通常用于加载资源
   * `doFilter()`方法在每次请求被拦截的时候都会执行，执行多次，
   * `destroy()`方法会在服务器关闭时销毁Filter对象，只执行一次，用于释放资源
3. 过滤器配置详解
   * 拦截路径配置：
     1. 具体资源路径：`/index.jsp`    只有访问index.jsp资源时才会执行过滤器
     2. 具体目录拦截：`/user/*`  访问/user下的所有资源时，都会执行过滤器
     3. 后缀名拦截：`*.jps`  访问后缀名为.jsp的资源时，会执行过滤器
     4. 无条件拦截：`/*`  拦截所有请求
   * 拦截方式配置：
     1. 注解配置`@WebFilter(dispatcherTypes={ Dispatcher.属性 , ...})`
        * REQUEST：默认值，浏览器直接请求时拦截
        * FORWARD：转发访问资源时拦截
        * INCLUDE：包含访问资源时拦截
        * ERROR：错误跳转资源时拦截
        * ASYNC：异步访问时拦截
     2. web.xml配置：可以在xml文件里配置`<dispatcherTypes>`标签来选择拦截方式
4. 过滤器链
   * 执行顺序：如果有两个及以上的过滤器，会以类似出入栈的方式执行
   * 过滤顺序：
     * 注解配置：按照类名的字符串比较规则执行，由小及大
     * web.xml：按照在xml文件中定义的顺序先后执行

### 实际使用

> ## 检测用户登录
>
> * 当用户访问相关页面时，如果没有登录，则会自动跳转到登录页面进行登录
> * 核心原理：登录时会将相关的用户信息存储在session或是cookie中，当访问网页的相关资源时，通过过滤器拦截下来并判断是否存储有相关用户信息
>   * 有：直接放行
>   * 没有：跳转至登录页面
> * 需要注意的是，登录页面也会请求一些数据，因此需要获取**请求路径**，如果请求路径来自于登录页面则直接放行

```java
public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain){
    //0.将ServletRequest强转成HttpServletRequest
    HttpServletRequest request = (HttpServletRequest) req;
    //1.获取资源请求路径
    String uri = req.getRequestURI();
    //2.判断路径是否包含登录相关资源路径，包含css,js,验证码Servlet等资源
    if(uri.contains(xxxx)||uri.contains("/js")||uri.contains("/fonts")||uri.contains("/css")){
        //包含登录相关路径，放行
    }else{
        //不包含，验证用户是否登录
        Object user = request.getSession().getAttribute("user");//通过Session获取User对象，此处可以替换
        if(user!=null){
            //已登录，放行
        }else{
            request.getRequestDispatcher("登录页面").forward(request,response)
        }
    }
}
```

> ## 敏感词汇顾虑
>
> * 对用户录入的数据进行敏感词过滤，首先获得用户输入的数据，将其与敏感词库进行对比，如果是敏感词汇，则将其替换
> * 核心原理：在过滤器中对`request`对象进行增强，采用设计模式——代理模式解决

代理模式简要介绍：

* 概念：通过`代理对象`代理`真实对象`，达到增强真实对象功能的目的
* 实现方式：
  * 静态代理：有一个类文件描述代理模式
  * **动态代理**：在内存中形成代理类
    1. 代理对象和真实对象实现相同的接口
    2. 通过`Proxy.newProxyInstance()`获取代理对象

       ```java
       public static void main(String[] args){
           //假设有一个真实对象类，名为User
           //1.创建真实对象类
           User user = new User();
           /*2.动态代理增强User对象
               三个参数：
               1、类加载器：真实对象.getClass().getClassLoader();确保是同一个类
               2、接口数组：真实对象.getClass().getInterfacs();确保实现相同接口
               3、处理器：new InvocationHandler()
              */ Object proxy_user = Proxy.newProxyInstance(User.getClass().getClassLoader(),User.getClass().getInterfaces(),new InvocationHandler() {
               /*
                   代理逻辑编写的方法，代理对象调用的所有方法都会触发该方法执行
                   三个参数：
                   1、proxy：代理对象
                   2、method：代理对象调用的方法被封装的对象
                   3、args：代理对象调用的方法时，传递的实际参数
               */
               @Override
               public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
                   //使用真实对象调用该方法，由于调用任何方法都会调用invoke方法，因此相当于用代理执行了真实对象的此方法，并传入原参数
                   Object obj = method.invoke(user,args);
               }
           });
       }
       ```

    3. 增强方法：
       * 增强参数列表
       * 增强返回值类型
       * 增强方法体逻辑
  * 

