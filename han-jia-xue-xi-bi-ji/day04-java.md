# Day04-Java

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

