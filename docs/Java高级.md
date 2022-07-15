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

**客户端和服务端均使用端口进行监听，客户端的端口是由`TCP/IP`来分配的随机端口，而服务端的端口是双方指定的。**



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



**控制台命令：**

`ipconfig`：查看主机的`ip`情况；

`netstat -an`：查看主机网络状况，包括端口监听情况和网络连接情况；

`netstat -anb`：查看主机网络状况，包括监听情况、网络连接情况、监听程序；

`netstat -anb | more`：可以分页显示；

`netstat -anb | find /i "8888"`：从结果中筛选包含`8888`关键字的项。

![](img/netstat.png)



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

数据、`ip`、对象等都被封装在`DatagramPacket`对象中，没有客户端、服务端之说，只有数据的发送端和接收端。



**发送方：**

```java
public class UDPSender {
    public static void main(String[] args) throws IOException {
        // 由于UDP既可以发送数据，又可以接收数据；
        // 而本程序的发送和接收端都在同一设备上；
        // 所以两边设置的端口最好不一样，便于区分。
        DatagramSocket socket = new DatagramSocket(9998);
        System.out.println("发送方监听9998端口....");

        String str = "bro,拉面以go贼?";
        // 由于本程序的发送和接收端都在同一设备上，所以可以使用 InetAddress.getLocalHost()
        // 但是在一般应用场景中，还是建议使用 InetAddress.getByName("")
        DatagramPacket packet = new DatagramPacket(str.getBytes(),
                str.getBytes().length,
                InetAddress.getByName("192.168.137.1"),
                9999);  // 注意发送中文时的长度，一定是经过编码后的长度
        socket.send(packet);

        // 接收回复
        byte[] data = new byte[1024];
        packet = new DatagramPacket(data, data.length);	// 这里一定要重新new一个，否则接收的数据长度不一定对
        socket.receive(packet);
        int len = packet.getLength();
        data = packet.getData();
        String msg = new String(data, 0, len);
        System.out.println("接收数据长度：" + len);
        System.out.println("接收回复：" + msg);

        socket.close();
        System.out.println("发送端退出");
    }
}
```



**接收方：**

```java
public class UDPReceiver {
    public static void main(String[] args) throws IOException {
        DatagramSocket socket = new DatagramSocket(9999);
        System.out.println("接收方监听9999端口....");

        byte[] bytes = new byte[64 * 1024]; // UDP包最大为64K
        DatagramPacket packet = new DatagramPacket(bytes, bytes.length);

        // 接收信息，填充到packet中
        // 如果没有数据，将会阻塞，一直等待接收数据
        socket.receive(packet);

        // 拆包
        int len = packet.getLength();
        byte[] data = packet.getData();
        String msg = new String(data, 0, len);
        System.out.println("接收数据长度：" + packet.getLength());
        System.out.println("接收数据：" + msg);

        // 回复消息
        String str = "走走走！";
        packet = new DatagramPacket(str.getBytes(),
                str.getBytes().length,
                InetAddress.getByName("192.168.137.1"),
                9998);
        socket.send(packet);

        socket.close();
        System.out.println("接收端退出");
    }
}
```



#### 3. 例题

##### （1）例题一

客户端向服务端发送一张图片，服务端返回消息。

**客户端：**

```java
public class Client {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket(InetAddress.getLocalHost(), 8888);

        // 创建读取文件
        String path = "e:/背景/5t5.jpg";
        BufferedInputStream biStream = new BufferedInputStream(new FileInputStream(path));
        byte[] bytes = StreamUtils.inputStreamToByteArray(biStream);

        // 获取网络输出流
        OutputStream oStream = socket.getOutputStream();
        oStream.write(bytes);
        oStream.flush();
        socket.shutdownOutput();

        // 接收回复信息
        InputStream iStream = socket.getInputStream();
        String str = StreamUtils.inputStreamToString(iStream);
        System.out.println(str);

        // 关闭流
        iStream.close();
        biStream.close();
        oStream.close();
        socket.close();
        System.out.println("客户端关闭...");
    }
}
```



**服务端：**

```java
public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(8888);
        Socket socket = serverSocket.accept();
        System.out.println("服务端在8888端口监听...");

        // 读入byte[]数据
        BufferedInputStream biStream = new BufferedInputStream(socket.getInputStream());
        byte[] bytes = StreamUtils.inputStreamToByteArray(biStream);

        // 输出到文件
        String path = Server.class.getResource("/").getPath() + "5t5.jpg";
        BufferedOutputStream boStream = new BufferedOutputStream(new FileOutputStream(path));
        boStream.write(bytes);
        System.out.println("服务端图片接收成功！");

        // 回复信息
        BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
        writer.write("服务端：收到!");
        writer.flush();
        socket.shutdownOutput();

        // 关闭流
        writer.close();
        boStream.close();
        biStream.close();
        socket.close();
        serverSocket.close();
        System.out.println("服务端关闭...");
    }
}
```



**工具类：**

```java
public class StreamUtils {

    /**
     * 输入流转byte[]
     *
     * @param iStream 输入流
     * @return byte[]
     * @throws IOException 输入输出异常
     */
    public static byte[] inputStreamToByteArray(InputStream iStream) throws IOException {
        ByteArrayOutputStream baoStream = new ByteArrayOutputStream();
        byte[] buf = new byte[1024];
        int len = 0;
        while ((len = iStream.read(buf)) != -1) {
            baoStream.write(buf, 0, len);
        }
        byte[] array = baoStream.toByteArray();
        baoStream.close();
        return array;
    }

    /**
     * InputStream转String
     *
     * @param iStream 输入流
     * @return 字符串
     * @throws IOException 输入输出异常
     */
    public static String inputStreamToString(InputStream iStream) throws IOException {
        BufferedInputStream biStream = new BufferedInputStream(iStream);
        byte[] buf = new byte[1024];
        int len = 0;
        StringBuilder str = new StringBuilder();
        while ((len = biStream.read(buf)) != -1) {
            str.append(new String(buf, 0, len)).append("\n");
        }
        return str.toString();
    }
}
```



##### （2）例题二

客户端向服务器发送要下载的文件名，服务端返回文件，客户端接收并保存。

（工具类`StreamUtils`同上例）

**客户端：**

```java
public class Client {
    public static void main(String[] args) throws IOException {
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入文件名：");
        String fileName = scanner.next();

        Socket socket = new Socket(InetAddress.getLocalHost(), 6666);
        OutputStream os = socket.getOutputStream();
        os.write(fileName.getBytes());
        os.flush();
        socket.shutdownOutput();

        // 读取返回的文件
        InputStream is = socket.getInputStream();
        byte[] bytes = StreamUtils.inputStreamToByteArray(is);

        // 保存文件
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(Server.class.getResource("/").getPath() + fileName));
        bos.write(bytes);

        // 关闭流
        bos.close();
        is.close();
        os.close();
        socket.close();
        System.out.println("客户端退出");
    }
}
```



**服务端：**

```java
public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(6666);
        Socket socket = serverSocket.accept();

        // 获取客户端需求
        InputStream is = socket.getInputStream();
        byte[] bytes = new byte[1024];
        int len = 0;
        String downloadFileName = "";
        while ((len = is.read(bytes)) != -1) {
            downloadFileName += new String(bytes, 0, len);
        }
        System.out.println("下载文件名：" + downloadFileName);

        // 从本地获取文件
        String path = "e:/背景/";
        File file = new File(path, downloadFileName);
        if (!file.exists()) {
            path += "5t5.jpg";
        } else {
            path += downloadFileName;
        }
        BufferedInputStream bis = new BufferedInputStream(new FileInputStream(path));
        byte[] data = StreamUtils.inputStreamToByteArray(bis);

        // 向客户端发送文件
        BufferedOutputStream bos = new BufferedOutputStream(socket.getOutputStream());
        bos.write(data);
        bos.flush();
        socket.shutdownOutput();

        // 关闭流
        bos.close();
        bis.close();
        socket.close();
        serverSocket.close();
        System.out.println("服务端退出");
    }
}
```





## 二、项目

[Java控制台即时通讯系统](https://github.com/WOC-BUG/Java-console-instant-messaging)

（注意，不同项目共用类进行网络传输时，该类所在的包名必须完全一致。因为序列化的对象会加上包名信息，直接使用会认为两者是不同的类。）



## 三、反射

### （一）程序的三个阶段

#### 1. 阶段图

![](img/编译、类加载、运行三阶段.png)

​      `Class.forName("全类名")`							`类名.class`											`对象.getClass()`



#### 2. Class类

* `Class`也继承自`Object`；

* `Class`类对象是由系统创建的，而不是`new`出来的。`new`和反射的方法创建对象，底层都会进行类加载：

  ![](img/ClassLoader.loadClass().png)

* 每个类都只会加载一次，在首次创建对象、调用类的静态方法/属性/代码块时；

* 通过`Class`对象可以得到一个类的所有信息；

* `Class`对象存放在堆，类的字节码二进制数据存放在方法区，又称为元数据。



#### 3. 获取`Class`类对象

##### （1）Class.forName("类的全类名")

已知类的全类名，且该类在类路径下，可以使用`Class.forName()`。可能抛出异常。

**适用于场景：**配置文件中，读取类的全路径

##### （2）类名.class

已知具体的类，通过类的`class`获取，该方式较为可靠。

**适用场景：**参数传递

##### （3）对象.getClass()

获取到的是类的运行类型，因为每个对象都会有引用指向其`Class`类对象，而`Class`类对象在将字节码加载到内存时记录了它的真正类型。

**适用场景：**有对象实例的情况。

##### （4）类加载器

在类加载阶段，会通过类加载器获取`Class`对象

```java
Cat cat = new Cat();
ClassLoader classLoader = cat.getClass().getClassLoader();
Class<?> cls = classLoader.loadClass(path);
```

##### （5）基本数据类型.class

自动装箱成了其包装类。

##### （6）基本数据类型的包装类.TYPE

`int.class` 和 `Integer.TYPE`得到的Class 对象是同一个

**适用场景：**基本数据类型的包装类



#### 4. 哪些类型有Class对象

* 外部类、成员内部类、静态内部类、局部内部类、匿名内部类
* 接口
* 数组
* 枚举
* 注解
* 基本数据类型
* `void`
* `Class`本身

```java
Class<String> cls1 = String.class;  // 外部类
Class<Serializable> cls2 = Serializable.class;  // 接口
Class<Integer[][]> cls3 = Integer[][].class;    // 数组
Class<Thread.State> cls4 = Thread.State.class;  // 枚举
Class<Test> cls5 = Test.class;  // 注解
Class<Integer> cls6 = int.class;    // 基本数据类型
Class<Void> cls7 = void.class;  // void
Class<Class> cls8 = Class.class;    // Class
```



#### 5. 类加载过程

##### （1）静态加载与动态加载

**静态加载：**

编译时加载，如果找不到类则报错（即使在分支语句中运行不到，依旧会报错），依赖性较强。

`new`、父类随子类加载、调用静态成员等是静态加载。

**动态加载：**

运行时加载，如果运行不到，就不会报错，依赖性降低。

反射是动态加载。



##### （2）类加载过程图

![](img/类加载.png)

![](img/类加载2.png)

* 加载和连接阶段是由`JVM`完成的。
* 注意和对象创建阶段区分开，类加载的初始化阶段只与静态成员有关。



##### （3）各阶段细节：

1. **加载：**将字节码从不同的数据源（`class`文件、`jar`包、网络）转化为二进制字节流加载到内存中（即，将数据加载到方法区），并生成一个代表该类的`Class`对象。

2. **连接：**

   1. **验证：**确保`Class`文件的字节流中的信息符合`JVM`规范，并且不会危害虚拟机自身安全。包括文件格式验证（是否以魔数`oxcafebabe`开头（字节码文件的开头是16进制的`cafebabe`八个字符））、元数据验证、字节码验证、符号引用验证等。

      可以使用`-Xverify:none`来关闭大部分验证，从而缩短虚拟机类加载的时间。

   2. **准备：**为静态变量分配内存，并初始化值。其中，`static`类型分配默认值，`static final`类型直接赋值。

      ![](img/类加载-准备阶段.png)

   3. **解析：**虚拟机将常量池的符号引用替换为直接引用。（逻辑地址转为物理地址）

3. **初始化：**执行<clinit>()方法，依次收集类中的所有静态变量的赋值操作、和静态代码块汇总的语句，进行合并，然后再初始化。虚拟机会保证该方法在多线程环境中被正确地加锁、同步（在源码的`loadClass()`中有·`synchronized`关键字）。




### （二）反射基础语法

```java
// 通过全名获取类对象
// <?>表示不确定的Java类型，也可以去掉
Class<?> cls = Class.forName("com.cuc.rg.reflection.Cat");

System.out.println(cls);    // class com.cuc.rg.reflection.Cat
System.out.println(cls.getClass()); // class java.lang.Class


System.out.println(cls.getName()）;	// 获取全类名
System.out.println(cls.getSimpleName()）;	// 获取简单类名
System.out.println(cls.getPackage());	// 获取包信息

// 创建对象
Object o = cls.newInstance();
System.out.println("o的运行类型" + o.getClass());

// 获取共有属性
Field age = cls.getField("age");
System.out.println(age.get(o));

// 设置属性
age.set(cat, 2);

// 获取所有公有属性(包括父类)
Field[] fields = cls.getFields();
for (Field field : fields) {
    System.out.println(field.getName() + "，" + field.get(cat));
}
// 获取所有属性，包括私有的
Field[] declaredFields = cls.getDeclaredFields();
                   
//以int形式返回修饰符
// 0 - 默认修饰符，1 - public，2 - private，4 - protected，8 - static，10 - final
// 组合关系会直接相加，如 public static 是 9
int num = field.getModifiers();	// 属性类型
int num2 = method.getModifiers();	// 返回值类型
                   
System.out.println(field.getType());
System.out.println(method.getReturnType());
                   
// 调用方法
Method eat = cls.getMethod("play", String.class);
method.invoke(eat, "毛线团");	// 如果有返回值，一律返回Object

// 获取所有公有方法
Method[] methods = cls.getMethods();
                   
// 获取所有方法，包括私有的
Method[] declaredMethods = cls.getDeclaredMethods();
                   
// 获得形参数组信息
Class<?>[] parameterTypes = declaredMethods.getParameterTypes();
                   
// 无参构造器
Constructor constructor1 = cls.getConstructor();
System.out.println(constructor1);

// 有参构造器
Constructor constructor2 = cls.getConstructor(String.class);
System.out.println(constructor2);

// 获取本类所有构造器，不包含父类
Constructor<?>[] declaredConstructors = cls.getDeclaredConstructor();
                   
// 构造器的参数信息
Class<?>[] types = declaredConstructor.getParameterTypes();
             
// 获取父类信息
Class<?> superClass = cls.getSuperClass();
                   
// 获取接口信息                 
Class<?>[] interfaces = cls.getInterface();	
                   
// 获取注解信息                 
Annotation[] annotations = cls.getAnnotations();	
```



#### 6. 通过`Class`创建对象的方法

```java
Class<?> cls = Class.forName(path);

// 1 直接创建，在Java9之后已经被弃用
Object cat1 = cls.newInstance();

// 2 公有构造器
Constructor<?> constructor1 = cls.getConstructor(String.class);
Object cat2 = constructor1.newInstance("猫猫");

// 3 私有构造器
Constructor<?> constructor2 = cls.getDeclaredConstructor(String.class, int.class);
constructor2.setAccessible(true);   // 允许暴力访问私有构造器，私有属性和方法同理
Object cat3 = constructor2.newInstance("猫猫猫", 3);
```





### （三）反射性能

#### 1. 对比

```java
public class Index {
    public static void main(String[] args) {
        m1();
        m2();
    }

    // 普通调用方法
    public static void m1() {
        Cat cat = new Cat();
        long start = System.currentTimeMillis();
        for (int i = 0; i < 90000000; i++) {
            cat.eat();
        }
        long end = System.currentTimeMillis();
        System.out.println("m1()耗时：" + (end - start) + "毫秒");
    }

    // 反射调用方法
    public static void m2() {
        try {
            Class cls = Class.forName("com.cuc.rg.reflection.Cat");
            Object o = cls.newInstance();
            Method eat = cls.getMethod("eat");

            long start = System.currentTimeMillis();
            for (int i = 0; i < 90000000; i++) {
                eat.invoke(o);
            }
            long end = System.currentTimeMillis();
            System.out.println("m2()耗时：" + (end - start) + "毫秒");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

/* 输出：
m1()耗时：8毫秒
m2()耗时：530毫秒
*/
```

#### 2. 优化

```java
// 在反射调用方法时，取消访问检测
// 能在一定程度上优化效率
eat.setAccessible(true);

/* 输出：
m1()耗时：9毫秒
m2()耗时：232毫秒
*/
```



### （四）例题

#### 1. 例一

使用反射创建`PrivateTest`对象，修改其私有属性`name`，并调用`getName`方法输出。

```java
public class Index {
    public static void main(String[] args) throws Exception {

        // 构造对象
        Class<PrivateTest> cls = PrivateTest.class;
        Constructor<?> constructor = cls.getConstructor();
        PrivateTest test = (PrivateTest) constructor.newInstance();

        // 获得私有属性，并修改
        Field name = cls.getDeclaredField("name");
        name.setAccessible(true);
        name.set(test, "李四");

        // 获得方法并输出
        Method method = cls.getMethod("getName");
        System.out.println(method.invoke(test));
    }
}

class PrivateTest {
    public PrivateTest() {
    }

    private String name = "张三";

    public String getName() {
        return name;
    }

```



#### 2. 例二

使用反射打印`File`的所有构造器，使用`newInstance()`获取`File`对象，并在`e`盘创建文件。

```java
public class Index {
    public static void main(String[] args) throws Exception {
        // 打印构造器
        Class<?> cls = Class.forName("java.io.File");
        Constructor<?>[] declaredConstructors = cls.getDeclaredConstructors();
        for (Constructor constructor : declaredConstructors) {
            System.out.println(constructor);
        }

        // 创建对象
        Constructor<?> constructor = cls.getDeclaredConstructor(String.class);
        Object file = constructor.newInstance("e:/背景/test.txt");
        
        // 调用方法创建文件
        Method createNewFile = cls.getDeclaredMethod("createNewFile");
        createNewFile.invoke(file);

    }
}
```

