#  java基础

## java三大版本

- javaSE:标准版
- javaME:嵌入式
- javeEE:E企业级开发（web端，服务器开发）

## JDK、JRE、JVM

JDK: java develpment kit （java开发工具包）

JRE: java runtime envionment (Java运行环境)

JVM: java virtual machine (Java虚拟机)

**JDK>JRE>JVM**

## Hellow word

### 代码

```JAVA
public class Hellow{
    public static void main(String[] args) {
        System.out.print("Hellow World!");
    }
} 
```

### 运行

```shell
➜  learn javac Hellow.java # 编译 生成Hellow.java
➜  learn java Hellow # 运行 运行的是Hellow.java
Hellow World!%   # 输出
```

### 注意点

1. 每个单词大小写不能出问题，java大小写敏感
2. 类名和文件名要保持一致，并且首字母大写

## 数据类型

> 强类型语言

要求变量的使用要求严格符合规定，所有变量都必须先定义才能使用

> 数据类型分为两类

基本类型（primitive type） 

> 整数类型

​	byte占1个字节，范围：-128至127 

​	short占2个字节，范围：32768至32767

​	int占4个字节

​	long占8个字节

> 浮点类型

​	float占4个字节

​	dlouble占8个字节

> 字符类型

​	 char占两个字节

> boolean类型

​	占1位 1个bit

```java
 // 八大基本数据类型

// 整数
int num1 = 10; // 最常用的
byte num2 = 20;
short num3 = 30;
long num4 = 40L; // long 类型要在数字后面加个L

// 浮点数
float num5 = 50.1F; // float类型要在数字后面加F
double num6 = 3.141592654;

// 字符
char name = '嘉';

// 布尔类型
boolean flag = true;
```



引用类型  (reference type)

### 数组

- 数组是相同类型数据的有序集合
- 数组长度是固定的，一旦被创建，大小不能被改变
-    数组是引用类型，数组也可以看成是对象，数组中的每个元素相当于该对象的成员变量



> 声明数组

```java
  int[] numbers;  // 声明数组
  numbers = new int[10]; // 数组赋值
  numbers.length // 10获取数组长度
  // 数组初始化
  // 静态初始化
  int [] numbers = {1,2,3,4,5};
	// 动态初始化,包含默认初始化  
   int[] numbers = new int[10]; // 数组赋值 
	 numbers[0] = 1;
  
```

> 多维数组

  ```java
  int [][] numbers = new int[2][5];
  ```

> 数组扩容

先新建个大容量的数据，然后将小容量数组中的数据拷贝到大的数组当中，数组扩容效率低，设计到内存拷贝，尽可能少进行数组拷贝。可以在创建数组的时候，预估数组的长度。

> 数组拷贝

```java
   public static void main(String[] args) {
        int[] src = {1, 3, 5, 2, 4, 73, 0, 10};
        int[] dist = new int[20];
        System.arraycopy(src, 0, dist, 0, src.length);
        for (int value:dist){
            System.out.println(value);
        }
    }
```

### 集合

> 什么是集合

集合是个容器，可以用来容纳不同类型的数据。集合不能直接存储基本类型，也不能直接存储java对象。集合中存储的是java对象的内存地址，存储的是引用类型。不同类型的集合代表了不同的数据结构。

  

## 类型转换

> 强制转化

（Type）var

> 类型转换必须是低类型转高类型

```java
// 低->高
byte,short,char->int->long->float->double
```

> 运算中，不同类型的数据先要转换为同一类型，然后再预算

> 注意点

1. 不能对布尔值进行转换
2. 不能把类型转换为不相关的类型
3. 高容量转低容量需要强制转换
4.  转换时可能存在内存溢出或者损失精度的问题

## 流程控制

> Scanner

```java
public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        if (scanner.hasNext()){
            System.out.println(scanner.next());
        }
        scanner.close();
    }
```

## 方法讲解

> 方法重载

重载就是在一个类中，有方法名相同但是参数不同的方法

## 内存分析

> 堆

- 存放new的对象和数组
- 可以被所有线程共享
- 使用完毕就编程垃圾，但是没有立即回收。会有垃圾回收期空闲的时候回收

> 栈

- 存放的是局部变量的值，包括1.基本数据类型的值，2。保存累的市里，即堆中对象的引用（指针）

- 使用完就释放

  

> ​     方法区

- 可以被所有线程共享
- 包含了所有的class和static变量

> 知识点

**1.**一个Java文件，只要有main入口方法，我们就认为这是一个Java程序，可以单独编译运行。

**2.**无论是普通类型的变量还是引用类型的变量(俗称实例)，都可以作为局部变量，他们都可以出现在栈中。只不过普通类型的变量在栈中直接保存它所对应的值，而引用类型的变量保存的是一个指向堆区的指针，通过这个指针，就可以找到这个实例在堆区对应的对象。因此，普通类型变量只在栈区占用一块内存，而引用类型变量要在栈区和堆区各占一块内存。

# OOP

## 构造方法

-  必须和类名相同
- 不能有返回类型，也不能写void
- 一旦定了有参构造方法，无参构造方法就需要显示定义

## This

**对象创建的过程和this的本质**

　　构造方法是创建Java对象的重要途径，通过new关键字调用构造器时，构造器也确实返回该类的对象，但这个对象并不是完全由构造器负责创建。创建一个对象分为如下四步：

　　1. 分配对象空间，并将对象成员变量初始化为0或空

　　2. 执行属性值的显示初始化

　　3. 执行构造方法

　　4. 返回对象的地址给相关的变量

　　this的本质就是“创建好的对象的地址”! 由于在构造方法调用前，对象已经创建。因此，在构造方法中也可以使用this代表“当前对象” 。

　　this最常的用法：

　　1.  在程序中产生二义性之处，应使用this来指明当前对象;普通方法中，this总是指向调用该方法的对象。构造方法中，this总是指向正要初始化的对象。

　　2. 使用this关键字调用重载的构造方法，避免相同的初始化代码。但只能在构造方法中用，并且必须位于构造方法的第一句。

　　3. this不能用于static方法中

## 特性

> 封装



> 继承

Java只有单继承

###  super()

- super调用父类的构造方法，必须在子类构造方法第一行
- super必须只能出现在子类的方法中
- super和this不能同时出现

### VS This() 

 代表队对象不同

​		this():当前对象,没有继承就可以使用，调用当前类的构造方法

​		super()：父类对象，只有在继承条件下才能使用，调用父类构造方法

  

> 多态

方法重写，需要由继承关系，子类重写父类的方法。

- 方法名必须相同
- 参数列表必须相同
- 修饰符，范围可以扩大，不能缩小 
- 抛出的异常，范围可以被缩小，但不能扩大

- 静态方法，调用方法与类型有关
- 动态方法，调用与对象有关

## Instanceof

判断两个类是否存在父子关系

## 类型转换

 父类可以转子类，需要强转。

子类可以自动转父类

## Static



## Object

这个类是java中所有类的基类，就算没有直接继承，也会间接继承 

> 常用方法

```java 
clone()  // 负责克隆对象

equals() 	// 判断两个对象是否相等  "=="比较两个对象会比较内存地址，改方法默认使用”==“,该方法不够用，我们应该重写该方法比较两个对象的内容。

finalize() // 垃圾回收期负责调用的方法

hashCode() // 获取对象哈希值的方法 可以看做一个对象的内存地址

toString() // 将对象转成字符串 默认是现实 类名@对象的内存地址，作用是通过调用本方法可以将一个java对象转换为字符串表示形式。输出引用的时候，自动调用该方法。
```





# 常用包

> StringBuffer 拼接字符串，方法都有synchronized关键字修改，线程安全

```java
   StringBuffer stringBuffer = new StringBuffer();

        stringBuffer.append("a").append("b");

        System.out.println(stringBuffer);
```

> StringBuilder 线程不安全



# 多线程

> 开启多线程的两种方式

1.编写一个类，直接继承java.lang.Thread,重写run方法

```java
// 定义线程类
public class MyThread extends Thread{
    public void run(){
        System.print.out("do some thing");
    }
}
// 创建线程对象
MyThread t = new MyThread();
// 启动线程
t.start();
```

2.编写一个类，实现java.lang.Runnable接口，实现run方法

```java
// 定义一个可运行的类
public class MyRunnable implements Runnable{
    public void run(){
           System.print.out("do some thing");
    }
}
// 创建线程对象
Thread t = new Thread(new MyRunnable);
// 启动线程
t.start();
```

# SpringBoot

## 原理

> pom.xml

> 启动器--SpringBoot的启动场景

```xml
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
</dependency>
自动导入web环境所有依赖
```

> 注解

```
@SpringBootConfiguration：SpringBoot的配置类
	@Configuration：Spring配置类
	@Component：说明这是一个组件
	
@EnableAutoConfiguration：自动配置
	@AutoConfigurationPackage：自动配置包
	@Import({Registrar.class})：导入选择器
	@Import({AutoConfigurationImportSelector.class})






```