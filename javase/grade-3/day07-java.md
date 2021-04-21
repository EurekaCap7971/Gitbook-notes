# 网络编程

在网络通信协议下，实现网络互联的不同计算机上运行的程序间可以进行数据交换

三要素：

1. IP地址
2. 端口
3. 协议

## IP地址

### InetAddress

`InetAddress`类是一个与IP地址相关的类，该类**没有构造方法**，因此需要通过特殊的方式返回其实例

* `public static InetAddress getByName(String host)`通过传入主机名/IP地址，可以获得其InetAddress对象
* `public String getHostName()`得到的对象可以通过此方法返回主机名
* `public String getHostAddress()`返回文本显示中的IP地址字符串

```java
public class Demo {
    public static void main(String[] args)  {
        try {
            InetAddress address = InetAddress.getLocalHost();
            System.out.println(address);
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }finally {
            System.out.println("成功？");
        }
        /*
        DESKTOP-MQ1UT6L/192.168.44.1
        成功？
        */

    }
}
```

## 端口

端口从0-65535，1024以内为常用端口

## 协议

### UDP协议

用户数据报协议，此协议是无连接协议。

该协议消耗资源小，通常应用于传输**音频、视频和普通数据**。由于是无连接协议，因此不能确保数据的完整性，是不可靠传输。

#### UDP协议传输过程

1. 创建`DatagramSocket`类的对象，分别作为发送端和接收端，其中接收端需要绑定监听的端口
2. 创建`DatagramPacket`类的对象，并在构造方法中将要发送的数据打包
3. 发送端调用`send()`方法发送，接收端通过`receive()`方法接受
4. 关闭发送端和接收端

#### 示例

发送端代码

```java
public class sendDemo {
    public static void main(String[] args) throws IOException {
        DatagramSocket ds = new DatagramSocket();
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        String line;
        while((line=br.readLine())!=null){
            DatagramPacket dp = new DatagramPacket(line.getBytes(),line.getBytes().length,InetAddress.getByName("192.168.44.1"),1080);
            //line.getBytes().length是获取中文字符转化成byte数组之后的长度，如果沿用原先的字符串长度，会乱码
            ds.send(dp);
            if("886".equals(line)){
                ds.close();
                return ;
            }
        }
    }
```

接收端代码

```java
public class receiveDemo {
    public static void main(String[] args) throws IOException {

        DatagramSocket ds = new DatagramSocket(1080);
        DatagramPacket dp = new DatagramPacket(new byte[1024],1024);

        while(true){
            ds.receive(dp);
            System.out.println(new String(dp.getData(),0,dp.getLength()));
            if(new String(dp.getData()).equals("886")){
                ds.close();
                return;
            }
        }
    }
}
```

`DatagramPacker(byte[],length,Address,port)`是数据报的构造方法 ，在发送端时需要输入接收端的IP地址以及接收的端口，在接收端时只需要新建`byte[]`以及输入长度即可

**特别注意，在传输中文字符时，需要由于中文字符占用的字节数不同，因此需要注意编码以及获取长度的问题**

### TCP协议

传输控制协议，此协议是面向连接的协议。

该协议要求通信双方建立确认重传机制，在这种机制下衍生出了**“三握四挥”**。**上传文件、下载文件、浏览文件**等都是用TCP协议。

#### TCP协议传输过程

1. 创建客户端的`Socket`对象，通过`Socket(String host, int port)`初始化
2. 创建服务器的`ServerSocket`对象，通过`ServerSocket(int port)`初始化
3. 获取输出流`OutputStream getOutputStream()`
4. 通过监听`ServerSocket.accept()`获取Socket对象
5. 获取输入流`Socket.getInputStream()`，通过`read()`读取
6. 关闭对象

如果想要传输文件，则需要在客户端通过输出流写入文件内容，服务端通过输入流获取传输的内容。

在传输时，**双方需要知道什么时候文件传输结束，什么时候没结束**，因此需要自定义一个结束标记`shutdownOutput()`来告知服务端，输出已经结束，此时服务端就不会继续等待获取输入流了。

#### 示例

客户端示例

```java
public class sendDemo {
    public static void main(String[] args) throws IOException {
        Socket s = new Socket("192.168.44.1",10000);
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
        String line ;
        byte[] bys = new byte[1024];
        while ((line=br.readLine())!=null) {
            bw.write(line);
            bw.newLine();
            bw.flush();

            InputStream is = s.getInputStream();
            int len = is.read(bys);
            System.out.println(new String(bys,0,bys.length));

            if ("886".equals(line)) {
                break;
            }
        }
        s.close();
    }
}
```

服务端示例

```java
public class serverDemo {
    public static void main(String[] args) throws IOException {
        ServerSocket ss = new ServerSocket(10000);
        Socket s;
        byte[] bys = new byte[1024];
        s=ss.accept();//获取发送端Socket
        while(true){
            BufferedReader br = new BufferedReader(new InputStreamReader(s.getInputStream()));
            String line = br.readLine();
            System.out.println("服务端收到消息："+line);

            BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(s.getOutputStream()));
            bw.write("客户端发送了："+line);
            bw.newLine();
            bw.flush();
        }
    }
}
```

