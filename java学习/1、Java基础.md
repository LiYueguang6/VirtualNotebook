# Java基础

Java基础还是要多研究一下的，面试跌倒在这种地方就不太好了。

## 一、数据类型

### 基本类型

byte/8
char/16
short/16
int/32
float/32
long/64
double/64
boolean/1bit存储，但是JVM会转成int，boolean数组是通过byte数组实现  

### 包装类型

基本类型都有包装类型
```java
Integer x = 1; // 装箱
int y = x; // 拆箱
```
### 缓存池
new Integer(123);和Integer.valueOf(123)的区别在于：
* new Integer(123);每次会新建一个对象。
* Integer.valueOf(123)会使用缓冲池中的对象。
**装箱**过程中自动调用valueOf。

引用同一个对象的包装类，==时相同，equals

缓冲池大小： 
boolean true false
byte all
short -128~127
int -128~127
char \u0000 to \u007F（也是127）

## 二、String
String是final类型的类，无法被继承。Java8中使用char数组存储String。Java9使用byte数组，并且使用coder标识编码类型。类型中的value数组也是final类型的，初始化后不能再引用其他数组，并且String类内部也无法改变value，所以可以保证String是不可变的。（注意：final修饰的数组是可变的，只是对象不能变，对象内部可变）

### 优点
1. String的hash值也不会变，可以用作HashMap的key
2. 如果一个String对象已经被创建，就会从String pool中取得引用。只有不变的String，才可以用在String Pool中。
3. 安全且线程安全。

### String、StringBuffer、StringBuilder
1. 可变性
String不可变
StringBuffer、StringBuilder可变
2. 线程安全
String安全
StringBuilder不安全，单线程作为StringBuffer的单线程替代
StringBuffer使用了synchronized，线程安全

### String Pool

**独立**在堆内存空间外

new String("abc")不会把创建新对象放在String Pool中，而是放在堆中；String对象的intern方法会在StringPool中寻找是否存在对象（使用equals），不存在则在String Pool中新建，并返回地址。```String s2 = s1.intern();```。

```java
String s = "abc";
```

这种方式创建的String，先去String Pool中匹配相应的对象，不存在则在StringPool新建对象。

## 三、==和equals

==在基本类型中比较值，在对象中比较地址，包装类中比较引用值地址（根据缓冲池和创建方法有所不同）

equals比较对象的引用，String和包装类重写了equals所以比较值。



int和Integer这种包装类使用==比较时，包装类会自动拆箱

## 四、面向对象三大特性

### 1.封装

### 2.继承

### 3.多态

#### 分派

网上常说Java语言是一门静态多分派、 动态单分派的语言，具体来说是因为

- Java中方法的执行要经历两个过程的选择，一个是**编译期的静态分派**，一个是**运行期的动态分派**。这种分派场景出现的根本原因是多态。 
- 而一个方法需要确定的宗量有两个，一个是方法的接受者（调用者），一个是方法的参数。
- 进行编译期的编译时，需要确定的参数有两个，一个是接受者，一个是参数，所以此时是多分派的。
- 编译之后，方法的参数就已经确定了。等进入到运行期，此时宗量就只剩下方法的接受者了，所以此时是单分派的。
- 确定方法参数的时候，能够影响到方法选择的，就是方法的重载。
- 确定方法接受者的时候，能够影响到方法选择的，就是方法的重写。

## 五、JAVA内存模型

JAVA内存模型（JMM）是线程间的通信控制机制。定义了主存和线程的关系。

共享变量存储在**主存**中，每一个线程拥有一个**本地内存（工作内存）**，存储了该线程读写变量的副本。

**内存模型八大操作**：

![img](1、Java基础.assets/8b7ebbad-9604-4375-84e3-f412099d170c.png)

- read：把一个变量的值从主内存传输到工作内存中
- load：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中
- use：把工作内存中一个变量的值传递给执行引擎
- assign：把一个从执行引擎接收到的值赋给工作内存的变量
- store：把工作内存的一个变量的值传送到主内存中
- write：在 store 之后执行，把 store 得到的值放入主内存的变量中
- lock：作用于主内存的变量
- unlock

**内存模型三大特性**：

1. 原子性

	八大操作具有原子性，但是不能保证对一个变量进行读写操作具有原子性（中间多个操作步骤），使用CAS变量（AtomicInteger等），或者synchronized加锁（lock unlock）保证原子性。**volatile不能保证原子性！**

2. 可见性

	可见性指当一个线程修改了共享变量的值，**其它线程能够立即得知这个修改**。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

	主要有三种实现可见性的方式：

	- volatile
	- synchronized，对一个变量执行 unlock 操作之前，必须把变量值同步回主内存。
	- final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值。

3. 有序性

	有序性是指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了**指令重排序**。在 Java 内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

	volatile 关键字通过添加**内存屏障**的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。

	也可以通过 synchronized 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。

