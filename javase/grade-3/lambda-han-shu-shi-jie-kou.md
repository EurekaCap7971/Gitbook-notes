# Lambda＆函数式接口

函数式思想：_要做什么，而不是怎么做_

## Lambda表达式

组成Lambda表达式的三要素：**形式参数、剪头、代码块**

表达式的格式

* \(形式参数\)-&gt;{代码块}
* 形式参数：如果有多个参数，用逗号隔开；无参留空即可
* -&gt;:代表指向动作
* 代码块：需要执行的动作

表达式适用前提：**有一个接口，且接口有且仅有一个抽象方法**

首先定义一个接口，仅有一个抽象方法

```java
public interface People {
    void eat();
}
```

然后定义测试类

```java
public class Demo{
    public static void main(String[] args){
        peopleEat(()->{
            System.out.println("人吃东西");
        })
    }

    private static void peopleEat(People e){
        e.eat();
    }
}
```

### Lambda省略模式

Lambda有很多省略模式：

* 在有参数的情况下，传入的参数类型可以省略，但是需要全部省略
* 如果只有一个参数，可以省略括号
* 如果代码块的语句只有一条，可以省略大括号和分号，有return时还可省略return

**注意事项**

使用Lambda时**必须有上下文环境**，才能推导出使用的接口是哪一个

如果使用Lambda表达式，则在编译字节码的时候不会生成字节码文件，程序会在运行时动态生成，使用匿名内部类时会产生一个.class文件

## 接口组成更新

在Java8之后，增加了**默认方法default**

在Java9之后，增加了**静态方法static**和**私有方法private**

### 默认方法

在接口中如果需要更新方法，会导致所有继承该接口的类都需要重写新增的方法，**默认方法**可以避免这个问题

关键词：`default`，使接口中的方法可以拥有方法体，可以被重写的同时不破坏接口的特性

### 静态方法

接口中的静态方法只能被接口直接调用，实际意义同普通的静态方法

关键词：`static`，可以通过接口名直接调用，如果接口A实现了接口B，且重写了静态方法，则调用方法时仍是使用

### 私有方法

接口中的私有方法同普通的而私有方法

关键词：`private`

## 方法引用

方法引用符`::`

```java
public interface Animal{
    void eat(String s);
}

public interface Demo{
    public static void main(Sting[] args){
        //使用Lambda表达式简化
        useAnimalEat(s->System.out.println(s));

        //使用方法引用
        useAnimaleat(System.out::println)
    }

    public static void useAnimalEat(Animal a){
        a.eat("吃吃吃");
    }
}
```

`::`该符号为引用运算符，所在的表达式被称为方法引用

Lambda表达式能省略的东西，**都可以通过方法引用进行改进**

方法引用会自动根据传入的参数进行推导

### 引用类方法

在Lambda表达式被类方法替代时，它的形式参数全部传递给静态方法作为参数

```java
public static void useConverter(Converter c){
    int num = c.convert("666");//实现定义Converter接口，内部定义了一个方法convert
    System.out.println(num)
}

//Demo
useConverter(s->Integer.parseInt(s));//把String类型的s通过Integer的静态方法转为int
useConverter(Integer::parseInt);//将s自动传入parseInt作为参数
```

### 引用对象实例方法

Lambda表达式被替代时，需要先创建该类的实例对象

```java
Person p = new Person();
System.out.println(p::ear;
```

同样，省略的所有形式参数都会传入实例方法作为参数

### 引用类实例方法

Lambda表达式被替代时，第一个参数作为调用者，后面的参数全部传入调用方法作为参数

### 引用构造器

引用构造器就是引用构造方法，Lambda表达式被替代时，所有参数传入构造方法

## 函数式接口

函数式接口：**有且仅有一个抽象方法的接口**，具体的体现就是Lambda表达式

函数式接口的官方认证：`@FunctionalInterface`注解，标注了这个接口内部**有且只能有一个抽象方法**

如果某个方法的参数是函数式接口，那么可以把**Lambda表达式作为参数进行传递**

同理，如果方法的返回值是函数式接口，那么可以把**Lambda表达式作为返回值返回**

#### 常用的函数式接口

**Supplier**

包含一个无参的方法`<T> get()`，用于获取数据

`Supplier<T>`接口也被称为生产型接口，如果制定了接口的泛型类型，那么其中的`<T> get()`方法就会返回什么类型的数据

```java
Supplier<String> supplier = () -> {
            return "xxx";
        };
System.out.println(supplier.get());
```

**Consumer**

`void accept(T t)`对给定的参数执行此操作

`default Consumer<T> andThen(Consumer<? super T> after)`返回一个组成的Consumer，依次执行此操作，然后执行after

`Consumer<T>`被称为消费型接口，消费的数据类型由泛型指定

```java
Consumer<String> consumer = (String s)->{
            System.out.println(s);
        };
consumer.accept("xxx");
```

**Predicate**

`test()`接受一个参数，对这个参数进行判断（规则由Lambda决定）

`default Predicate<T> negate()`逻辑否定，对应逻辑非

`default Predicate<T> and(Predicate other)`组合判断，对应短路与

`default Predicate<T> or(Predicate other)`组合判断，对应短路或

```java
Predicate<Integer> predicate = (Integer i ) ->{
            return i>9;
        };
System.out.println(predicate.test(7));//false
System.out.println(predicate.negate().test(7));//true
```

**Function**

`Function<T,R>`表示接受一个参数并产生结果的函数

## Stream流

Stream流通常与Lambda表达式一起使用

通常分为三种操作：**生成流-&gt;中间操作-&gt;终结操作**

#### 生成

Collection体系的集合可以使用默认方法`Stream()`生成流

Map体系可以通过集合间接的生成流

数组可以通过`Stream`接口的静态方法`of(T... values)`来生成

#### 中间操作

`filter(Predicate predicate)`对流中的数据进行判断，采用`Predicate`函数式接口

`limit(long maxSize)`截取前X个元素的数据并返回流

`skip(long n)`跳过前n个元素并返回流

`static<T> concat(Stream a, Stream b)`合并a和b为一个流

`distinct()`返回由该流去重后的流（是否重复根据`Object.equals(Object)`判断）

`sorted()`返回元素自然排序的流，`sorted(Comparator comparator)`返回根据特定规则排序的流

`<R> Stream<R> map(Function mapper)`返回由给定的函数应用于此流元素的结果组成的流

`IntStream mapToInt(ToIntFunction mapper)`返回一个`IntStream`其中包含将给定函数应用于此流的元素的结果

返回的`IntStream`是Int的原始流，可以调用`sum()`等方法对流中元素执行后续操作

#### 终结操作

`forEach(Consumer action)`对每个元素执行action操作

`toSet()`把流中数据返回到一个集合

`toList()`把流中数据返回到一个列表

`toMap()`把流中数据返回到一个Map，需要有键和值

