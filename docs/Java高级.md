## 一、网络编程

### （一）基本概念

**网络：**两台/多台设备通过物理设备连接起来。

**网络编程：**数据通过网络从一台设备传输到另一台设备。

`java.net`包提供了一系列类、接口，用于实现网络通信。



**ip地址：**唯一标识网络中的每一台计算机。

`ipv6`的长度是`ipv4`的四倍，为128字节，并用16进制表示。

![](img/ipv4.png)

**域名：**www.baidu.com

**端口号：**标识计算机上某个特定的网络程序。（网络开发中，尽量不要使用 $0-1024$ 的端口，因为他们基本已经被知名的服务占用）



**协议：**

![](img/OSI.png)



**TCP：**

1. 传输控制协议；
2. 先建立TCP连接才能使用，传输前采用“三次握手”的方式，是可靠的；
3. TCP协议间通信的两个进程：客户端、服务端；
4. 在连接中可进行大数据量的传输；
5. 传输完毕后需要释放连接，效率低。

**UDP：**

1. 用户数据报协议；
2. 数据、源、目的三者封装成数据包，无需建立连接，是不可靠的；
3. 数据报大小限制在$64K$以内；
4. 无连接，速度快。



### （二）InetAddress

```java
// 获取本机的InetAddress对象
InetAddress localHost = InetAddress.getLocalHost();
System.out.println(localHost);  // DESKTOP-U75NRR3/192.168.137.1

// 根据指定主机名获取InetAddress对象
InetAddress host = InetAddress.getByName("www.baidu.com");
System.out.println(host);   // www.baidu.com/110.242.68.4

// 根据InetAddress对象获取域名、ip地址
String name = host.getHostName();
String address = host.getHostAddress();
System.out.println(name + "\n" + address);
// www.baidu.com
// 110.242.68.4
```



### （三）Socket

`Socket`是两台机器间通信的端点，网络通信实际上就是`Socket`通信。

它允许程序把网络当场一个流，数据在两个`Socket`之间通过`IO`流传输（`socket.getInputStream()`和`socket.getOutputStream()`）。



#### 1. TCP编程

##### （1）字节流

**客户端：**

```java
public class SocketTCPClient {
    public static void main(String[] args) throws IOException {
        // 连接本机的9999端口
        Socket socket = new Socket(InetAddress.getLocalHost(), 9999);
        System.out.println("Client linking 9999 port....");

        // 发送消息
        OutputStream oStream = socket.getOutputStream();
        oStream.write("Hello,server!".getBytes());
        socket.shutdownOutput();    // 一定要加这句，表示发送结束，否则会卡住

        // 接收消息
        InputStream iStream = socket.getInputStream();
        byte[] buf = new byte[1024];
        int len = 0;
        while ((len = iStream.read(buf)) != -1) {
            System.out.println(new String(buf, 0, len));
        }

        // 关闭
        oStream.close();
        iStream.close();
        socket.close();
        System.out.println("客户端退出");
    }
}
```

**服务端：**

```java
public class SocketTCPServer {
    public static void main(String[] args) throws IOException {
        // ServerSocket可以创建多个socket，被多个客户端连接
        // 9999端口不能被占用
        ServerSocket serverSocket = new ServerSocket(9999);

        // 没有客户端连接9999端口时会阻塞，否则会返回socket对象
        Socket socket = serverSocket.accept();
        System.out.println("Server listening 9999 port...");

        // 接收消息
        InputStream iStream = socket.getInputStream();
        byte[] buf = new byte[1024];
        int len = 0;
        while ((len = iStream.read(buf)) != -1) {
            System.out.println(new String(buf, 0, len));
        }

        // 回复消息
        OutputStream oStream = socket.getOutputStream();
        oStream.write("Hello,client!".getBytes());
        socket.shutdownOutput();    // 一定要加这句，表示发送结束，否则会卡住

        // 关闭
        iStream.close();
        oStream.close();
        socket.close();
        serverSocket.close();
        System.out.println("服务端退出");
    }
}
```



##### （2）字符流

需要使用转换流，将字节流转为字符流。

**客户端：**

```java
public class SocketTCPClient {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket(InetAddress.getLocalHost(), 9999);

        // 转换流：OutputStream 转 Writer
        OutputStream oStream = socket.getOutputStream();
        OutputStreamWriter osWriter = new OutputStreamWriter(oStream);
        BufferedWriter writer = new BufferedWriter(osWriter);

        writer.write("Hello, Server!");
        writer.newLine();
        writer.write("嘤嘤嘤...");
        writer.flush();
        socket.shutdownOutput();    // 一定要加这行，否则服务端会一直等待接收信息

        // 转换流：InputStream 转 Reader
        InputStream iStream = socket.getInputStream();
        InputStreamReader isReader = new InputStreamReader(iStream);
        BufferedReader reader = new BufferedReader(isReader);

        String str = reader.readLine();
        System.out.println(str);

        reader.close();
        writer.close();
        socket.close();
        System.out.println("客户端退出");
    }
}

```

**服务端：**

```java
public class SocketTCPServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(9999);
        Socket socket = serverSocket.accept();

        // 转换流：InputStream 转 Reader
        InputStream iStream = socket.getInputStream();
        InputStreamReader isReader = new InputStreamReader(iStream);
        BufferedReader reader = new BufferedReader(isReader);

        String str = "";
        while ((str = reader.readLine()) != null)
            System.out.println(str);

        // 转换流：OutputStream 转 Writer
        OutputStream oStream = socket.getOutputStream();
        OutputStreamWriter osWriter = new OutputStreamWriter(oStream);
        BufferedWriter writer = new BufferedWriter(osWriter);

        writer.write("Hello,client!");
        writer.newLine();
        writer.flush();

        writer.close();
        reader.close();
        socket.close();
        serverSocket.close();
        System.out.println("服务端退出");
    }
}
```



#### 2. UDP编程

```java

```

