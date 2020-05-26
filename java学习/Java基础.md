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
装箱过程中自动调用valueOf。

缓冲池大小： 
boolean true false
byte all
short -128~127
int -128~127
char \u0000 to \u007F（也是127）

## 二、String
String是final类型的类，无法被继承。Java8中使用char数组存储String。Java9使用byte数组，并且使用coder标识编码类型。类型中的value数组也是final类型的，初始化后不能再引用其他数组，并且String类内部也无法改变value，所以可以保证String是不可变的。

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
StringBuffer不安全
StringBuilder使用了synchronized，线程安全

### String Pool
new String("abc")不会把创建新对象放在String Pool中，而是放在堆中；String对象的intern方法会在StringPool中寻找是否存在对象（使用equals），不存在则在String Pool中新建，并返回地址。```String s2 = s1.intern();```。

```java
String s = "abc";
```

这种方式创建的String，先去String Pool中匹配相应的对象，不存在则新建对象。