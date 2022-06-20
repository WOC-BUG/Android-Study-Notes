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



### （一） 局部内部类

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



### （二） 匿名内部类

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



### （三） 成员内部类

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



### （四） 静态内部类



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



## 六、注解

