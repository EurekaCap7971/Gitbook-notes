# 线程

## 线程

单线程：进程如果只有单条执行路径，就被成为单线程程序

多线程：进程如果有多条执行路径，则成为多线程程序

### 多线程的实现方式

有两种方法实现：

#### 继承`Thread`类

* 定义一个类（如`MyThread`），并继承`Thread`类
* 在`MyThread`类中重写`run()`方法
* 创建`MyThread`类对象
* 启动线程

```java
public class Student extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(i);
        }
    }
}

public class Demo{
    Student s1 = new Student();
    Student s2 = new Student();
    /*
    s1.run();
    s2.run();这两种方法实际上还是类方法的直接调用，并没有启动线程
    */
    s1.start();
    s2.start();//要启动线程，必须调用类对象的start()方法
}
```

**Thread类常用方法**

重写`run()`方法是因为**它封装了线程执行的代码**，在直接调用的情况下相当于普通的方法调用

当调用`start()`方法时会启动线程，此时Java虚拟机会自动调用该线程的`run()`方法

`Thread`类创建的线程也有对应的名字，可以通过默认的`getName()`方法获取该线程的名字，同时还可以在创建线程对象时初始化线程名，**前提是继承该类的对象在构造方法中加上`super(name)`方法**，这样才能够正常设置线程名

`Thread`类还有一个`static Thread currentThread()`方法，可以返回当前所处线程的名字。由于是静态的所以可以通过类名直接调用

```java
public class Student extends Thread {

    public Student(){};

    public Student(String name){
        super(name);
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(getname()+","+i);
        }
    }
}

public class Demo{
    Student s1 = new Student("线程1");
    Student s2 = new Student("线程2");
    /*
    s1.run();
    s2.run();这两种方法实际上还是类方法的直接调用，并没有启动线程
    */
    s1.start();
    s2.start();//要启动线程，必须调用类对象的start()方法
    System.out.println(Thread.currentThread().getName());//main
}
```

**线程调度**

线程调度一般有两种模式：**分时调度**和**抢占式调度**，而Java采用的是**抢占式调度**

抢占式调度会根据线程的优先级分配CPU使用时间，优先级越高使用时间越长，可以通过`getPriority()`的方式返回该线程的优先级，也可以通过`setPriority(int newPriority)`的方式设置线程优先级

线程的最高优先级为**10**，最低优先级为**1**，默认创建的线程优先级为**5**

**线程控制**

线程有三个方法可以控制：

* `static void sleep(long millis)`使当前正在执行的线程暂停指定的**毫秒数**
* `void join`其他线程必须等待这个线程死亡才能进行，**需要进行异常捕获**
* `void setDaemon(boolean on)`将该线程标记为**守护线程**，如果当前运行的线程都是守护线程时，JVM会退出（并不是立即退出）

**线程的生命周期**

跟操作系统类似，拥有“创建→就绪→运行→阻塞→就绪···→运行→结束”的流程，需要等待CPU分配时间片

#### 实现`Runnable`接口

* 定义一个类`Demo`实现`Runnable`接口
* 在`Demo`类中重写`run()`方法
* 创建`Demo`类的对象
* 创建`Thread`类的对象，**把`Demo`类对象作为构造方法的参数传入**
* 启动线程

```java
public class Demo implements Runnable{
        @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(Thread.currentThread().getName()+":"+i);//由于不继承Thread类，因此需要调用类名.静态方法的方式来获取当前线程的名字
        }
    }
}

public static void main(String[] args){
    Student s1 = new Student();

    Thread t1 = new Thread(s1,"EU");//需要将继承了Runnable接口的类对象作为参数传入Thread类
    Thread t2 = new Thread(s1,"CK");

    t2.start();
    try {
        t2.join();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    t1.start();
}
```

相比于继承`Thread`类，实现`Runnable`接口的好处是能够使当前类继承其他父类，同时可以使用多个相同代码的程序**共同使用一个资源**，上述事例可以看出两个线程共用了传入的s1对象。

在多线程的实践中，这种方式可以实现数据的有效分离。

### 线程同步

出现线程不安全问题的三个基本条件：

1. 处于多线程环境
2. 具有共享的数据
3. 具有多条语句操控共享数据

如果要想转化为线程安全，就需要在这三个基本条件中下手

#### `synchronized`

```java
synchronized(Object Obj){
    //code
}
```

使用`synchronized`可以创建一个同步代码块，可以将多条操作语句通过**一个对象锁**将其锁起来，不同的线程通过**同一个对象锁**进行同步

优点：解决了多线程的安全性问题

缺点：多个线程同时运行时，由于存在同步锁，每个线程都需要判断并等待锁的释放，非常耗费资源

**同步方法**

将`synchronized`关键字在定义方法时添加：`synchronized type name()`，借由这种方法可以实现方法的同步，此时同步锁的对象为`this`对象

在同步静态方法时也是采用关键字修饰，但是静态方法同步锁的对象不是`this`，而是**类名的字节码对象**，也就是`className.class`

**线程安全的类**

线程安全的类都需要执行同步，因此在日常情况下没有使用这几种类，而是选择了线程不安全的、拥有同样功能的类：

* StringBuffer——StringBuilder
  * 线程安全，同样是可变字符序列
  * 从JDK5开始，被StringBuilder取代，后者不执行同步，速度更优
* Vector——ArrayList
  * 该类改进了List接口，且实现了同步
  * 如果不需要线程安全的实现，建议使用ArrayList
* Hashtable——HashMap
  * 该类实现了哈希表，映射键值对，且任何非null对象都可以用作键或是值
  * 该类实现了Map接口，且实现了同步，HashMap是线程不安全的同等替换

Vector和Hashtable在实际应用中有更优的替代品：

`static <T> List<T> Collections.synchronizedList(List<T> list)`返回由指定列表支持的线程安全列表

#### Lock锁

Lock作为一个接口，具有`lock()`方法和`unlock()`方法来获得锁和释放锁

`ReentrantLock`是Lock接口的实例类

### 生产者消费者问题

生产者消费者模式是经典的多线程模式

* 生产者：负责生产数据，存放到共享数据区域
* 消费者：负责获取数据，从共享数据区域中提取

为了同步生产和消费过程中的**等待**和**唤醒**操作，Object类提供了一些方法：

* `void wait()`使当前线程等待，直到另一个线程调用当前线程的唤醒方法
* `void notify()`唤醒正在等待对象监视器的**单个线程**
* `void notifyAll()`唤醒正在等待对象监视器的**所有线程**

因为施加了同步锁，如果在代码中想要使用等待和唤醒操作，需要在对应的方法前添加`synchronized`关键词来确保同步锁的对象为同一个对象

> 简易的生产—消费模型
>
> 生产者生产五瓶牛奶，消费者一旦发现有牛奶就消费牛奶

**Customer消费者类**，实现Runnable接口

```java
public class Customer implements Runnable{
    private Box b;

    public Customer(Box b){
        this.b=b;
    }

    @Override
    public void run() {
        while(true){
            b.get();
        }
    }
}
```

**Producer生产者**，实现Runnable接口

```java
public class Producer implements Runnable{
    private Box b;

    public Producer(Box b){
        this.b=b;
    }

    @Override
    public void run() {
        for (int i = 1; i <= 5; i++) {
                b.put(i);
            }
        }
}
```

**Box奶箱类**，作为多线程的共享数据区域，提供生产和消费功能

```java
public class Box {
    private int milk;
    private boolean state = false;
    //state用于记录奶箱内是否有牛奶

    public synchronized void put(int milk) {
        //判断是否有牛奶，如果有牛奶则等待消费
        if (state) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }


        this.milk = milk;
        System.out.println("送奶小哥往奶箱里放入第" + this.milk + "瓶牛奶");
        state=true;

        notifyAll();
    }

    public synchronized void get() {
        //判断是否有牛奶，如果没有牛奶则等待生产
        if (!state) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println("秃头大哥从箱子里拿了第" + this.milk + "瓶牛奶");
        this.milk--;
        state=false;

        notifyAll();
    }
}
```

**Demo测试类**

```java
public static void main(String[] args)  {
        Box b = new Box();
        Producer p = new Producer(b);
        Customer c = new Customer(b);

        Thread t1 = new Thread(p,"送奶小哥");
        Thread t2 = new Thread(c,"秃头小哥");

        t1.start();
        t2.start();
    }
```

