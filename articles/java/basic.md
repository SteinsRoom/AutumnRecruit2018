
<!-- TOC -->

- [关键字](#关键字)
    - [线程相关](#线程相关)
        - [__Java的内存机制__](#__java的内存机制__)
    - [final](#final)
        - [类](#类)
        - [方法](#方法)
        - [变量](#变量)
    - [static](#static)
        - [方法](#方法-1)
        - [变量](#变量-1)
        - [代码块](#代码块)
    - [transient](#transient)
    - [ThreadLocal](#threadlocal)
- [泛型](#泛型)
    - [为什么引入泛型？](#为什么引入泛型)
    - [泛型的优点](#泛型的优点)
    - [泛型类型擦除](#泛型类型擦除)
    - [java和C++的最大区别](#java和c的最大区别)

<!-- /TOC -->

# 关键字

## 线程相关
* synchronized
    
* volatile  
变量修饰符，只能用来修饰变量。volatile修饰的成员变量在每次被线程访问时，都强迫从共享内存中重读该成员变量的值。而且，当成员变量发生变化时，强迫线程将变化值回写到共享内存。这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值。

### __Java的内存机制__

Java使用一个主内存来保存变量当前值，而每个线程则有其独立的工作内存。线程访问变量的时候会将变量的值拷贝到自己的工作内存中，这样，当线程对自己工作内存中的变量进行操作之后，就造成了工作内存中的变量拷贝的值与主内存中的变量值不同。  

Java语言规范中指出：为了获得最佳速度，允许线程保存共享成员变量的私有拷贝，而且只当线程进入或者离开同步代码块时才与共享成员变量的原始值对比。  

这样当多个线程同时与某个对象交互时，就必须要注意到要让线程及时的得到共享成员变量的变化。  

而volatile关键字就是提示VM：对于这个成员变量不能保存它的私有拷贝，而应直接与共享成员变量交互。  

使用建议：在两个或者更多的线程访问的成员变量上使用volatile。当要访问的变量已在synchronized代码块中，或者为常量时，不必使用。

由于使用volatile屏蔽掉了VM中必要的代码优化，所以在效率上比较低，因此一定在必要时才使用此关键字。


## final
final关键字可以用来修饰类、方法和变量（包括成员变量和局部变量）。
### 类
当用final修饰一个类，表明这个类永远不会被继承。同时类的成员方法也会被隐式的指定为final方法。

### 方法
下面这段话摘自《Java编程思想》第四版第143页：

> 使用final方法的原因有两个。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升。在最近的Java版本中，不需要使用final方法进行这些优化了。

**private方法会隐式的指定为final方法**

### 变量
* 永远不变的常量
* 运行时被初始化，而且不希望变量值被改变
* 对于一个final变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。

```java 
final int value = 1;
//value = 2;    //v不能被赋值
final Test test = new Test();
test.value = 2;
Test test1 = new Test();
//test = test1;     //test不能被改变引用
```

## static
### 方法
static方法一般称作静态方法，由于静态方法不依赖于任何对象就可以进行访问，因此对于静态方法来说，是没有this的，因为它不依附于任何对象，既然都没有对象，就谈不上this了。并且由于这个特性，在静态方法中不能访问类的非静态成员变量和非静态成员方法，因为非静态成员方法/变量都是必须依赖具体的对象才能够被调用。

但是要注意的是，虽然在静态方法中不能访问非静态成员方法和非静态成员变量，但是在非静态成员方法中是可以访问静态成员方法/变量的。

### 变量
static变量也称作静态变量，静态变量和非静态变量的区别是：静态变量被所有的对象所共享，在内存中只有一个副本，它当且仅当在类初次加载时会被初始化。而非静态变量是对象所拥有的，在创建对象的时候被初始化，存在多个副本，各个对象拥有的副本互不影响。

static成员变量的初始化顺序按照定义的顺序进行初始化。

### 代码块
static关键字还有一个比较关键的作用就是 用来形成静态代码块以优化程序性能。static块可以置于类中的任何地方，类中可以有多个static块。在类初次被加载的时候，会按照static块的顺序来执行每个static块，并且只会执行一次。

　　为什么说static块可以用来优化程序性能，是因为它的特性:只会在类加载的时候执行一次。下面看个例子:
```java
class Person{
    private Date birthDate;
     
    public Person(Date birthDate) {
        this.birthDate = birthDate;
    }
     
    boolean isBornBoomer() {
        Date startDate = Date.valueOf("1946");
        Date endDate = Date.valueOf("1964");
        return birthDate.compareTo(startDate)>=0 && birthDate.compareTo(endDate) < 0;
    }
}
```
```java
class Person{
    private Date birthDate;
    private static Date startDate,endDate;
    static{
        startDate = Date.valueOf("1946");
        endDate = Date.valueOf("1964");
    }
     
    public Person(Date birthDate) {
        this.birthDate = birthDate;
    }
     
    boolean isBornBoomer() {
        return birthDate.compareTo(startDate)>=0 && birthDate.compareTo(endDate) < 0;
    }
}
```

## transient  
类型修饰符，只能用来修饰字段。被transient标记的变量不会被序列化。
```java
class Test {  
    transient int a; // 不会被持久化  
    int b; // 持久化  
}
```

## ThreadLocal

# 泛型
## 为什么引入泛型？
在泛型之前，使用的是Object，但是Object有以下缺点
* 每次获取到值都需要强转，类型可能转换失败
* 错误只能再运行时抛出

## 泛型的优点
* 类型安全
    * 泛型的主要目标就是提高java的类型安全
    * 编译时期就可以检查出类型不正确导致的ClassCastException异常
    * 符合越早出错代价越小的原则
* 消除强制类型转换
    * 可以直接得到目标类型，消除许多强制类型转换
    * 所得即所需，提高代码的可读性
* 潜在的性能收益
    * 泛型的实现方式，支持泛型(几乎)不需要修改JVM或类文件
    * 所有工作都在编译其中完成
    * 编译器生成的代码跟不使用泛型(和强制类型转换)时所写的代码一致，更能确保类型安全

## 泛型类型擦除
## java和C++的最大区别
* c++中模板实例化会对每种类型产生一套不同的代码，这就是代码膨胀
* java并不会产生这个问题，JVM中没有泛型类型的对象，所有对象都是普通类。