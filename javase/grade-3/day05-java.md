# IO流

## File

File是文件和目录路径名的**抽象表示**，文件和目录可以通过File封装成对象，具有多种构造方法

* `File(String pathName)`
* `File(String parent,String child)`
* `File(File parent,String child)`

以上三种方式都可以构建File对象，且由于是文件和目录路径名的抽象表示，因此及时路径名不存在也可以用File封装成对象并进行操作

File对象重写了toString\(\)方法，因此在程序中输出File对象时会输出对象的内容

#### 创建功能

File类具有三种创建方法，分别用于创建文件、目录、多级目录：

* `public boolean createNewFile()`：当文件名不存在时，会创建一个由提供的抽象路径名为名的新文件
* `public boolean mkdir()`：创建以提供的抽象路径名为名的目录
* `public boolean mkdirs()`：创建由抽象路径名为名的目录，父目录如果不存在也可以一并创建

返回的值用于判断是否已存在相关文件/目录，如果使用`mkdir()`创建多级目录也会返回false

#### 判断和获取

基本的判断，用于判断是否是文件、路径，是否存在等：

* `public boolean isDirectory()`：是否为目录
* `public boolean isFile()`：是否为文件
* `public boolean exists()`：抽象路径表示的File是否存在

获取名称：

* `public String getAbsolutePath()`：返回绝对路径字符串
* `public String getPath()`：将抽象路径转换成字符串
* `public String getName()`：获取抽象路径名表示文件或目录的名字

获取路径内的文件名字或File对象：

* `public String[] list()`：返回String数组，存储的是目录下的文件和目录名
* `public File[] listFile()`：返回File数组，用目录下的文件和目录创建File对象

#### 删除功能

`public boolean delete()`删除对应抽象目录下的文件或目录

## IO流

IO流根据数据流向不同有不同的分类：

* 输入流：读取数据
* 输出流：写数据

按照数据的类型来分：

* 字节流：字节输入流＆字节输出流
* 字符流：字符输入流&字符输出流

### 字节流

字节流的抽象基类有两个：InputStream和OutputStream，其子类名都是以其父类名作为子类名的后缀

#### OutputStream

`FileOutputStream(String name)`创建文件输出流，以指定的名称写入文件，其类中含有各种方法用于写入数据

* `write(int b)`将指定的字节写入输出流，一次传输一个字节数据，传输的是ASCII码
* `write(byte[] b)`将数组长度的字节从指定的字节数组写入输出流，一次传输一个字节数组数据
* `write(byte[] b , int off , int len)`将len字节从指定的字节数组开始，从偏移量off开始写入数组，一次写一个字节数组的部分数据
* 所有操作执行完之后都需要调用`close()`方法，关闭输出流对象并释放与此流相关的任何系统资源

```java
public static void main(String[] args){
    FileOutputStream fos = new FileOutputStream("F:\\Desktop\\Java.txt");
    /*
    FileOutputStream在创建对象的时候需要提供一个目标，用于指定输出流的目的地
    在创建时有三个步骤：
        1、调用系统功能创建文件
        2、创建字节输出流对象
        3、将字节输出流对象指向创建好的文件
    */
    fos.write(97);//将ACSII码 97的字节传入，最后在文件中的结果是“a”
    fos.close();//完成写入之后需要关闭输出流对象
}
```

#### InputStream

`FileInputStream(String name)`创建文件输入流，可以用`read()`等方法读取指定路径的数据

* `int read(byte[] b)`读取对应字节数组长度的数据，返回**所读的字节长度**
* 有时候读取的字节数组长度超过了剩余字节的长度，可以通过`String(byte[] b ,int offset, int length)`方法来将**已读字节长度**转化成String字符，防止上一次读取的缓存影响本次的输出

```java
public class Demo{
    public static void main(String[] args){
        File f = new File("xxxx");
        FileInputStream fis = new FileInputStream(f);
        FileOutputStream fos = new FileOutputSTream(new File("xxxxx"));
        byte[] bys = new byte[1024];//用于接受读取的字节数组
        int len;//用于记录字节数组长度
        while((len=fis.read(bys))!=-1){//当字节数组的长度不为-1时，说明还有内容可以读取，由于每次只能读取特定字节数组bys大小的内容，因此需要重复读取
            fos.write(bys,0,len);//通过OutputStream写入相关文件
        }
    }
}
```

#### 字节流写数据存在的小问题

字节流写换行需要手动添加换行符，但是不同的系统换行符不同：

* Windows：`/r/n`
* Linux：`/n`
* Mac：`/r`

追加写入需要在创建输出流对象的时候额外添加一个参数：`FileOutputStream(String name, boolean append)`，第二个元素决定写入数据时是从头写还是从末尾开始写

#### 字节缓冲流

缓冲流存在的含义是使每次进行读取或写入操作时，可以将数据从输入输出流中直接写入和读取，减少底层系统的调用。字节缓冲流**仅提供缓冲区**，实际的写入和读取操作还是要依靠基本字节流对象进行

`BufferOutputStream`：缓冲输出流，使应用程序能够向底层输出流写入字节，而不必为写入的每个字节导致底层系统的调用，构造方法为`BufferOutputStream(OutputStream)`

`BufferInputStream`：缓冲输入流，创建一个内部缓冲区数组，当从流中读取或跳过字节时，内部缓冲区会根据需要从包含的**输入流**中重新填充**大量字节**，构造方法为`BufferInputStream(InputStream)`

```java
public class Demo{
    public static void main(String[] args){
        File f = new File("xxxx");
        BufferedInputStream bis = new BufferedInputStream(f);
        BufferedOutputStream bos = new BufferedOutputSTream(new File("xxxxx"));
        byte[] bys = new byte[1024];//用于接受读取的字节数组
        int len;//用于记录字节数组长度
        while((len=bis.read(bys))!=-1){//当字节数组的长度不为-1时，说明还有内容可以读取，由于每次只能读取特定字节数组bys大小的内容，因此需要重复读取
            bos.write(bys,0,len);//通过OutputStream写入相关文件
        }
    }
}
```

### 字符流

在Java中，**字符流=字节流+编码表**

汉字在存储的时候，系统会根据其编码存储来决定他的字节个数。GBK编码的中文每个字占2个字节，UTF-8编码的中文每个字占3个字节。

不管是哪一种编码方式，汉字在转换成字节的时候第一个字节**一定是负数**，系统再通过编码方式来决定是读取负数后一位还是负数后两位，以便获取一个完整的汉字。

字符流具有两个抽象基类：

* `Reader()`字符输入流的抽象类
* `Writer()`字符输出流的抽象类

基于这两个类，字符流中还有和编码解码问题相关的两个类：

* `InputStreamReader(InputStream Input)`
* `OutputStreamWriter(OutputStream Output)`

#### OutputStreamWriter

`OutputStreamWriter`的构造函数需要把`OutputStream`的实例对象作为参数传入

在字符流中也有类似字节流的写入数据的方式：

* `void write(int c)`
* `void write(char[] cbuf)`
* `void write(char[] cbuf, int off, int len)`
* **`void write(String str)`**可以直接写入字符串
* **`void write(String str ,int off ,int len)`**写入部分字符串

由于OutputStreamWriter方法写入的是缓冲区，因此调用`write()`方法时，不能直接写入文件，需要调用**`flush()`**方法刷新缓冲，才能把数据写入文件

在调用**`close()`**关闭连接时，也会**先进行刷新**，再关闭连接

#### InputStreamReader

`InputStreamReader`的构造函数需要把`InputStream`的实例对象当做参数传入

字符流的写入方法也与字节流类似：

* `int read()`
* `int read(char[] cbuf)`

#### 简化类

`InputStreamReader`拥有一个简化类`FileReader(String fileName)`，是用于读取字符文件的便捷类

同样的，`OutputStreamWriter`也拥有一个简化类`FileWriter(String fileName,boolean append)`是作为写入字符文件的简化类

以上两种简化类都是**默认本地编码不变**的情况下，用于简化书写的类。如果碰到了编码不同的情况，还是需要使用原生类`InputStreamReader`和`OutputStreamWriter`

#### 字符缓冲流

字符流也同样拥有缓冲流，用于缓冲字符，提供高效率的写入和读入

`BufferedReader(Reader in)`和`BufferedWriter(Writer out)`

#### 字符缓冲流特有功能

`BufferedWriter`：`void newLine()`写一行行分隔符，会根据系统属性自动定义分隔符

`BufferedReader`：`String readLine()`读一行文字，返回包含整行结果的字符串，不包括行终止符，如果已经到末尾，则返回null

## 特殊操作流

### 标准输入输出流

`System`存在特殊的**字节输入输出流**，`System.in`和`System.out`，他们都使用`final`进行修饰，因此可以通过`System.in`或`System.out`这种直接静态方式调用。

通常默认`System.in`是默认从键盘获取输入，而`System.out`是将内容输出到屏幕上。

这两个特殊的输入输出流是**一直打开着的，也就意味着不需要new生成对象和使用`close()`关闭流**

```java
public class Demo {
    public static void main(String[] args) throws IOException {
        InputStream is = System.in;//通过多态的方式实现InputStream
        InputStreamReader isr = new InputStreamReader(is);//可以通过转换流将字节输入流转化成字符输入流，方便输出
        BufferedReader br = new BufferedReader(isr);//也可以通过字符缓冲流的方式获取一行数据
        int len;
        System.out.println(br.readLine());
        //Scanner sc = new Scanner(System.in);系统也拥有一个建议的扫描类Scanner，可以直接将读取的字节流转化成字符流
        //System.out.println(sc.next());通过next()方法获取sc的内容
    }
}
```

### 打印流

**只负责输出数据，不负责读取数据**，拥有自己独特的方法

#### 字节打印流

`PrintStream`继承了`OutputStream`，用于打印字节流

* `print()`：该方法可以**不转码**，直接传递值
* `println()`：上述方法的换行版

#### 字符打印流

`PrintWriter`具有两种构造方法：

* `PrintWriter(String fileName)`可以使用指定的文件夹名创建一个新的对象，不需要自动刷新
* `PrintWriter(Writer out, boolean autoFlush)`out常规输出流，autoFlush如果为真，则`println`,`printf`,`format`方法将自动刷新输出缓冲区

### 对象序列化流

将对象保存到磁盘中，或者在网络中传输对象

该流会将**对象序列化**，通过将对象的类型、属性以及数据等一系列存储的信息转化为**字符序列**，并将其保存在文件中，可以实现对象的持久化。同样，还可以通过**反序列化**将字节从文件中读取回来，并还原成一个对象

如果想要实现类的序列化，还必须使**序列化的类继承`Serializable`接口**，如果不继承此接口，则无法使类序列化或反序列化。`Serializable`接口仅仅是一个**标记接口**，并没有任何需要重写的方法

#### ObjectOutputStream

继承自`OutputStream`，是一个字节输出流，用于将对象序列化

* `ObjectOutputStream(OutputStream out)`：创建一个写入指定的OutputStream的该类对象
* `void writeObject(Object obj)`：传入一个**继承了`Serializable`接口的对象**，将其序列化

#### ObjectInputStream

继承自`InputStream`，是一个字节输入流用于将对象反序列化

* `ObjectInputStream(InputStream in)`：创建一个写入指定`InputStream`的该类对象
* `Object readObject()`：读取对象并写入文件

#### 对象序列化流常见问题

> _当一个对象被序列化后，在被反序列化读取之前如果原类文件属性被修改会发生什么_

报错：`InvalidClassException`

有以下三种情况会触发此异常：

1. 当类的串行版本与从流中读取到类描述符的类型不匹配时会出现
2. 该类包含未知的数据类型时会出现
3. 该类没有可访问的无参构造函数时会出现

序列化运行时会自动与**每一个可序列化的类关联一个序列号**`serialVersionUID`，通常在反序列化中使用，便于验证序列化对象的发送者和接受和是否加载了与序列化兼容的对象的类。

所有可序列化的类都可以显示声明`serialVersionUID`，且该字段必须是`static`、`final`和`long`类型，并尽可能地使用`private`进行修饰，防止被修改。通过这种方式可以忽略类之间的细微差别，使其能够正常反序列化类对象

> _如果一个对象中的某个成员变量的值不想被序列化，该如何实现？_

在定义类时中，不想被序列化的值可以通过`transient`关键字进行修饰，被修饰的值不参与序列化

### Properties

`Properties`继承自`HashTable`，是一个Map集合类，表示一组持久的属性，可以**在流中写入和加载**

由于定义时没有定义泛型，因此无法使用泛型初始化，因此存入`Properties`对象的数据类型都为`Object`类型

#### 特有方法

* `Object setProperty(String key, String value)`设置集合的键值对，都为`String`类型。其底层调用`HashTable`的`put()`方法
* `String getProperty(String key)`通过键获取值
* `Set<String> stringPropertyNames()`从该属性列表中返回一个**不可修改的**键集，其中键及其对应值都为字符串

#### 与IO流结合的方法

* `void load(InputStream inStream)`从**输入字节流**读取属性列表
* `void load(Reader reader)`从**输入字符流**读取属性列表
* `void store(OutputStream out ,String comments)`将键值对写入表中，以适合于使用`load()`的格式写入**输出字节流**
* `void sotre(Writer writer , String comments)`写入键值对，以便于使用`load()`的格式写入**输出字符流**

## 总结

IO流主要分为两大类，一类是字节流，一类是字符流，各有不同的用处

* 字节流：用于传输字节等信息，可以用于**传输音频、视频**等文件，泛用性较强
  * 字节输入流InputStream:
    * FileInputStream：用于处理文件的字节输入流
    * BufferedInputStream：字节缓冲流，能够提高字节传输的速率，减少底层系统的调用
  * 字节输出流OutputStream:
    * FileOutputStream：用于处理文件的字节输出流
    * BufferedOutputStream：字节缓冲流
* 字符流：专门用于传输字符串等信息，在聊天平台等专门传输文字信息时效率较高
  * 字符输入流Reader：
    * InputStreamReader：用于处理字符串的字符输入流
      * FileReader：简化类，在涉及编码问题时仍需使用父类来处理
    * BufferedReader：字符缓冲流，具有特殊方法`readLine()`，按行读取字符串
  * 字符输出流Writer：
    * OutputStreamWriter：用于处理字符串的字符输出流
      * FileWriter：简化类
    * BufferedWriter：用于处理字符串的字符缓冲流，具有特殊方法`newLine()`,手动换行

