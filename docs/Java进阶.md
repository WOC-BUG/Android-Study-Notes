## 一、工具

`javac` 编译工具

`javap` 反编译工具(从`class` 文件得到`java`文件)：

![](img/javap_enum.png)



## 二、抽象

当父类方法不确定具体实现内容时，可以定义其为抽象方法。

```java
public abstract class Animal{
    public abstract void eat();
}
```

**注意：**

1. 包含抽象方法的类，必须声明为抽象类；
2. 抽象方法没有方法体；
3. 抽象类可以没有抽象方法；
4. `abstract`只能修饰类和方法；
5. 继承了抽象类的子类，必须实现其父类的所有抽象方法，除非它自己也声明为抽象类；
6. **抽象方法不能使用`private`、`final`、`static`，因为加了这三个关键字的方法无法被重写。**



**应用：模板方法模式**



## 三、接口

### （一）基本概念

实现接口是对`java`单继承机制的补充，继承是满足`is-a`的关系，实现接口是满足`like-a`的关系。

* `Jdk7`之前，接口中的所有方法都没有方法体；

* `Jdk8`之后，接口中可以有静态方法、默认方法（`default`修饰）；

* 接口中的抽象方法可以不加`abstract`关键字；

* 一个类可以实现多个接口；

* 接口不能继承类，只能**继承**其他接口，注意是`extends`关键字；

  ```java
  interface A{}
  interface B{}
  interface C extends A,B {}
  ```

* 接口中的所有方法都是`public`的；

* 接口本身只能是`public`或默认的；

* 接口中的属性实际上隐藏了`static final`修饰符，因此必须初始化；

  ```java
  interface A{
  	int n = 10; // 等价于public static final n = 10;
  }
  ```

* 实现了接口的抽象类，可以不实现接口中的方法；

* **接口在一定程度上实现代码的接口**。



### （二）接口的多态特性

#### 1. 多态参数

```java
interface Usb {
    public void start();
    public void end();
}

class Camera implements Usb {
    @Override
    public void start(){
        System.out.println("相机开始工作");
    }
    
    @Override
    public void stop(){
        System.out.println("相机停止工作");
    }
}

class Phone implements Usb {
    @Override
    public void start(){
        System.out.println("手机开始工作");
    }
    
    @Override
    public void stop(){
        System.out.println("手机停止工作");
    }
}

class Computer {
    public void work(Usb usb) {	// 传入接口
        usb.start();
        usb.stop();
    }
}
```



#### 2. 多态数组

```java
interface Usb{}
class Camera implements Usb{}
class Phone implements Usb{
    public void call(){
        System.out.println("打电话...")
    }
}

public class Index{
    public static void main(String[] args) {
        Usb[] usbs = new Usb[2];
        usbs[0] = new Phone();
        usbs[1] = new Camera();
        
        for(Usb i: usbs){
            if(i instanceof Phone){
                ((Phone)i).call(); // 向下转型
            }
        }
    }
}
```



#### 3. 接口的多态传递现象

```java
interface IA { 
    void hi();
}
interface IB extends IA { }
class C implements IB {
	@Override
    public void hi() { }
}

public class Index{
    public static void main(String[] args) {
        IA a = new C();
        a.hi();
    }
}
```



### （三）继承与接口

**继承与接口同时使用时，相同的属性必须明确其含义**

```java
interface A {
    int x = 10;
}

class B{
    int x = 20;
}

class C extends B implements A{
    public void printX(){
        // System.out.println(x);  // x is ambiguous
        System.out.println(A.x);
        System.out.println(super.x);
    }
}
```





## 四、内部类

**类的五大成员：属性、方法、构造器、初始化块、内部类**

```java
class OuterOther{	// 外部其他类
    
}
class Outer{	// 外部类
    int n = 10; // 属性
    Outer() {	// 构造器
        
    }
    {
        // 初始化块
    }
    int getN(){	// 方法
        return n;
    }
    class Inner{	// 内部类
    }
}
```



### （一）局部内部类

``` java
class OuterOther{
    // 5. 无法访问Inner
}
class Outer{
    private int n = 10;
    private void m1() {}
    
    public void m2() {
        final class Inner{	// 3. 作用域仅在m2()中
            int n = 20;	// 6. 变量n重名
            public void f1(){
                // 1. 直接访问外部类的所有成员
                m1();                
                
                System.out.println(n);	// 6. 就近原则，输出20
                System.out.println(this.n);	// 内部类的n
                System.out.println(Outer.this.n);	// 6. 外部类的n，this为调用m2()的对象
            }
        }
    }
    
    {
        class Inner{	// 作用域仅在初始化块中
            public void f1(){
        		m1();
            }
        }
    }
    
    Inner inner = new Inner(); // 4. 外部类通过创建实例访问
}
```



1. 能直接访问外部类的所有成员，包括私有属性；
2. 不能添加访问修饰符，因为它相当于一个局部变量，`final`、`abstract`等可以添加；
3. 作用域仅在方法/代码块中；
4. 外部类通过创建内部类的实例调用方法；
5. 外部其它类，不能访问局部内部类（因为它相当于一个局部变量）；
6. 外部类和局部内部类的成员重名时，遵循就近原则，若想访问外部类的成员，可以使用`外部类名.this.成员`去访问；



### （二）匿名内部类

``` java
class Outer {
    private int n = 10;

    public void method() {
        // 编译类型：IA
        // 运行类型：匿名内部类（底层会分配一个类名Outer$1）
        IA tiger = new IA() {
            @Override
            public void cry() {
                System.out.println("虎啸...");
            }
        };
        tiger.cry();
        
        // 编译类型：Father
        // 运行类型：匿名内部类（底层会分配一个类名Outer$2）
        Father father = new Father("Jack"){
            @Override
            public void test(){
                System.out.println("匿名内部类重写test()");
            }
        }
        father.test();
    }
}

interface IA {
    public void cry();
}

class Father{
    Father(String name){ }
    public void test(){
        
    }
}
```



1. 能直接访问外部类的所有成员，包括私有的；

2. 不能添加访问修饰符，因为它相当于一个局部变量，`final`可以添加；

3. 作用域仅在方法/代码块中；

4. 本质是类，同时还是一个对象；

   ```java
   // 接上例，也可以直接调用方法
   new Father("Jack"){
       @Override
       public void test(){
           System.out.println("匿名内部类重写test()");
       }
   }.test();
   ```

5. `jdk`底层会给匿名内部类分配名字：外部类名$编号；

6. 匿名内部类只能使用一次（注意，不是说匿名内部类的对象）；

7. 外部其它类，不能访问匿名内部类（因为它相当于一个局部变量）；

8. 外部类和匿名内部类的成员重名时，遵循就近原则，若想访问外部类的成员，可以使用`外部类名.this.成员`去访问；

9. 不能重写构造器。



### （三）成员内部类

```java
class Outer{
    private int n = 10;

    private class Inner{
      void fun(){
          System.out.println(n);
      }
    };

    public void printInnerFun(){
        Inner inner = new Inner();
        inner.fun();
    }
}
```



1. 能直接访问外部类的所有成员，包括私有的；

2. 本质是类，同时也是属性；因此，**可以加访问修饰符，作用域为整个类体**；

3. 外部类访问成员内部类，可以通过创建对象；

4. 外部其他类访问成员内部类：

   ```java
   // 1. 通过外部类创建对象
   Outer outer = new Outer();
   Outer.Inner inner = outer.new Inner();
   
   // 2. 在外部类中编写方法，返回内部类对象
   class Outer{
       // ...
       public Inner getInner(){
       	return new Inner();
   	}
       // ...
   }
   ```

5. 外部类和成员内部类的成员重名时，遵循就近原则，若想访问外部类的成员，可以使用`外部类名.this.成员`去访问。



### （四）静态内部类



1. 可以访问外部类的所有静态成员，包括私有的；

2. 本质是类，同时也是静态属性；因此，**可以加访问修饰符，作用域为整个类体**；

3. 外部类访问静态内部类，可以通过创建对象；

4. 外部其他类访问静态内部类：

   ```java
   // 1. 通过类名直接访问
   Outer.Inner inner = new Outer.Inner();
   
   //  2. 在外部类中编写方法，返回内部类对象
   class Outer{
       // ...
       public Inner getInner(){
       	return new Inner();
   	}
       public static Inner getInner_(){
       	return new Inner();
   	}
       // ...
   }
   ```

5. 外部类和成员内部类的成员重名时，遵循就近原则，若想访问外部类的成员，可以使用`外部类名.成员`去访问。



## 五、枚举

1. 一组有限常量的集合
2. 不能修改



### （一）自定义枚举

```java
class Season{
    private String name;
    private String detail;

    // 固定几个public static对象
    // 加入final修饰符优化
    public static final Season SPRING = new Season("春天","花");
    public static final Season SUMMER = new Season("夏天","西瓜");
    public static final Season AUTUMN = new Season("秋天","枫叶");
    public static final Season WINTER = new Season("冬天","雪");

    // 私有化构造器，避免多余的对象产生
    private Season(String name,String detail){
        this.name = name;
        this.detail = detail;
    }

    // 去除set方法，避免修改属性
    public String getName() {
        return name;
    }

    public String getDetail() {
        return detail;
    }
}
```



### （二）enum关键字实现

```java
enum Season{

    // 必须写在最前面，且必须用逗号隔开
    SPRING("春天","温暖"),
    SUMMER("夏天","西瓜"),
    AUTUMN("秋天","凉爽"),
    WINTER("冬天","雪");

    private String name;
    private String detail;

    // 私有化构造器，避免多余的对象产生
    private Season(String name,String detail){
        this.name = name;
        this.detail = detail;
    }

    // 去除set方法，避免修改属性
    public String getName() {
        return name;
    }

    public String getDetail() {
        return detail;
    }
}
```

如果是无参构造可以简写为：

```java
// 无参构造的四个Season对象
enum Season{
    SPRING,SUMMER,AUTUMN,WINTER;
}
```



**例题：**

```java
enum Gender{
    BOY,GIRL;
}

// enum 类的 toString() 方法返回的是枚举项的 name
System.out.println(Gender.BOY);	// 输出BOY
```



### （三）常用方法

![](img/enum_method.png)



```java
Season autumn = Season.AUTUMN;
Season summer = Season.SUMMER;

System.out.println(autumn.ordinal());	// 输出2，下标从0开始
System.out.println(autumn.name());	// 输出AUTUMN
Season[] values = autumn.values();	//返回所有枚举项
Season autumn2 = Season.valueOf("AUTUMN");	// 按名称返回对应枚举项，autumn2和autumn是同一个对象
System.out.println(summer.compareTo(autumn));	// 输出-1，是两个的序号相减的结果
```



**注意：`enum`有隐式继承`Enum`类，因此不能继承其他类，但是可以实现接口。**



## 六、注解（Annotation）

又称为元数据(`Metadata`)

### （一）@Override

用于重写父类方法

```java
@Target(ElementType.METHOD)	// 限制注解使用在哪些元素上
@Retention(RetentionPolicy.SOURCE)	// 限制注解只保留在源文件，其他选项有:CLASS、RUNTIME
public @interface Override {	// @interface表示是注解，不是接口
}
```

`Target`和`Retention`是修饰注解的注解，称为元注解。



### （二）@Deprecated

* 表示某个元素（类、方法等）已过时，不推荐使用
* 可以用于版本升级时的过渡

```java
@Documented	// 表示该注解应当被javadoc工具记录
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, MODULE, PARAMETER, TYPE})
public @interface Deprecated {
    /**
     * Returns the version in which the annotated element became deprecated.
     * The version string is in the same format and namespace as the value of
     * the {@code @since} javadoc tag. The default value is the empty
     * string.
     *
     * @return the version string
     * @since 9
     */
    String since() default "";

    /**
     * Indicates whether the annotated element is subject to removal in a
     * future version. The default value is {@code false}.
     *
     * @return whether the element is subject to removal
     * @since 9
     */
    boolean forRemoval() default false;
}
```



### （三）@SuppressWarning

抑制（不显示）编译器警告

常用警告类型：

1. `unchecked`未检查的警告
2. `rawtypes`没有指定泛型的警告
3. `unused`没有使用某个变量的警告
4. `all`所有警告

点击左侧黄色提醒，即可快速添加`SupressWarning`

```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE, MODULE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    /**
     * The set of warnings that are to be suppressed by the compiler in the
     * annotated element.  Duplicate names are permitted.  The second and
     * successive occurrences of a name are ignored.  The presence of
     * unrecognized warning names is <i>not</i> an error: Compilers must
     * ignore any warning names they do not recognize.  They are, however,
     * free to emit a warning if an annotation contains an unrecognized
     * warning name.
     *
     * <p> The string {@code "unchecked"} is used to suppress
     * unchecked warnings. Compiler vendors should document the
     * additional warning names they support in conjunction with this
     * annotation type. They are encouraged to cooperate to ensure
     * that the same names work across multiple compilers.
     * @return the set of warnings to be suppressed
     */
    String[] value();
}
```



### （四）元注解

#### 1. Retention

用于指定注解可以保留的时长

参数：

1)` RetentionPolicy.SOURCE`：注解只保留在源文件中

2) `RetentionPolicy.CLASS`：注解保留到编译后，记录在`class`文件中

3) `RetentionPolicy.RUNTIME`：注解保留到运行时，程序可以通过反射获取该注解

#### 2. Target

指定能够修饰的程序元素：类、方法、局部变量、属性、构造器、参数、包、模块等

#### 3. Documented

指定当前注解能被`javadoc`提取成文档

#### 4. Inherited

被修饰的注解将具有继承性，若某个类的注解被`Inherited`修饰，则它的子类自动具有该注解



## <font color='red'>七、异常</font>

### （一）try-catch

```java
try {
	// 可能出错的代码块
} catch(NullPointerException e) { // 异常发生时会直接进入catch块，不执行异常后面的代码；若没有异常则不会进入
    System.out.println(e.getMessage());
} catch(ArithmeticException e) {
    System.out.println(e.getMessage());
}catch(Exception e) {	// 子类异常要写在父类异常前面
	e.printStackTrace();
    System.out.println(e.getMessage());
} finally {
    // 不管有没有发生异常，必定执行的代码
}
```



`catch`也可以不写：

```java
try{
    int n1 = 10;
    int n2 = 0;
    System.out.println(n1 / n2);
} finally {
    System.out.println("finally");
}
System.out.println("程序继续执行");

/*
输出：
finally
*/
```

虽然`finally`执行了，但是没有捕获异常，不会输出后面的内容。



### （二）异常分类

$$ 异常\left\{ \begin{aligned} Error \\ Exception \end{aligned} \right. $$

#### 1. Error

`Java`虚拟机无法解决的严重错误，会导致程序崩溃：`JVM`系统内部错误、资源耗尽等。

$$ Error\left\{ \begin{aligned} StackOverFlow \\ OutOfMemory \end{aligned} \right. $$

#### 2. Exception

其他编程错误，或偶然的外在因素导致的一般性问题：空指针访问、读取不存在的文件、网络中断等。

$$ Exception \left\{ \begin{aligned} 1) 运行时异常（默认throws抛给父类，直到JVM） \\ 2) 编译时异常（必须用try-catch或throws处理） \end{aligned} \right. $$



$$ 1) 运行时异常\left\{ \begin{aligned} NullPointerException （空指针异常） \\ ClassCastException （类型转换异常） \\ NumberFormatException （数字格式异常） \\ ArrayIndexOutOfBoundsException （数组越界异常） \\ ArithmeticException（算术运算异常） \end{aligned} \right. $$



$$ 2) 编译时异常 \left\{ \begin{aligned} FileNotFoundException （找不到文件异常） \\ ClassNotFoundException （找不到类异常） \\ IOException （文件异常） \\ SQLException （数据库异常） \\ EOFException （操作文件到文件末尾） \\ IllegalArgumentException （参数异常） \end{aligned} \right. $$



### （三）异常体系图

![](img/Throwable.png)



### （四）throws

* 将错误抛出给调用自己的方法，直到抛给`JVM`（若程序员没有显示地处理异常，默认使用`throws`）
* `throws`可以抛出一系列异常
* `throws`抛出的异常可以是发生异常的父类

![](img/throws.png)

1. 子类重写父类方法时，抛出的异常必须是**父类异常**或**父类异常的子类**；
2. `try-catch`和`throws`二选一，有了`try-catch`可以不必写`throws`;
3. 注意，子类抛出的编译时异常，父类一定要处理或继续抛出，而运行异常不用，因为有默认处理机制。

区别|定义|位置|后面跟的东西
---|---|---|---
throws|异常处理的方式|方法签名后|异常类型
throw|生成异常对象|方法体中|异常对象




### （五）自定义异常

```java
public class Index {
    public static void main(String[] args) {
        int age = 150;
        if(!(age>=0 && age <= 120)){
            throw new AgeException("年龄需要在0到120岁之间！");
        }
        System.out.println("年龄正确！");
    }
}

class AgeException extends RuntimeException {
    AgeException(String message){
        super(message);
    }
}
```

抛出效果：

![](img/age_exception.png)

注意，自定义异常一般都继承`RuntimeException`，如果继承编译时异常还要手动抛出，比较麻烦。



### （六）异常例题

例一：

```java
try {
    String[] arr = new String[3];	// 1
    if (arr[1].equals("hello")) {	// 2
        System.out.println(arr[1]);
    } else {
        arr[3] = "world";
    }
    return 1;
} catch (ArrayIndexOutOfBoundsException e){
    return 2;
} catch (NullPointerException e){	// 3
    return 3;	// 4
} finally {	// 5
    return 4;	// 6
}

// 返回4
```



例二：

```java
int i = 1;
try {
    i++;
    String[] arr = new String[3];
    if (arr[1].equals("hello")) {
        System.out.println(arr[1]);
    } else {
        arr[3] = "world";
    }
    return i;
} catch (ArrayIndexOutOfBoundsException e) {
    return i;
} catch (NullPointerException e) {
    return ++i;     // 临时保存i的值：int tmp = i;
} finally {
    ++i;
    System.out.println("i = " + i); // i = 4
}

// 输出i = 4，返回3
```



例三：

```java
public static void main(String[] args) {
    try {
        func();
        System.out.println("A");
    }catch (Exception e){
        System.out.println("C");
    }
    System.out.println("D");
}

public static void func(){
    try {
        throw new RuntimeException();
    } finally {
        System.out.println("B");
    }
}

/* 输出：
B
C
D
*/
```





习题：

让用户输入一个整数，如果输入的不是整数，就一直重新输入。要求用异常实现。

```java
public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    int num = 0;
    while (true) {
        try {
            String str = scanner.next();
            num = Integer.parseInt(str);
            break;
        } catch (NumberFormatException e) {
            System.out.println("你输入的不是整数");
        }
    }
    System.out.println("你输入的是：" + num);
}
```

如果输入正确，则直接`break`，否则会被`catch`，继续进入循环。









## 八、常用类

### （一）包装类

| 基本数据类型 | 包装类    |
| ------------ | --------- |
| boolean      | Boolean   |
| char         | Character |
| byte         | Byte      |
| short        | Short     |
| int          | Integer   |
| long         | Long      |
| float        | Float     |
| double       | Double    |



**Boolean和Character类结构图：**

![](img/Boolean.png)

**其余包装类结构图如下：**

![](img/wrapper.png)



#### 1. 装箱与拆箱

```java
int n = 100;

// 手动装箱
Integer integer = new Integer(n);
Integer integer2 = Integer.valueOf(n);

// 手动拆箱
int n2 = integer.intValue();


// jdk5以后
// 自动装箱
Integer integer3 = n;	// 底层使用了Integer.valueOf(n);

// 自动拆箱
int n3 = integer;	// 底层使用了integer.intValue()
```



#### 2. 包装类与String相互转换

**包装类转String：**

1. `String str = n + "";`
2. `String str = n.toSting();`
3. `String str = String.valueOf(n);`

**String转包装类：**

1. `Integer n = Integer.parseInt(str);`

2. `Integer n = new Integer(str);`

   

#### 3. 例题

**例1：**

```java
// 1
Object obj = true?new Integer(1):new Double(2.0);
System.out.println(obj);
// 输出 1.0
// 因为三元运算符是一个整体，要把类型转换为范围大的

// 2
Object obj;
if(true){
    obj = new Integer(1);
} else {
    obj = new Double(2.0);
}
System.out.println(obj);
// 输出 1
```



**例2：**

```java
// 1
Integer n1 = new Integer(1);
Integer n2 = new Integer(1);
System.out.println(n1 == n2);	// false

// 2
Integer n3 = 1;
Integer n4 = 1;
System.out.println(n3 == n4);	// true

// 3
Integer n5 = 128;
Integer n6 = 128;
System.out.println(n5 == n6);	// false
```

查看源码可知，自动装箱调用了`valueOf()`方法：

![](img/valueof.png)

$[-128,127]$范围内的数，返回的是`cache`数组中的值，因此地址一样；范围外的数是直接`new`的`Integer`对象。



**例3：**

``` java
Integer n1 = 127;
int n2 = 127;
System.out.println(n1 == n2);	// true

int n3 = 127;
Integer n4 = 127;
System.out.println(n3 == n4);	// true
```

主要有基本数据类型，判断的就是值是否相等。



### <font color='red'>（二）String类</font>

#### 1. 特点

![](img/string.png)

* 实现了`Serializabele`接口可以串行化，可用于网络传输；
* 实现了`Comparable`接口可以相互比较；
* `String name = "Kun";`中的`"Kun"`是常量，保存在常量池中，再由`name`指向；
* `String`类中有一个属性用于存放字符串：`private final char value[]`，`value[]`是一个`final`类型，说明它的地址不可以修改。



#### 2. String的内存布局

* **String str = "Hello"**

  `str`直接指向常量池中的地址。

![](img/JVM内存_String.png)



* **String str = new String("Hello")**

  堆中开辟空间，`str`指向堆，对象内的`value`属性指向常量池中的地址。

![](img/JVM内存_String2.png)



#### 3. 例题

**例1：**

```java
String a = "abc";
String b = "abc";
System.out.println(a.equals(b));	// true
System.out.println(a == b);	// true
```



**例2：**

```java
String a = "hello";
String b = new String("hello");

System.out.println(a.equals(b));	// true
System.out.println(a == b);	// false
System.out.println(a == b.intern());	// true
System.out.println(b == b.intern());	// false
```

`intern()`方法：返回常量池中该字符串的地址；若没有，就将其添加到池中，再返回地址。以上面的图为例，`str`的地址是`ox33`，该方法返回的是`0x99`。



**例3：**

```java
String a = "hello";
String b = "hi";
// 产生了2个对象
// （这里把引用指向的空间算成一个对象）

String c = "Hello" + "World";
// 产生了1个对象
// 编译器会优化为"HelloWord"

String d = "Hello";
String e = "World";
String f = d + e;
// 产生了4个对象
// 1. d 和 e 各占一个对象
// 2. 创建了一个StringBuilder对象builder
// 3. 该对象通过两次执行append()方法，连接字符串HelloWorld
// 4. f = builder.toString();创建了一个String对象
```



**例4：**

```java
public class Index {
    String str = new String("hello");
    char[] ch = new char[]{'j', 'a', 'v', 'a'};		// 只有这种写法才能直接输出char[]

    public static void main(String[] args) {
        Index index = new Index();
        index.change(index.str, index.ch);

        System.out.print(index.str + " ");
        System.out.println(index.ch);
    }

    public void change(String str, char[] ch) {
        str = "java";
        ch[0] = 'h';
    }
}
// 输出：hello hava
```



**例4过程分析如下：**

1. 没有执行`change()`方法前，`main`方法中的`index`引用指向堆中的对象。
2. 对象中有`str`和`ch`两个引用，`str`指向堆中的`String`对象，`ch`指向堆中的`char[]`对象。
3. `String`对象指向常量池中的`hello`常量，而`char[]`对象中直接保存了字符值。

![](img/String例题1.png)

4. 执行了`change`方法后，栈中新增栈帧，传入的`str`和`ch`引用被拷贝到方法中成为局部变量。
5. 局部变量`str`直接指向常量池中的`java`常量，局部变量`ch`修改了堆中的字符。

![](img/String例题2.png)

6. 回到`main`方法后，堆中的`str`依旧指向`hello`，`ch`所指向的字符数组内容已经被改变。

（注意，`final char[]`表示引用保存的地址不可修改，而不是内容）

