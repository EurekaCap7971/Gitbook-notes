# 抽象类与接口

## 多态

多态指的是同一个对象在不同时刻表现出来的不同形态

多态的前提和体现：

* 有继承/实现关系
* 有方法重写
* 有父类引用指向子类对象

```java
public class Animal{
    private int age = 50;

    public void eat(){
        System.out.println("动物吃食物");
    }
}
public class Cat extends Animal{//有继承关系
    private int age=40;
    private int weight=30;
    @Override//有方法重写
    System.out.println("猫吃鱼");
}
---
public static void main(String[] args){
    Animal a = new Cat();//有父类引用指向子类对象
}
```

### 多态中成员访问特点

访问成员变量：**编译看左边，执行看左边**

访问成员方法：**编译看左边，执行看右边**

```java
//代码承上
public static void main(String[] args){
    Animal a = new Cat();
    System.out.println(a.age);//50
    System.out.println(a.weight);//报错
    a.eat();//猫吃鱼
}
```

原因：成员方法有重写，而成员变量没有

### 多态的好处和弊端

当使用多态时，可以提高程序的拓展性，减少重复性工作：定义方法时，如果使用**多态中的父类作为参数**，那么当创建了不同的子类时，传递进去的数据类型是父类的，但是执行的内容则是**子类重写过的**（访问成员方法）

弊端：由于设置了父类作为参数，在使用方法时则不能使用子类特有的功能

### 多态转型

向上转型：父类引用**指向**子类对象，从子到父

向下转型：父类引用**转为**子类对象，从父到子

```java
/*
Animal为父类，Cat为子类
*/
public static void main(String[] args){
    Animal a = new Cat();//向上转型
    a.eat();
    a.playgame();//由于父类引用中没有playgame()方法，因此会报错

    Cat c = (Cat) a;//向下转型
    c.playgame();//此时父类引用已经转化为子类，因此可以使用playgame()方法
}
```

**转型**必须是父类和子类之间的转型，如果是不同子类继承同一父类，那子类之间是没有办法相互转型的

```java
public static void main(String[] args){
    Animal a = new Cat();//a.adr=001
    a.eat();

    Cat c = (Cat) a;//c.adr=001
    c.eat();

    a = new Dog();//a.adr=002
    a.eat();

    Cat cc = (Cat) a;//Error，此时a.adr=002，Cat类的adr为001，继承同一父类的子类不可相互转型
}
```

## 抽象类

一个**没有方法体**的方法被称为**抽象方法**，一个**含有抽象方法**的类**必须被定义为抽象类**

抽象类不是具体的，因此**不能直接创建实例对象**

```java
public abstract class Animal{
    public abstract void eat();
}

public class Cat() extends Animal{
    @Override
    public void eat(){
        System.out.println("猫吃鱼");
    }
}

public class Demo{
    public static void main(String[] args){
        Animal a = new Animal();//报错，抽象类不可实例化
        Animal a = new Cat();//可以通过多态的方式实例化
    }
}
```

### 抽象类的特点

抽象类使用**abstract**关键字 `public abstract Animal(){···};`

抽象方法一定要在抽象类内，但是抽象类内不一定有抽象方法

抽象类虽然不能直接创建实例对象，但是可以**通过多态的方式创建对象**，被称为**抽象类多态**

子类需要重写父类中的抽象方法才能使用，但抽象类中的普通方法可以正常使用

抽象类的子类**要么重写继承的抽象方法**，**要么自身也是一个抽象类**

### 抽象类的成员特点

抽象类与类基本一致，有以下几点不同：

* 抽象类也有构造方法，但是不能实例化。构造方法仅用于**子类访问父类数据的初始化**
* 抽象类的抽象方法用于**限定子类必须完成某些动作**，普通方法则是提高代码的复用性

## 接口

接口属于**公共规范**，只要符合了规范，则可以通用

Java中的接口更多的体现在**对行为的抽象**

接口和类属于实现关系，类实现接口，可以多实现

接口与接口之间可以**多继承**

### 接口的特点

接口使用**interface**关键字修饰，`public interface Animal(){···};`

类使用**implements**实现接口，`public class Cat() implements Animal(){···};`

接口**不能实例化**，但可以**通过多态**实现类对象实例化，被称为**接口多态**

接口的实现类**要么重写抽象方法**，**要么自身是个抽象类**

### 接口的成员特点

接口中**没有成员变量**，**全部都是成员常量，且默认为静态成员常量**

接口中**没有构造方法**，实现类会默认**继承Object类**的构造方法

接口中的成员方法**只能是抽象方法**，所有方法前会默认修饰词 public abstract

### 接口和抽象类共同的特点

接口和抽象类都只能够通过多态调用自己原本定义的抽象类，但一般会有一个实现类同时继承抽象类和实现接口

因此一般在使用时，都会直接使用最简单的方式调用抽象方法——直接创建实现类的实例对象并调用

### 接口和抽象类的区别

抽象类主要对**类**进行抽象，包括**属性，行为**

接口主要对**行为**进行抽象，主要是**行为**

举个例子：

门常驻有**开**和**关**两种功能，因此放在抽象类中

**报警**属于额外功能，单独设计一个接口进行实现

如果有个门除了正常的开关之外，还有报警功能，则可以通过**接口**的方式实现

如果把报警功能放在**抽象类**中，则所有继承了**门**这一个抽象类的类，都需要实现报警功能，即使**没有这个需求**

```java
public interface Alarm(){
    void alarm();//接口实现报警功能
}
public abstract class Door(){
    public abstract open();//抽象类抽象开关方法
    public abstract close();
}
public class AlarmDoor extends Door implements Alarm{
    public void open(){···};
    public void close(){···};
    public void alarm(){···};
}
```

抽象类是对**事物**抽象，接口是对**行为**抽象

## 形参和返回值

### 抽象类的形参和返回值

抽象类名作为形参，需要使用多态的方式创建抽象类的实例对象，再将其作为形参传入，实际上需要的是**该抽象类的子类对象**

返回值同理，返回的是**抽象类的子类对象**

### 接口名作为形参和返回值

接口名作为形参，需要使用多态的方式创建实现类的对象，再将其作为形参传入，实际上需要的是**该接口的实现类对象**

返回值同理，返回的是**接口的实现类对象**

## 内部类

内部类就是**在一个类中定义另一个类**

内部类的访问特点：

* 内部类作为外部类的一个**成员变量**，可以直接访问外部类的所有内容
* 但是内部类作为一个类，外部类想要访问则必须**创建内部类的实例对象**

### 成员内部类

内部类根据为止不同，可以分为两种：

* 在类的成员位置：成员内部类

  ```java
  public class Outer{
      public class Inner{
          public void show(){···};
      }
  }
  ---
  public class Demo{
      Outer.Inner oi = new Outer().new Inner();//成员内部类的实例化方式采用“外部类名.内部类名”的方式进行实例化
      oi.show();
  }
  ```

  但是以上方法不常用，通常创建内部类的原因是**想要隐藏相关信息**，因此内部类的修饰符通常为**private**

  ```java
  public class Outer{
      private class Inner{
          public void show(){···};
      }

      public void method(){
          Inner i = new Inner();
          i.show();
      }
  }
  ```

  如果采用了private内部类的方式，则一般使用**外部类的public方法**访问内部类的属性

* 在类的局部位置：局部内部类

  外界无法直接访问局部内部类，通常在**局部方法内创建内部类的实例对象，借此访问局部内部类的方法**

  ```java
  public class Outer{
      private int num=20;
      public void method(){
          class Inner{
              public void show(){
                  System.out.println(num);
              }
          }
          Inner i = new Inner();
          i.show();
      }
  }
  ```

  局部内部类可以**访问外部类的成员**，也可以**访问方法内的局部变量**

* 匿名内部类：局部内部类的**特殊形式**

  ```java
  new 类名或接口名(){
      //重写方法
  }
  ```

  匿名内部类的本质**是一个继承了该类或实现了该接口的子类匿名对象**

  ```java
  //前置Inter接口
  public class Outer{
      public void method(){
          new Inter(){
              @Override
              public void show(){···}
          }.show();//作为一个匿名对象，可以直接使用.方法名的方式来调用方法

          Inter i = new Inter(){
              @Override
              public void show(){···}
          };
          i.show();//既然是匿名对象，也可以采用多态的方式用父类/接口引用指向子类/实现类对象
      }
  }
  ```

  **实际的使用**

  在实际的应用当中，如果创建操作类的对象，来调用操作某个类中的方法，会需要创建一个实现类或者是子类

  匿名内部类的返回值实际上就是一个实现类或者是子类，因此在调用的时候可以直接在操作类的方法中（某个需要传入类对象的作为参数的方法也可）直接创建匿名内部类，来完成对类的操作

  ```java
  public interface Jumpping{
      void jump();
  }

  public class JumppingOperator{
      public void method(Jumpping j){
          j.jump();
      }
  }

  public class JumppingDemo{
      public static void main(String[] args){
          JumppingOperator jo = new JumppingOperator();
          jo.method(new Jumpping(){
              @Override
              public void Jump(){
                  System.out.println("跳高高");
              }
          });
      }
  }
  ```

