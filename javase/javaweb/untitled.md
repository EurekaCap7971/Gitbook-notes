# 注解

## Junit单元测试

Java中存在单元测试专用的包，搭配`@test`注解能够有效进行测试

Junit属于**白盒测试**，步骤如下：

1. 建立测试类/测试用例
   * 通常类名为被测试的类名+test
   * 存放该测试类的包名为xx.xx.xx.test
2. 定义可以独立运行的测试方法
3. 给该方法加入`@test`注解
4. 导入junit包

### @test

通常运行时不通过判断输出来判断正确与否，而是观察运行结果。运行的结果为绿色则是正常，红色则是失败。如果需要通过比较来判断是否正确的话，则可以使用**断言`Assert.xx()`**来判断结果与预期是否相等

### @Before&@After

在运行测试方法之前，如果需要进行初始化等操作，可以通过添加**@Before**注解来执行。

@Before代表初始化方法，通常用于资源申请。所有测试方法在执行之前都会执行此方法

@After代表释放资源方法，在所有测试方法执行完后，都会自动执行该方法

## 注解

在JDK1.5之后出现的新特性，是用于给计算机看的注释，可以帮助计算机判断代码功能是否符合注解的标记

根据作用可以进行分类：

* 编写文档：通过标识的元数据生成.doc文档
  * 可以通过`javadoc`的代码来生成对应的文档，例如@auther,@since
* 代码分析：通过标识的元数据对代码**使用反射**进行分析
* 编译检查：通过标识的元数据让编译器能够实现基本的编译检查，包括是否符合`@Override`等

### JDK常见注解

#### @Override

检测被该注解的方法是否是继承自父类/接口

#### @Deprecated

标记该方法已经过时

#### @SuppressWarnings

用于压制警告，需要传参，一般传递“all”来压制所有警告

### 自定义注解

有时候可以定义一个自定义注解来实现自己想要的目标

```java
元注解
public @interface name(){}
```

这种方式可以自定义一个名为`@name`的注解

注解本质上是一个接口，该接口默认继承了Annotation接口`extends java.lang.annotation.Annotation`

#### 属性

由于本质是接口，因此可以在注解中定义方法，一般注解中的方法被称为注解的属性

注解可以返回对应的数据类型：

* 基本数据类型
* String
* 枚举
* 注解
* 以上数据类型的数组

定义了属性之后，需要在使用注解的时候给属性赋值，也可以在定义属性时使用`default`设置默认值

```java
@Anno(age=12,name="LLH")
@Anno2()//age默认12
@Anno3(11)//只有一个属性，且属性名为value
```

当属性名为value时，可以直接进行赋值

### 元注解

用于描述注解的注解

#### @Target

描述注解能够作用的位置，其属性是Element\[\]数组，有三个取值

* Element.TYPE可以作用于类上
* Element.METHOD可以作用于方法上
* Element.FIELD可以作用于成员变量上

三者可以同时存在

#### @Retention

描述注解被保留的阶段，通常是RetentionPolicy.RUNTIME，运行时有效

#### @Documented

描述注解是否被抽取到api文档中

#### @Inherited

描述注解是否被子类继承

### 注解解析

前面可以通过反射+配置文件的方法来直接从class文件中获取方法和成员变量，这一功能也可以通过**解析注解**来实现

注解的属性可以定义为`className`和`methodName`等，然后通过解析注解可以获得注解属性的值

```java
@Anno(className="xxx.xx",methodName="xxx")
public class Demo(){
    public static void main(String[] args){
        //首先获取该类的class文件，然后从该类的class文件中解析注解
        Class<Demo> c = Demo.class;
        Anno an = c.getAnnotation(Anno.class);

        //从解析的注解当中可以获得注解的值
        String className = c.className();
        String methodName = c.methodName();
    }
}
```

