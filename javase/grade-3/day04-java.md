# 集合

## 集合

集合类的特点：提供一种存储空间可变的存储模型，存储的数据容量可以随时发生改变（诸如ArrayList），同时**集合只能够存放引用类型**，例如，如果存放的是整数int，则需要的是Integer。

集合通过存储结构和存储特性的不同，可以分为以下几类：

* _Collection_（单列）：
  * _List_（元素可重复）：
    * **ArrayList**
    * **LinkedList**
    * **···**
  * _Set_（元素不可重复）
    * **HashSet**
    * **TreeSet**
    * **···**
* _Map_（双列）：
  * **HashMap**
  * **···**

_斜体_表示接口，**粗体**表示实现类

### Collection

单列集合的顶层接口，它表示一组对象，本身不提供任何直接实现，但提供更具体的子接口**Set**和**List**实现

#### Collection常用方法

Iterator：迭代器，集合的专用遍历方式

* `Iterator <E>`：返回此集合中**元素的迭代器**，通过集合的iterator\(\)方法得到
* 迭代器是通过Iterator\(\)方法得到的，因此是依赖于集合而存在的
* 常用的方法有`E next()`返回迭代中的下一个元素，`boolean hasNext()`判断是否有下一个元素

  ```java
  public class Demo {
      public static void main(String[] args) {
          Collection<String> c = new ArrayList<String>();
          c.add("Hello");
          c.add("world");

          Iterator<String> it = c.iterator();//依赖于集合存在
          System.out.println(it.next());
          System.out.println(it.hasNext());
          System.out.println(it.next());
          System.out.println(it.hasNext());
      }
  }
  ```

### List

List被称为**有序集合**或**序列**，可以通过索引来进行精确的插入或取值

与Set集合不同，List允许存在重复元素

由于继承自Collection，因此可以使用迭代器Iterator进行边路

#### List特有 方法

* `void add(int index, E element)`：在指定索引index处插入element
* `E remove(int index)`：返回并删除指定索引index处的元素
* `E set(int index,E element)`：修改指定索引index处的元素为element
* `E get(int index)`：返回指定索引index处的元素

以上方法索引都不能越界

**并发修改异常**

```java
//遍历一个集合，如果集合中出现了“java”这个元素，则在集合中添加"Hello"
public class Demo{
    public static void main(String[] args){
        List<String> l = new ArrayList<String>();
        l.add("Hello");
        l.add("world");
        l.add("java");

        Iterator<String> it = l.iterator();
        while(it.hasNext()){
            String s = new String(it.next());
            if(s.equals("java")){
                l.add("Hello");//并发修改异常异常：java.util.ConcurrentModificationException
            }
        }
    }
}
```

产生的原因：modCount（实际修改集合的次数）和expectedModCount（预期修改集合的次数）不一致。

在**ArrayList的内部类Itr类**中，创建迭代器时会初始化`modCount = expectedModCount`。每次执行next\(\)时都需要判断`if (modCount==expectedModCount)`。在通过迭代器遍历的过程中，通过**集合的方式添加元素会调用add方法，而add方法会执行modCount++**。因此会导致modCount和expectedModCount数值不一致，抛出并发修改异常。

对于并发修改异常，可以通过for循环和get等方法来获取元素，并通过集合的add方法来添加

```java
//遍历一个集合，如果集合中出现了“java”这个元素，则在集合中添加"Hello"
public class Demo{
    public static void main(String[] args){
        List<String> l = new ArrayList<String>();
        l.add("Hello");
        l.add("world");
        l.add("java");



        for(int i=0;i<l.size();i++){
            String s = l.get(i);
            if(s.equals("java")){
                l.add("Hello");//虽然同样是通过add方法添加元素，但是并没有使用迭代的Itr类，因此不会触发并发修改异常
            }
        }
    }
}
```

**ListIterator**

ListIterator：列表迭代器

* 通过`public interface ListIterator()`方法得到，是List特有的迭代器
* 允许程序员沿任意方向进行遍历，可以向前也可以向后
* 常用方法
  * `E next()`向后遍历
  * `E previous()`向前遍历
  * `boolean hasNext()`下一元素是否为空
  * `boolean hasPrevious`上一元素是否为空\
  * `void add(E e)`将指定的元素插入列表

:exclamation:ListIterator迭代器`void add(E e)`在修改进行添加的时候，**不会触发并发修改异常**，因为在源码中ListIterator含有重写的add方法，重写的add方法在添加元素的时候会将modCount和expectedModCount同步，因此不会出现两个数值不同的情况

#### ArrayList

ArrayList是List接口的可调整大小的数组实现

特点：**查询快，增删慢**

#### LinkedList

LinkedList是List接口的链表实现

特点：**查询慢，增删快**

以上两种List集合都能够用**Iterator迭代器、for循环、增强for循环**的方法来进行遍历，可以根据需求的不同来选择不同的集合

```java
public class Demo{
    public static void main(String[] args){
        ArrayList<String> array = new ArrayList<String>();

        array.add("Hello");
        array.add("World");
        array.add("Java");

        //Iterator迭代器
        Iterator<String> it = array.iterator();
        while(it.hasNext()){
            String s = it.next();
            System.out.println(s);
        }

        //for循环
        for(int i=0;i<array.size();i++){
            String s = array.get(i);
            System.out.println(s);
        }

        //增强for循环
        for(String s:array){
            System.out.println(s);
        }
    }
}
```

LinkedList集合有特殊的方法对元素进行操作：

* `public void addFirst(E e)`在开头插入元素
* `public void addLast(E e)`在末位插入元素
* `public E getFirst()`获取开头元素
* `public E getLast()`获取末位元素
* `public E removeFirst()`去掉头
* `public E removeLast()`去掉尾

### Set

Set是不包含重复元素的集合，同时**没有带索引的方法**，因此不能用索引遍历（但同样可以使用迭代器和增强for来进行遍历）

#### HashSet

实现Set接口，由**哈希表**支持，同时**对集合的迭代顺序不做保证**

特点：

* 底层的数据结构是**哈希表**
* 对集合的迭代顺序不做保证
* 没有使用索引
* Set集合不存在重复元素

**哈希值**

JDK会根据对象的**地址**或者**字符串**或者**数字**算出来的**int类型**的数值

在Object类中有`public int hashCode()`可以放回对象的哈希值，在默认情况下（使用Object类的`hashCode()`时），不同对象的哈希值是不同的。重写`hashCode()`方法也可以使哈希值相同

**哈希表**

哈希表采用的是**数组+链表**的形式进行存储，依赖于`hashCode()`和`equals()`方法进行判断

哈希表的存储过程如下：

现有六个字符串：“Hello”,”world”,”Java”,”你好“,“world”,”再见”。使用`hashCode()`方法可以获取到这六个字符串的哈希值

代码如下：

```java
public class Demo {
    public static void main(String[] args) {
        HashSet<String> hs = new HashSet<String>();
        hs.add("Hello");//Java,2301506
        hs.add("world");//world,113318802
        hs.add("Java");//Hello,69609650
        hs.add("world");//world,113318802
        hs.add("你好");//你好,652829
        hs.add("再见");//再见,682452

        for (String s:hs) {
            System.out.println(s+","+s.hashCode());
        }
    }
}
```

哈希表的存储空间大小一般为16，即索引为0-15的一个数组，数组的元素为链表。

1. 当读取到“Hello”时，首先计算其内容的哈希值`hashCode()`，为2301506，用哈希值对16取余得2，因此会    去哈希表中索引为2的位置查找。
2. 哈希表中索引为2的位置如果没有元素，则会在直接添加“Hello”，然后继续执行添加
3. 读取“world”，计算哈希值为113318802，对16取余得2，因此会去索引为2的位置查找
4. 由于索引为2的位置有元素，此时会判断**要添加的元素是否相等**`equals()`。元素不相同，因此会添加在索引为2的链表的末尾
5. 以此类推

在添加往哈希表中存储对象类型的元素时，由于创建的**每一个对象地址都不同**，而对象的内容有可能相同，因此需要在对象的类中**重写`hashCode()`方法和`equals()`方法**，来确保“内容相同的对象不会重复存储”。

#### LinkedHashSet

LinkedHashSet是由**哈希表和链表**实现的Set接口，具有**可预测的存取迭代次序**，用链表保证存取顺序，用哈希表保证元素唯一

#### TreeSet

TreeSet间接实现Set接口，其元素实现了**自然排序**，也可以根据比较器Comparator进行排序，取决于其构造方法

* 无参构造方法`TreeSet()`：根据元素的自然排序进行排序
* 带参构造方法`TreeSet(Comparator comparator)`：根据指定的比较器进行排序

**自然排序Comparable**

> 定义一个学生类，拥有姓名和年龄等基本属性
>
> 将其按顺序存入TreeSet集合中，并且按照年龄从小到大，姓名按照字母排序的方式进行排序

用TreeSet集合存储自定义的对象时，默认采用无参构造方法，也就是**自然排序**进行比较

自然排序，就是让**添加的元素所属的类实现Comparable接口**，同时要**重写**`compareTo()`

重写方法会根据返回的值，将新添加的元素插入到不同的位置

```java
public class Student implements Comparable<Student>{
    /*
    内容略过
    */
    private int age;

    @Override
    public int compareTo(Student s){
        //return 0;返回值为0时判断为相同元素，不插入
        //return 1;返回值为1时判断为大于，会在下方插入
        //return -1;返回值为-1时判断为小于，会在上方插入
        //int temp = this.age-s.age;//this代指当前插入的对象的值，s指前一个元素，可以通过这种方式来判断插入的顺序
        //return temp1;这种方式为从小到大，插入的元素大于前一个元素则插在后方，反之则插在前方
        int temp2=s.age-this.age;//这种方式为从大到校，插入的元素如果小于前一个元素则为正，插在后方，反之则插在前方
        return temp2;
    }
}
```

:alarm\_clock:值得一提的是，**String类也实现了Comparable接口**，因此String类也具有自然排序的功能

```java
String a = "a";
String b= "b";
b.compareTo(a);//会根据a-z A-Z的方式进行排序，此时b小于a，因此排在a的后面
```

**比较器Comparator**

调用Comparator需要在创建TreeSet时指定规则

`TreeSet(Comparator<? super E> comparator)`需要传入的值实际上是**Comparator接口的实现类**，通常可以采用匿名类的方式进行构造。

```java
public class Demo{
    TreeSet<Student> ts = new TreeSet<Student>(new Comparator<student>(){
        @Override
        public int compareTo(Student s1,Student s2){
            int num = s1.getAge()-s2.getAge();//此时的s1相当于this,s2相当于后一个插入的值
            int num2 = num==0?s1.getName().compareTo(s2.getName());
            return num2;
        }
    });
}
```

在重写方法时，一定要注意按照排序规则来重写`compareTo()`方法，注意主要排序条件和次要排序条件

### 泛型

泛型的本质是**参数化类型**，操作的数据类型被指定为一个参数，将原来的具体的类型参数化，然后在使用和调用时传入具体的类型

具有泛型类、泛型方法、泛型接口等不同地方

`<类型>`就是泛型的基本格式，

#### 泛型类

`public class Generic<T>`

#### 泛型方法

```java
public class Demo<T>{
    public void show(T t){
        System.out.println(t);
    }
}
```

#### 泛型接口

`public interface Generic<T>`

#### 类型通配符

为了表示各种泛型List的父类，可以使用类型通配符

* 类型通配符：`<?>`
* `List<?>`：表示元素类型未知的List，其元素可以匹配任何的类型
* 这种带通配符的List仅表示它是**各种泛型List的父类**，并不能把元素添加到其中

如果我们不希望`List<?>`是任何类型父类，而希望其局限于我们希望的某一类泛型List的父类，则我们可以使用**类型通配符的上限**，当然也存在**类型通配符的下限**

* 类型通配符上限：`List<? extends type>`
* `List<? extends Number>`：它表示的类型是**Number类，或是其子类型**
* 类型通配符下限：`List<? super type>`
* `List<? super Number>`：它表示的类型是Number或者其父类型

#### 可变参数

可变参数可以不局限方法的参数个数

* 表达方式：`public static void sum(int... a)`
* 实际上在可变参数中，a代表的是**数组**
* 如果一个方法具有多个参数，且其中一个参数为可变参数，则可变参数需要放在最后`public static void sum(int a,int... b)`

在其他场合也有可变参数的使用：

* Arrays工具类
  * 存在一个静态方法`public static <T> List<T> asList(T... a)`
  * 该方法返回的列表支持固定大小，不能做增删，只能修改
* List接口
  * 存在一个静态方法`public static <E> List<E> of(E... elements)`
  * 该方法返回包含任意数量元素的列表，不能做增删改操作
* Set接口
  * 存在一个静态方法`public static <E> Set<E> of(E... elements)`
  * 该方法返回包含任意数量元素的集合，参数内不能有重复元素，同时不能做增删改操作

### Map

`public interface Map<K,V>` K代表键的类型，V代表值的类型

#### 常用方法

* `V put(K key,V value)`根据键值对添加元素
* `V remove(Object K)`根据键删除值，返回的是值
* `void clear()`清空所有内容
* `boolean containsKey(Object key)`判断集合是否存在这个键
* `boolean isEmpty()`判断集合是否为空
* `int size()`集合的长度，即存在多少键值对
* `V get(Object key)`根据键返回值
* `Set<T> keySet()`获取所有键的集合，可以根据返回的键集合进行**遍历**
* `Collection<V> values()`获取所有值的集合
* `Set<Map.Entry<K,V>> entrySet()`返回Map中包含的映射的Set视图，即所有键值对对象的集合

#### HashMap

HashMap底层原理为哈希表，是Map接口的实现类，具有**自然排序**

### Collections

Collections是**针对集合操作**的工具类，是一个具体类

**常用方法**

* `public static <T extends Comparable<?super T>> void sort(List<T> list)`将列表按照**升序**进行排序
* `public static void reverse(List<?> list)`翻转列表元素顺序
* `public static void shuffle(List<?> list)`随机打乱元素顺序

> 使用ArrayList集合存储学生对象并根据年龄进行排序
>
> 两种方法，一是在Student类中实现Comparable接口并重写compareTo\(\)方法，二是在Collections.sort\(\)进行排序的时候指定Comparator（匿名类）

```java
//Student类
package xyz.eucaplee.practice;

public class Student {//重写compareTo方法需要 implements Comparable接口
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    /*
    @Override
    public int compareTo(Student s){
        int num = this.getAge()-s.getAge();
        return num;
    }
    */

}
```

测试类，采用的是第二种方法，即通过给`Collections.sort()`提供比较器来决定排序规则

```java
package xyz.eucaplee.practice;

import java.util.*;

public class Demo {
    public static void main(String[] args) {
        ArrayList<Student> list = new ArrayList<Student>();

        Student s1 = new Student("小明",28);
        Student s2 = new Student("小王",49);
        Student s3 = new Student("小李",20);
        Student s4 = new Student("小羊",24);

        list.add(s1);
        list.add(s2);
        list.add(s3);
        list.add(s4);

        for (Student s : list) {
            System.out.println(s.getName()+","+s.getAge());
        }
        Collections.sort(list, new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2) {
                int num = o1.getAge()-o2.getAge();
                return num;
            }
        });
        for (Student s : list){
            System.out.println(s.getName()+","+s.getAge());
        }

    }
}
```

