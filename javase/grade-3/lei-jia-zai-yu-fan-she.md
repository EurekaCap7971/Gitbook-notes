# 类加载与反射

当程序要使用某个类，而此类还没有被加载到内存中时，系统会通过**类的加载、类的连接、类的初始化**这三个步骤来对类进行初始化。正常情况下JVM会连续执行这三个步骤，统称为**类加载/类初始化**

类的加载：

* 将class文件读入内存，并为其创建一个java.lang.Class对象，任何类被使用时都会创建这个对象

类的连接：

* 验证阶段：用于检验被加载的类是否有正确的内部结构，并且和其他类协调一致
* 准备阶段：准备为类的变量分配内存，并设置默认初始值
* 解析阶段：将类的二进制数据中的符号引用替换为直接饮用

类的初始化：

* 如果类没有加载，则系统优先加载并连接该类
* 如果直接该类的直接父类没有被初始化，则先初始化其直接父类
* 如果类中有初始化语句，则系统依次执行初始化语句

## 类加载器

负责将.class文件加载到内存中，并生成对应的java.lang.Class对象

### JVM的类加载机制

全盘负责：当加载一个Class时，该加载器会**同时负责加载该Class依赖的其他Class**，除非显示使用其他类加载器

父类委托：当加载某个Class时，先让**父类加载器试图加载该Class**，如果不行才尝试从自己的类路径中加载该类

缓存机制：保证所有加载过的Class都会被缓存，当加载某个Class对象时，类加载器会先从缓存区中搜索该Class，只有当缓存区中不存在该Class时，才会选择读取对应的二进制数据并进行加载

通常有三种加载器：

* Bootstrap ：JVM内置类加载器通常为null,父类为null,是所有加载器的祖先
* Platform Class loader：平台类加载器，是所有平台类加载器的祖先，父类为Bootstrap，实现类和JDK特定的运行时类
* System Class loader：应用程序类加载器，通常用语定义应用程序类路径、模块路径和JDK特定工具上的类

## 反射

每个类在创建对象时都有对应的.class文件，可以通过创建的.class文件来创建对应类的对象，这种方式被称为反射

### 获取Class对象

每个对象都有.class属性可以得知对象类型，也可以通过创建的对象调用`getClass()`来获取class

### 通过Class对象获取构造方法

`Class`类中用于获取对象的构造方法有特定的几个方法：

* 返回多个构造方法的数组：
  * `Constructor<?>[] getConstructors()`返回**所有公共构造方法对象的数组**
  * `Constructor<?>[] getDeclareConstructors()`返回**所有构造方法对象的数组**
* 返回特定构造方法：
  * `Constructor<T> getConstructor(Class<?>... param)`
  * `Constructor<T> getDeclaredConstructor(Class<?>... param)`

注意，此处传入方法的值是.class类型的对象，如果是String类型则需要传入String. Class，以此类推

返回的构造方法可以通过`newInstance()`来创建对应的Object对象，通过传入对应的参数可以调用不同的构造方法

如果要调用的构造方法为`private`类型，则可以通过`setAccessible()`设为true来**跳过访问检查**，这种方式被称为**暴力反射**

### 通过Class对象获取成员变量

与获取构造方法类似，`getFields()`及`getDeclaredFields()`等都可以返回`Fieleds[]`，或是返回单个`Field`对象

由于获取到的`Field`对象本身也是一个对象，因此它具有对应的方法可以初始化及设置该对象对应的值

```java
Class<?> c = Class.forName("xyz.eucaplee");//获取对应目录下的.class文件
Constructor con = c.getConstructor();//获得该类的构造方法
Object obj = con.newInstance();//利用构造方法创建实例
Field f = c.getField("name");//获得该类的成员变量的对象
f.set(obj,"LeeCap");//根据之前的对象调用set()来设施obj实例的成员变量的值
```

### 通过Class对象获取成员方法

`getMethods()`&`getDeclaredMethods()`可以获得`Method[]`，也可以获得单个`Method`

除了获取该类定义的方法之外，还能够获取**该类继承的方法**

```java
Class<?> c = Class.forName("xyz.eucaplee");
Constructor con = c.getConstructor();
Object obj = con.newInstance();
Method m = c.getMethod("setName");
m.invoke(obj,"LeeCap")
```

### 反射越过泛型检查

通常在决定了泛型的类型之后是不能够添加非泛型类型的元素的，但是可以通过反射来越过泛型检查

> 往`ArrayList<Interget>`里添加String类型的元素

```java
ArrayList<Integer> intArray = new ArrayList<Integer>();

intArray.add(10);
intArray.add(20);
intArray.add(30);

Class<? extends ArrayList> c = intArray.getClass();

Method m =c.getMethod("add", Object.class);
m.invoke(intArray,500);
m.invoke(intArray,"Hello");

System.out.println(intArray);//[10, 20, 30, 500, Hello]
```

### 配置文件

反射可以通过`className`以及`methodName`获取到对应的类名以及方法名，这种方式可以在**代码外部修改程序运行时要调用的方法**，一般把这种存放`className`和`mtehodName`的文件称为**配置文件**

首先可以创建一个配置文件class.txt，然后通过程序读取配置文件中写好的`className`&`methodName`来获取对应的类实例及方法，通过`Properties`可以从文件中读取对应的键值对，这样就能起到修改程序功能的作用

```java
/*
配置文件class.txt
className=
methodName=
*/
Properties prop = new Propertoies();
FileReader fr = new FileReader("xxx/class.txt");
prop.load(fr);
fr.close;

String className = prop.getProperty("className");
String methodName = prop.getProperty("methodName");

Class<?> c = Class.forName(className);//通过className获取类名
Constructor<?> con = c.getConstructor();//默认无参
Object obj = con.newInstance();

Method m = c.getMethod(methodName);//调用方法
m.invoke(obj);
```

同样也可以通过反射用配置文件的方式来设置成员变量

## 模块化

Java9才出现模块化内容，目的是为了使Java轻量化，减少需要加载的文件，提高速度

不同的模块在没有设置配置文件之前是不能够跨模块调用的，因此需要在每个模块内创建一个module-info.java文件来说明该模块**能够被其他模块使用的包**以及**该模块能够使用的其他模块的包**

`exports`+包名可以将本模块内的包提供给其他模块使用

`requires`+模块名可以导入其他模块提供的包

#### 模块服务

Java9允许将服务接口定义在某个模块中，针对该服务接口提供**不同的服务实现类**，在该模块的module-info.java文件中需要通过`provides 接口名 with 接口实现类`的方式向外提供接口服务

使用该服务的模块需要在module-info.java文件中使用`uses 接口名`语句来声明该服务接口

在声明了服务接口的模块内可以使用`ServiceLoader`方法来加载服务实现，会返回一个`ServiceLoader`类型的对象，该对象实现了`Iterable`接口，可以使用增强for进行遍历，其中每个元素的类型为`MyService`

通过`MyService.方法名`可以调用其他模块下的接口实现类重写的方法，这样可以使用其他模块提供的结构服务

