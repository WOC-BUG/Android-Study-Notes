## 一、抽象

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



## 二、接口

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

