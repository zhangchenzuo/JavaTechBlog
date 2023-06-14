- [Java基础](#java基础)
  - [8种基本数据类型](#8种基本数据类型)
    - [为什么要有包装类](#为什么要有包装类)
  - [深拷贝和浅拷贝](#深拷贝和浅拷贝)
    - [创建对象的方法](#创建对象的方法)
  - [== 和 equals 的区别](#-和-equals-的区别)
  - [关键字](#关键字)
    - [final](#final)
    - [static](#static)
  - [反射](#反射)
  - [异常](#异常)
    - [try-catch-finally](#try-catch-finally)
  - [函数式编程](#函数式编程)
  - [泛型](#泛型)
  - [排序算法](#排序算法)
    - [Comparable和Comparator区别](#comparable和comparator区别)
- [常用类](#常用类)
  - [String，StringBuffer，StringBuilder](#stringstringbufferstringbuilder)
  - [接口与抽象类](#接口与抽象类)
  - [Object类内的方法](#object类内的方法)
- [面向对象](#面向对象)
  - [重载和重写的区别](#重载和重写的区别)
  - [Java和C++的区别](#java和c的区别)
- [容器](#容器)
  - [HashMap源码](#hashmap源码)
    - [Q :为什么不用AVL树，用红黑树？](#q-为什么不用avl树用红黑树)
  - [ArrayList源码](#arraylist源码)
  - [LinkedList源码](#linkedlist源码)
  - [遍历hashmap的方法](#遍历hashmap的方法)
    - [foreach 和 迭代器的区别](#foreach-和-迭代器的区别)
  - [Stream之list转成map](#stream之list转成map)
- [IO](#io)
  - [字符和字节流](#字符和字节流)
  - [IO模型](#io模型)
    - [同步/异步，阻塞/非阻塞](#同步异步阻塞非阻塞)
  - [IO多路复用模型](#io多路复用模型)
    - [select接口](#select接口)
    - [poll](#poll)
    - [epoll](#epoll)
    - [性能对比](#性能对比)
    - [水平触发与边沿触发](#水平触发与边沿触发)
- [设计模式](#设计模式)
  - [设计的六大原则](#设计的六大原则)
  - [单例模式](#单例模式)
  - [代理模式](#代理模式)
    - [静态代理、动态代理](#静态代理动态代理)
    - [动态代理的方式](#动态代理的方式)
    - [cglib和JDK代理的对比](#cglib和jdk代理的对比)
  - [工厂模式](#工厂模式)
    - [普通工厂](#普通工厂)
    - [抽象工厂](#抽象工厂)
- [Java11，17](#java1117)
- [创建对象的过程](#创建对象的过程)

# Java基础
## 8种基本数据类型
    byte/8
    char/16
    short/16
    int/32
    float/32
    long/64
    double/64
    boolean/~(1)

八种基本类型都有包装类，Byte,Short,Integer,Long,Character,Boolean的包装类实现了常量池。也就是对于[-128~127]的内容直接在常量池内进行引用。
>
    Integer i1=40；Java 在编译的时候会直接将代码封装成 Integer i1=Integer.valueOf(40);，从而使用常量池中的对象。
    Integer i1 = new Integer(40);这种情况下会创建新的对象。

### 为什么要有包装类
基本类型并不具有对象的性质，为了让基本类型也具有对象的特征，就出现了包装类型（如我们在使用集合类型Collection时就一定要使用包装类型而非基本类型），它相当于将基本类型“包装起来”，使得它具有了对象的性质，并且为其添加了属性和方法，丰富了基本类型的操作。

另外，当需要往ArrayList，HashMap中放东西时，像int，double这种基本类型是放不进去的，因为容器都是装object的，这是就需要这些基本类型的包装器类了。

装箱就是自动将基本数据类型转换为封装类型，拆箱就是自动将封装类型转换为基本数据类型。


## 深拷贝和浅拷贝

java中方法的参数的传递机制（都是传递副本）

如果一个对象被当成一个参数传递到一个方法以后，方法可以改变对象的属性，并返回。这里依然是值传递。

对于基本类型的引用，传入的是副本；对于引用类型的，传入的是传递地址的副本。因此如果我们直接修改了这个地址的指向的内容，传入的内容也收到影响（浅拷贝）。但是如果我们在方法内new了一个，副本指向新的内容。（深拷贝）

比如传入一个`PriorityQueue pq`，这个是一个副本地址，但是如果我们`pq.offer()`。函数外面的pq也改变。因此我们需要`new PriorityQueue<>(pq)`。在进行操作。

如何深拷贝：clone()是浅拷贝，只拷贝基本类型，不拷贝引用的对象。深拷贝只能序列化这个对象。

### 创建对象的方法
1. new  这个就是直接调用构造方法
2. clone  要使用clone方法，我们需要先实现Cloneable接口并实现其定义的clone方法。用clone方法创建对象并不会调用任何构造函数。clone()是浅拷贝，只拷贝基本类型，不拷贝引用的对象。深拷贝只能序列化这个对象。
3. 利用反射的newInstance 方法. 这种方式要求该Class对象的对应类有默认的构造器
4. 反序列化  为了反序列化一个对象，我们需要让我们的类实现Serializable接口.当我们序列化和反序列化一个对象，jvm会给我们创建一个单独的对象。在反序列化时，jvm创建对象并不会调用任何构造函数
5. String s = "abc"（这个是比较特殊的） 字符串线程池?

   
## == 和 equals 的区别
== ：对于基本数据类型是比较值，如果是引用类型是比较地址。注意对于Integer -127-128之间是有一个映射的，也是会比较值不是地址。

equals：是继承自object，本质也是比较地址。但是有些类进行了重写，比如Date，String等。优先比较数据类型，然后比较内容是否一致。因此对于字符串的比较需要使用equals。

使用equals的情况：1. 如果没有重写equals，就会使用object类中的方法。实际上还是比较地址。2.如果复写了equals，按照复写方法执行。一般就是先判断是不是空的，然后判断数据类型，然后判断内容是否一样。

> 推荐使用`Objects.equals(a,b)`。不会抛出空指针异常。

* **为什么重写equals时候还要重写hashCode（）？**
在map等数据结构中，判断对象相同的逻辑是：如果两个对象相等，则 hashcode 一定也是相同的。两个对象相等,对两个对象分别调用 equals 方法都返回 true。但是，两个对象有相同的 hashcode 值，它们也不一定是相等的 。因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖。

>hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

* string是如何计算哈希的？
实际上string 的底层 使用的char[] 保存的字符串，因此字符串的哈希其实也是这样计算的。s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]。31是一个素数，可以被优化为31*x == (x<<5)-x。


## 关键字
### final
1. 数据：对于基本类型不可变；对于引用类型，引用的地址不能变化。但是引用的对象本身可以修改。
2. 方法：声明的方法不可以被子类重写。
3. 类：不可以被继承


### static
1. 静态变量：在类加载时候进入方法区可以被共享。
2. 静态方法：可以直接被调用的。工具类的方法。必须实现不能是抽象的。不能依赖于实例（无this）。
3. 静态代码块：在类初次被加载时候使用。
4. 静态内部类

>静态变量和静态语句块优先于实例变量和普通语句块，静态变量和静态语句块的初始化顺序取决于它们在代码中的顺序。

``` java
public static String staticField = "静态变量";

static {

    System.out.println("静态语句块");
}

public String field = "实例变量";

{
    System.out.println("普通语句块");
}

最后才是构造函数的初始化。

public InitialOrderTest() {
    System.out.println("构造函数");
}
```

存在继承的情况下，初始化顺序为：

    父类（静态变量、静态语句块）
    子类（静态变量、静态语句块）
    父类（实例变量、普通语句块）
    父类（构造函数）
    子类（实例变量、普通语句块）
    子类（构造函数）


## 反射
定义：JAVA 反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种**动态获取的信息以及动态调用对象的方法的功能**称为 java 语言的反射机制。

举例：
    我们在使用 JDBC 连接数据库时使用 Class.forName()通过反射加载数据库的驱动程序；
    Spring 框架的 IOC（动态加载管理 Bean）创建对象以及 AOP（动态代理）功能都和反射有联系；
    动态配置实例的属性；

获取Class对象三种方式：

- 通过对象.getClass（）方式
- 通过类名.Class 方式
-  通过Class.forName 方式

缺点：
1. 性能问题：反射包括了一些动态类型，所以JVM无法对这些代码进行优化。因此，反射操作的效率要比那些非反射操作低得多。
2. 安全性问题：由于反射允许代码执行一些在正常情况下不被允许的操作（比如访问私有的属性和方法）


应用：动态代理。见下文
## 异常
分为Error和Exception。error是jvm处理不了的错误，exception分为受检异常（try...catch捕捉）与非受检（除0）。

![异常体系（错误，受检异常，不受检异常）](https://simplesnippets.tech/wp-content/uploads/2018/05/java-exception-handling-class-hierarchy-diagram-1024x615.jpg)

### try-catch-finally
1. 不管有没有异常，finally中的代码都会执行
2. 当try、catch中有return时，finally中的代码依然会继续执行
3. finally是在return后面的表达式运算之后执行的，**此时并没有返回运算之后的值，而是把值保存起来**，不管finally对该值做任何的改变，返回的值都不会改变，依然返回保存起来的值。也就是说方法的返回值是在finally运算之前就确定了的。
4. finally代码中最好不要包含return，程序会提前退出，也就是说返回的值不是try或catch中的值



## 函数式编程
是java8引入的一个核心功能。也就是lambda表达，具体就是**形参列表->函数体**。比较常用的就是在优先队列中重写排序方法时候，或者定义Runnable接口的时候。

我们最常用的面向对象编程（Java）属于命令式编程.
## 泛型
泛型，即“参数化类型”。泛型可以使用在类或者方法上。使用泛型的主要目的是**在编译的时候检查类型安全，而不是在运行时报错。**
Java中的泛型基本上都是在编译器这个层次来实现的。在生成的Java字节码中是不包含泛型中的类型信息的。**使用泛型的时候加上的类型参数，会在编译器在编译的时候去掉**。这个过程就称为类型擦除。

在编译之后程序会采取去泛型化的措施。也就是说Java中的泛型，只在**编译阶段有效**。在编译过程中，正确检验泛型结果后，会将泛型的相关信息擦出，并且在对象进入和离开方法的边界处添加类型检查和类型转换的方法。也就是说，**泛型信息不会进入到运行时阶段。**

泛型能够保证容器里存放元素的类型，但是通过运行时的反射机制能够绕过泛型的编译验证。`Method add = list.getClass().getMethod("add", Object.class);add.invoke(list,"a");`

泛型有三个类型，泛型类，泛型接口，泛型方法。泛型类就是各种容器类。一般可以用T(type 类)、K(Key)、V(Value)、N(number)、？(通配符)。

## 排序算法
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE3LmNuYmxvZ3MuY29tL2Jsb2cvODQ5NTg5LzIwMTcxMC84NDk1ODktMjAxNzEwMTUyMzMwNDMxNjgtMTg2NzgxNzg2OS5wbmc?x-oss-process=image/format,png)

### Comparable和Comparator区别
Comparable是排序接口，在类内的比较器，一般就是需要排序的类进行继承接口。本质上是静态绑定。比如String，Integer都使用到了这个接口在内部进行了实现。

Comparator是比较器，在类外使用，可以按照我们的想法进行排序。



# 常用类
## String，StringBuffer，StringBuilder
**String是不可变的对象**，因此每次在对String类进行改变的时候都会生成一个新的string对象，然后将指针指向新的string对象，所以经常要改变字符串长度的话不要使用string，因为每次生成对象都会对系统性能产生影响，特别是当内存中引用的对象多了以后，JVM的GC就会开始工作，性能就会降低；


StringBuffer类是线程安全的，StringBuilder不是线程安全的；是通过synchronize锁实现的。

## 接口与抽象类
接口：内部变量都是`static final`的。为了多继承。在java8开始，接口也可以有默认的方法实现。java9开始可以有private方法，为了复用。

抽象类：内部可以有实现的方法，抽象方法都是public或者protected。继承类需要实现全部方法。也不能创建对象。

抽象类需要满足里氏替换原则，是is-A关系，接口是has-A的关系。


## Object类内的方法
```java
public final native Class<?> getClass()//native方法，用于返回当前运行时对象的Class对象，使用了final关键字修饰，故不允许子类重写。

public native int hashCode() //native方法，用于返回对象的哈希码，主要使用在哈希表中，比如JDK中的HashMap。

public boolean equals(Object obj)//用于比较2个对象的内存地址是否相等，String类对该方法进行了重写用户比较字符串的值是否相等。

protected native Object clone() throws CloneNotSupportedException//naitive方法，用于创建并返回当前对象的一份拷贝。一般情况下，对于任何对象 x，表达式 x.clone() != x 为true，x.clone().getClass() == x.getClass() 为true。Object本身没有实现Cloneable接口，所以不重写clone方法并且进行调用的话会发生CloneNotSupportedException异常。

public String toString()//返回类的名字@实例的哈希码的16进制的字符串。建议Object所有的子类都重写这个方法。

// 下面的方法都是线程操作需要用到的。因为在对对象加锁的时候需要确保已经获得对象，因此wait和notify在object类中。
public final native void notify()//native方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。

public final native void notifyAll()//native方法，并且不能重写。跟notify一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。

public final native void wait(long timeout) throws InterruptedException//native方法，并且不能重写。暂停线程的执行。注意：sleep方法没有释放锁，而wait方法释放了锁 。timeout是等待时间。

public final void wait(long timeout, int nanos) throws InterruptedException//多了nanos参数，这个参数表示额外时间（以毫微秒为单位，范围是 0-999999）。 所以超时的时间还需要加上nanos毫秒。

public final void wait() throws InterruptedException//跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念

protected void finalize() throws Throwable { }//实例被垃圾回收器回收的时候触发的操作
```
# 面向对象
继承，多态，封装。

其中，多态一句话概括。**父类的引用可以指向子类的实例。**
## 重载和重写的区别
重载和重写都是对于多态的体现。
* 重载：同一类中同一函数名，不同的参数数量或者类型（不包括返回值）。是**类内的多态体现**，属于静态分派。在类加载时候被确定。
* 重写：子类中有与父类完全一样的函数（名字，参数，返回类型）。是**继承中多态的体现，属于动态分派**。
  
1. 子类方法的访问权限必须大于等于父类方法；
2. 子类方法的返回类型必须是父类方法返回类型或为其子类型。
3. 子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型。

> 构造器 不能被 override（重写）,但是可以 overload（重载）,所以你可以看到一个类中有多个构造函数的情况。

- 成员变量：编译看左边父类，运行也是父类。
- 成员方法：编译看父类，运行看子类。动态绑定。
- 静态方法：都看父类。因为静态与类相关。不算重写

分派（Dispatch）是指根据方法的接收者（Receiver）确定要调用的实际方法的过程。
```java
//重载：方法名相同，参数列表不同
public class StaticDispatch {
	static abstract class Human{}
	static class Man extends Human{}
	static class Woman extends Human{}
	public static void sayHello(Human guy){
		System.out.println("hello,guy!");
	}
	public static void sayHello(Man guy){
		System.out.println("hello,gentlemen!");
	}
	public static void sayHello(Woman guy){
		System.out.println("hello,lady!");
	}
	
	public static void main(String[] args) {
		Human man=new Man();
		Human woman=new Woman();
		sayHello(man);   //输出hello,guy
		sayHello(woman); //输出hello,guy
	}
}
```
编译器在重载时是通过参数的静态类型而不是实际类型作为判定的依据。并且静态类型在编译期可知，因此，编译阶段，Javac编译器会根据参数的静态类型决定使用哪个重载版本

```java
// 重写：动态分配
public class DynamicDispatch {
	static abstract class Human{
		protected abstract void sayHello();
	}
	static class Man extends Human{ 
		@Override
		protected void sayHello() { 
			System.out.println("man say hello!");
		}
	}
	static class Woman extends Human{ 
		@Override
		protected void sayHello() { 
			System.out.println("woman say hello!");
		}
	} 
	public static void main(String[] args) {
		
		Human man=new Man(); 
		Human woman=new Woman();
		man.sayHello();   //输出 man say hello
		woman.sayHello(); //输出 woman say hello
	}
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6e98ea2272ad4c1c8d4ca872fd751b4c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)
> 虚拟机的实现
> 
虚方法表中存放着各个方法的实际入口地址。如果某个方法在子类中没有被重写，那子类的虚方法表里面的地址入口和父类相同方法的地址入口是一致的，都是指向父类的实际入口。如果子类中重写了这个方法，子类方法表中的地址将会替换为指向子类实际版本的入口地址。
## Java和C++的区别
1. 都是面向对象的语言，都支持封装、继承和多态
2. Java 不提供指针来直接访问内存，程序内存更加安全
3. Java 的类是单继承的，C++ 支持多重继承；虽然 Java 的类不可以多继承，但是接口可以多继承。
4. Java 有自动内存管理垃圾回收机制(GC)，不需要程序员手动释放无用内存
# 容器
![在这里插入图片描述](https://camo.githubusercontent.com/c0506ba8f5134d89ed6a398e1c165865d50c68f6c7af4e01b75248a95e0da37d/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230383232303934383038342e706e67)

![](https://camo.githubusercontent.com/ce6470fc8cfd0f0c74ba53bd16ee9467b21c5ca7fc0566413df4701342a96a15/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303230313130313233343333353833372e706e67)

1. Set

    TreeSet：基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。

    HashSet：基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。**源码层面直接是使用的，hashmap实现的，val是一个空的Object**

    LinkedHashSet：具有 HashSet 的查找效率，并且内部使用双向链表维护元素的插入顺序。

2. List

    ArrayList：基于动态数组实现，支持随机访问。

    Vector：和 ArrayList 类似，但它是线程安全的。

    LinkedList：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。

3. Queue

    LinkedList：可以用它来实现双向队列。
    
    PriorityQueue：基于堆结构实现，可以用它来实现优先队列。

4. Map

    TreeMap：基于红黑树实现。

    HashMap：基于哈希表实现。

    HashTable：和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程同时写入 HashTable 不会导致数据不一致。它是遗留类，不应该去使用它，而是使用 ConcurrentHashMap 来支持线程安全，ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁。

    LinkedHashMap：使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。

## HashMap源码
[参考](https://tech.meituan.com/2016/06/24/java-hashmap.html)

1.8版本中，结构是数组+链表/红黑树。实际上一个Node[]节点数组构成了最底层，并且采用拉链法解决哈希冲突。将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树。链表长度大于8转换为红黑树提高效率。并且是尾插法。

采用的哈希算法：`(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)`。高16位与低16位进行异或。在最后寻找位置时候在进行`(n - 1) & hash`得到对于的索引位置。

在进行扩容的时候，并不会全部重新哈希节点。而是判断多出来的一位hash是0还是1。如果是0保持不变放入原来的桶，否则放在index+oldCap这个桶里。

>为什么高低16位异或：
增加随机性：通过将高16位和低16位进行异或，可以将原始哈希码中的高位信息混合到低位中，增加了哈希码的随机性。这样可以使得原始哈希码的一些高位特征在最终哈希码中得到保留，减少了哈希冲突的可能性。

![put方法的实现](https://camo.githubusercontent.com/6e61b336220f0690540fad2acc0d8c19106a32b278768582cb3e973a25a061b6/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d372f7075742545362539362542392545362542332539352e706e67)

参考文献：[详解HashMap](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/collection/HashMap(JDK1.8)%E6%BA%90%E7%A0%81+%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%88%86%E6%9E%90.md)



>补充：

 ① 在 JDK1.7 的时候，ConcurrentHashMap（分段锁） 对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 到了 JDK1.8 的时候已经摒弃了 Segment 的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作。（JDK1.6 以后 对 synchronized 锁做了很多优化） 整个看起来就像是优化过且线程安全的 HashMap，虽然在 JDK1.8 中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本。synchronized 只锁定当前链表或红黑二叉树的**首节点**，这样只要 hash 不冲突，就不会产生并发，效率又提升 N 倍。

 ② Hashtable(同一把锁) :使用 synchronized 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。



### Q :为什么不用AVL树，用红黑树？

A： **红黑树与AVL树都是平衡的。但是AVL的平衡更严格，适用于查询多（O（1）），增删少的场景。红黑树更适合插入和删除。** 因为可以他的不平衡都可以在三次以内旋转解决。就插入节点导致树失衡的情况，AVL和RB-Tree都是最多两次树旋转来实现复衡rebalance，旋转的量级是O(1)。删除节点导致失衡，AVL需要维护从被删除节点到根节点root这条路径上所有节点的平衡，旋转的量级为O(logN)，而RB-Tree最多只需要旋转3次实现复衡，只需O(1)，所以说RB-Tree删除节点的rebalance的效率更高，开销更小！

* 什么是[红黑树（美团技术博客）](https://tech.meituan.com/2016/12/02/redblack-tree.html)？


（1）每个节点或者是黑色，或者是红色。
（2）根节点是黑色。
（3）每个叶子节点（NIL）是黑色。 注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！
（4）如果一个节点是红色的，则它的子节点必须是黑色的。
（5）从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210325233128501.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)

> 为什么选择在长度为8的时候转换

默认是链表长度达到 8 就转成红黑树，而当长度降到 6 就转换回去，这体现了时间和空间平衡的思想。使用链表存储需要的空间很少，但是查询复杂度高。

理想情况下，在随机哈希代码下，桶中的节点频率遵循泊松分布，由频率表可以看出，桶的长度超过8的概率非常非常小，概率仅为 0.00000006。所以作者应该是根据概率统计而选择了8作为阀值，由此可见，这个选择是非常严谨和科学的。

> 扩容以后重新计算索引
计算索引的的方法就是与11111与，扩容以后其实就是多了一位二进制，如果是1，就对应到新的索引位置。否则不变。
## ArrayList源码
ArrayList 是 List 的主要实现类，底层使用 Object[ ]存储，适用于频繁的查找工作，线程不安全 ；Vector 是 List 的古老实现类，底层使用 Object[ ]存储，线程安全的。

以无参数构造方法创建 ArrayList 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为 10。

每次扩容时候扩大为旧的1.5倍。

## LinkedList源码
LinkedList是一个实现了List接口和Deque接口的双端链表。LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）。





## 遍历hashmap的方法
1. foreach

```java
　　Map map = new HashMap();
　　Iterator iter = map.entrySet().iterator();
　　while (iter.hasNext()) {
    　　Map.Entry entry = (Map.Entry) iter.next();
    　　Object key = entry.getKey();
    　　Object val = entry.getValue();
　　}

　　Map map = new HashMap();
　　Iterator iter = map.keySet().iterator();
　　while (iter.hasNext()) {
    　　Object key = iter.next();
    　　Object val = map.get(key);
　　}
```
2. 迭代器
```java

Map<String, String> map = new HashMap<String, String>();
for (String key : map.keySet()) {
	map.get(key);
}

Map<String, String> map = new HashMap<String, String>();
for (Entry<String, String> entry : map.entrySet()) {
	entry.getKey();
	entry.getValue();
}
```

### foreach 和 迭代器的区别
Foreach通常用于一次性遍历整个集合，通常不会暂停，大大提升了代码的简洁性和可阅读性。而Iterator可以更好地控制便利过程的每一步。

**Foreach在遍历过程中严禁改变集合的长度**，进行对集合的删除或添加等操作，**而使用Iterator可以在遍历过程中对集合元素进行删除操作**。**Iterator中的remove()方法只能删除当前迭代器返回的最后一个元素，也就是说，每调用一次next()只能调用一次remove()**，如果要在遍历过程中对集合添加元素，需要使用ListIterator，是List专用。

迭代器模式可以和组合模式一起使用，来控制树状结构的遍历。

**Foreach的内部实现是通过迭代器来完成的**，实现Iterable接口的类可以使用Foreach句型，Iterable中的iterator()方法返回遍历所用的迭代器。虽然数组也可以使用Foreach句型，但数组并不是Iterable。
   
**foreach需要知道自己的集合类型，甚至要知道自己集合内的元素类型，不能实现多态。** 这个使用的语法上都可以表示出来。foreach可以遍历任何集合或者数组，但是使用者需要知道遍历元素的类型。

Iterator是一个接口类型，它不关心集合的类型和集合内的元素类型，**因为它是通过hasnext和next来进行下一个元素的判断和获取**，这一切都是在集合类型定义的时候就完成的事情。迭代器统一了对容器的访问模式，这也是对接口解耦的最好表现

## Stream之list转成map
将list集合转换成Map，使用Stream的Collectors的toMap方法。
```java
//声明一个List集合
List<Person> list = new ArrayList();  
        list.add(new Person("1001", "小A"));  
        list.add(new Person("1002", "小B"));  
        list.add(new Person("1003", "小C"));
        System.out.println(list);
//将list转换map
Map<String, String> map = list.stream().collect(Collectors.toMap(Person.getId(), Person.getName()));

```


有可能存在报错，key值重复，可以考虑覆盖或者拼接。
`list.stream().collect(Collectors.toMap(Person::getId, Person::getName,(key1, key2)-> key2 )`

`Map<String, String> map = list.stream().collect(Collectors.toMap(Person::getId, Person::getName,(key1 , key2)-> key1+","+key2 ));`

还一个collectors.groupingBy()；方法可以实现value为list的map。
# IO
## 字符和字节流
- 字节流【byte = 8bit】
- 字符流【char = 2 * byte = 16bit】
  
底层设备永远只接受字节数据，有时候要写字符串到底层设备，需要将字符串转成字节再进行写入。

字符流是字节流的包装，字符流则是直接接受字符串，它内部将串转成字节，再写入底层设备，这为我们向 IO 设别写入或读取字符串提供了一点点方便。

![在这里插入图片描述](https://img-blog.csdnimg.cn/4d349d1d770a43c9970a3d805d305cf3.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)
## IO模型
* BIO 阻塞IO模型，也就是最基础的IO模型。是面向流的，是基于字符流和字节流的。线程一旦开始读取或者写入就只能一直进行。**适用于在活动连接数不是特别高的场景。**

 用户进程调用内核态kernel线程，kernel线程读取数据直到完成，然后将数据从kernel中拷贝到用户内存，然后kernel返回结果，用户进程才解除block的状态。

* NIO 新的IO模型。依赖于缓存区和通道。数据总是从缓存区写入通道，或者从通道读入缓存区。 kernel读取好了时候，进程来访问kernel，然后kernel拷贝到用户缓存区，然后返回ok。

NIO 实现了 IO 多路复用中的 Reactor 模型，一个线程 Thread 使用一个选择器 Selector 通过轮询的方式去监听多个通道 Channel 上的事件，从而让一个线程就可以处理多个事件。因为创建和切换线程的开销很大，因此使用一个线程来处理多个事件而不是一个线程处理一个事件，对于 IO 密集型的应用具有很好地性能。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210312150907372.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)

监听的通道 Channel 为非阻塞，那么当 Channel 上的 IO 事件还未到达时，就不会进入阻塞状态一直等待，而是继续轮询其它 Channel，找到 IO 事件已经到达的 Channel 执行。

- 高并发性：NIO 模型使用单线程或少量线程处理多个连接，通过事件驱动的方式处理 I/O 操作，因此能够处理大量并发连接。这种非阻塞的特性使得一个线程可以同时处理多个连接，避免了传统阻塞 I/O 模型中每个连接都需要一个独立线程的问题。

- 资源占用低

- 快速响应：NIO 模型通过事件驱动的方式处理 I/O 操作，当有数据可读或可写时立即触发相应的事件，而不需要等待阻塞或轮询。这使得应用程序能够更快地响应事件，并及时处理数据，减少了等待时间和延迟。

- 支持非阻塞操作：NIO 模型提供了非阻塞的 I/O 操作，允许应用程序在进行 I/O 操作时不必等待数据的到达或数据的发送完成。应用程序可以继续执行其他任务，当数据就绪时再进行读取或写入，提高了系统的吞吐量和效率。


* AIO 异步IO模型。当线程从通道读取数据到缓冲区时候，线程可以进行别的任务。读入完毕，再去缓存操作。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。


### 同步/异步，阻塞/非阻塞
同步和异步：线程的调用是1.调用就得到结果还是 2.立即返回，有结果的时候再通知

阻塞和非阻塞：线程在得到调用结果之前能不能干别的事情
## IO多路复用模型
IO多路复用是要和NIO一起使用的。尽管在操作系统级别，NIO和IO多路复用是两个相对独立的事情。NIO仅仅是指IO API总是能立刻返回，不会被Blocking；而IO多路复用仅仅是操作系统提供的一种便利的通知机制。

对IO多路复用，还存在一些常见的误解，比如：

❌IO多路复用是指多个数据流共享同一个Socket。其实IO多路复用说的是多个Socket，只不过操作系统是一起监听他们的事件而已。

❌IO多路复用是NIO，所以总是不Block的。其实IO多路复用的关键API调用(select，poll，epoll_wait）总是Block的，正如下文的例子所讲。

❌IO多路复用和NIO一起减少了IO。实际上，IO本身（网络数据的收发）无论用不用IO多路复用和NIO，都没有变化。请求的数据该是多少还是多少；网络上该传输多少数据还是多少数据。IO多路复用和NIO一起仅仅是解决了调度的问题，避免CPU在这个过程中的浪费，使系统的瓶颈更容易触达到网络带宽，而非CPU或者内存。要提高IO吞吐，还是提高硬件的容量（例如，用支持更大带宽的网线、网卡和交换机）和依靠并发传输（例如HDFS的数据多副本并发传输）

IO多路复用是由一些接口来支持的，包括以下
### select接口
```java
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```
它接受3个文件描述符的数组，分别监听读取(readfds)，写入(writefds)和异常(expectfds)事件。

首先，为了select需要构造一个fd数组。之后，用select监听了read_fds中的多个socket的读取时间。调用select后，程序会Block住，直到一个事件发生了，或者等到最大1秒钟(tv定义了这个时间长度）就返回。之后，需要遍历所有注册的fd，挨个检查哪个fd有事件到达(FD_ISSET返回true)。如果是，就说明数据已经到达了，可以读取fd了。读取后就可以进行数据的处理。

* 缺点：
1. select能够支持的最大的fd数组的长度是1024。这对要处理高并发的web服务器是不可接受的。
2. fd数组按照监听的事件分为了3个数组，而且每次调用select前都要重设它们（因为select会改这3个数组)；调用select后，这3数组要从用户态复制一份到内核态；事件到达后，要遍历这3数组。很不爽。
3. select返回后要挨个遍历fd，找到被“SET”的那些进行处理。这样比较低效。
4. select是无状态的，即每次调用select，内核都要重新检查所有被注册的fd的状态。select返回后，这些状态就被返回了，内核不会记住它们；到了下一次调用，内核依然要重新检查一遍。于是查询的效率很低。

### poll
简单的升级`int poll(struct pollfd *fds, nfds_t nfds, int timeout);`poll优化了select的一些问题。比如不再有3个数组，而是1个polldfd结构的数组了，并且也不需要每次重设了。数组的个数也没有了1024的限制。

但其他的问题依旧：
1. 依然是无状态的，性能的问题与select差不多一样；
2. 应用程序仍然无法很方便的拿到那些“有事件发生的fd“，还是需要遍历所有注册的fd。
### epoll
epoll是Linux下的IO多路复用的实现。与select和poll不同，要使用epoll是需要先创建一下的。`int epfd = epoll_create(10);`epoll_create在内核层创建了一个数据表（维护一个数据结构来存放文件描述符，并且时常会有插入，查找和删除的操作发生。因此使用了红黑树），接口会返回一个“epoll的文件描述符”指向这个表。epoll是有状态的，不像select和poll那样每次都要重新传入所有要监听的fd，这避免了很多无谓的数据复制。epoll的数据是用接口epoll_ctl来管理的（增、删、改）

第二步是使用epoll_ctl接口来注册要监听的事件。`int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);`,第三步使用epoll_wait来等待事件的发生`int epoll_wait(int epfd, struct epoll_event *evlist, int maxevents, int timeout);`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210312151536213.png)

### 性能对比
相比于select，epoll最大的好处在于它不会随着监听fd数目的增长而降低效率。因为在内核中的select实现中，它是采用轮询来处理的，轮询的fd数目越多，自然耗时越多。

epoll相比于select并不是在所有情况下都要高效，例如在如果有少于1024个文件描述符监听，且大多数socket都是出于活跃繁忙的状态，这种情况下，select要比epoll更为高效，因为epoll会有更多次的系统调用，用户态和内核态会有更加频繁的切换。

epoll除了性能优势，还有一个优点——同时支持水平触发(Level Trigger)和边沿触发(Edge Trigger)。

### 水平触发与边沿触发


- 水平触发level trigger  LT（状态达到）

当被监控的文件描述符上有可读写事件发生时，会通知用户程序去读写，如果用户一次读写没取完数据，他会**一直通知用户**。

缺点：如果这个描述符是用户不关心的，它每次都返回通知用户，则会导致用户对于关心的描述符的处理效率降低。

复用型IO中的select和poll都是使用的水平触发方式。

- 边缘触发edge trigger  ET（状态改变）

　　当被监控的文件描述符上有可读写事件发生时，会通知用户程序去读写，**它只会通知用户进程一次**，这需要用户一次把内容读取完，相对于水平触发，效率更高。

缺点：如果用户一次没有读完数据，再次请求时，不会立即返回，需要等待下一次的新的数据到来时才会返回，这次返回的内容包括上次未取完的数据。

epoll既支持水平触发也支持边缘触发，默认是水平触发。

- 比较

　　水平触发是状态达到后，可以多次取数据。这种模式下要注意多次读写的情况下，效率和资源利用率情况。

边缘触发是状态改变一次，取一次数据。这种模式下读写数据要注意一次是否能读写完成。

在水平触发的情况下，必须不断的轮询监控每个文件描述符的状态，判断其是否可读或可写。内核空间中维护的 I/O 状态列表可能随时会被更新，因此用户程序想要拿到 I/O 状态列表必须访问内核空间。

而边缘触发的情况下，只有在数据到达网卡，也就是说 I/O 状态发生改变时才会触发事件，在两次数据到达的间隙，I/O 状态列表是不会发生改变的。这就使得用户程序可以缓存一份 I/O 状态列表在用户空间中，减少系统调用的次数。
# 设计模式
## 设计的六大原则
1. 单一职责：一个类不应该承担太多的原则。
2. 开放封闭原则：类应该是可以拓展的，但是对于修改应该是封闭的。增加一个抽象的**功能类**，让增加和删除和查询的作为这个抽象功能类的子类，这样如果我们再添加功能，你会发现我们不需要修改原有的类，只需要添加一个功能类的子类实现功能类的方法就可以了。
3. 里氏替换原则：所用引用父类的地方都可以用子类进行替代。因此在程序中尽量把父类设计为抽象类或者接口，尽量使用**父类类型来对对象进行定义**，而在运行时再确定其子类类型，用子类对象来替换父类对象。 
4. 依赖倒置原则：高层模块不应该依赖于底层模块，而是应该依赖于抽象。实现类也依赖于抽象。**模块间通过抽象发生，实现类之间不发生直接依赖关系，其依赖关系是通过接口或者抽象类产生的**。
5. 迪米特原则：软件实体之间尽可能减少耦合。
6. 接口隔离原则：建立单一接口，尽量细化接口，接口中的方法尽量的少。

合成聚合复用原则：优先使用聚合或合成关系复用代码。合成聚合复用想表达的是优先考虑关联（Has-A）关系，而不是继承（Is-A）关系复用代码。

## 单例模式
1. 双重检验
 
同步块内部再进行一次检查。并且需要对实例加上`volatile`
* 优点：线程安全，延迟加载，效率高
* 为什么使用`volatile`修饰实例，利用防止重排序，保证完整的新建实例；保证可见性（其实没必要，因为synchronize的已经解决了）。

```java
public class Singleton {
    //
    private volatile static Singleton uniqueInstance;

    private Singleton() {
    }

    public static Singleton getUniqueInstance() {
        if (uniqueInstance == null) {
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```
>为什么使用volatile：

对于`uniqueInstance = new Singleton();`实际上是三个大步骤：创建内存空间；在内存空间中初始化对象 Singleton；将内存地址赋值给 instance 对象(执行了此步骤，instance 就不等于 null 了)。

试想一下，如果不加 volatile，那么线程 1 在可能会执行指令重排序，将原本是 1、2、3 的执行顺序，重排为 1、3、2。但是特殊情况下，线程 1 在执行完第 3 步之后，如果来了线程 2 执行到上述代码的null判断，判断 instance 对象已经不为 null，但此时线程 1 还未将对象实例化完，那么线程 2 将会得到一个被实例化“一半”的对象，从而导致程序执行出错，这就是为什么要给私有变量添加 volatile 的原因了。

2. 静态内部类

当 Singleton 类被加载时，静态内部类 SingletonHolder 没有被加载进内存。只有当调用 getUniqueInstance() 方法从而触发 SingletonHolder.INSTANCE 时 SingletonHolder 才会被加载，此时初始化 INSTANCE 实例，并且 JVM 能确保 INSTANCE 只被实例化一次。

这种方式不仅具有**延迟初始化**的好处，而且由 **JVM 提供了对线程安全的支持**。
```java
public class Singleton {

    private Singleton() {
    }

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getUniqueInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```


3. 枚举实现
该实现可以防止反射攻击。在其它实现中，通过 setAccessible() 方法可以将私有构造函数的访问级别设置为 public，然后调用构造函数从而实例化对象，如果要防止这种攻击，需要在构造函数中添加防止多次实例化的代码。该实现是由 JVM 保证只会实例化一次，因此不会出现上述的反射攻击。
```java
public enum Singleton {

    INSTANCE;

    private String objName;


    public String getObjName() {
        return objName;
    }
    public void setObjName(String objName) {
        this.objName = objName;
    }
    //对外暴露一个获取User对象的静态方法
    public static User getInstance(){
        return SingletonEnum.INSTANCE.getInstance();
    }

}
```
## 代理模式
### 静态代理、动态代理
静态代理“代理类Proxy的Java代码在JVM运行时就已经确定了，也就是在编码编译阶段就确定了Proxy类的代码。静态代理需要我们提前编码代理类。

动态代理：是指在JVM运行过程中，动态的创建一个类的代理类，并实例化代理对象。实际的代理类是在运行时创建的。动态代理可以使用反射，让jvm帮我们生成这个代理类。我们只需要告诉他规则。

>应用: Spring中的AOP就是一个动态代理的典型应用，有了动态代理

### 动态代理的方式
[设计模式（11）动态代理 JDK VS CGLIB面试必问](https://www.jianshu.com/p/3caa0c23a157)

- 基于JDK实现的动态代理。
- 基于CGLIB类库实现的动态代理。

>基于JDK实现的动态代理。

通过接口的方式。

![](https://upload-images.jianshu.io/upload_images/7432604-a9eb9262c0c8f2ea.png?imageMogr2/auto-orient/strip|imageView2/2/w/767)

1. 创建被代理对象的接口类 subject。
2. 创建具体被代理对象接口的实现类 realSubject。
3. 创建一个InvocationHandler的实现类`ConcreteInvocationHandler `，并持有被代理对象`realSubject`的引用。然后在invoke方法中利用反射调用被代理对象的方法。
4. 利用Proxy.newProxyInstance方法创建代理对象`proxy`，利用代理对象实现真实对象方法的调用。

```java
public class ConcreteInvocationHandler implements InvocationHandler {

    private Subject subject;

    public ConcreteInvocationHandler(Subject subject) {
        this.subject = subject;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        return method.invoke(subject, args);
    }
}

public class JDKDynamicProxyTest {
    public static void main(String[] args) {
        Subject subject = new RealSubject();
        InvocationHandler handler = new ConcreteInvocationHandler(subject);
        // create proxy
        Subject proxy = (Subject)Proxy.
        newProxyInstance(RealSubject.class.getClassLoader(),
                RealSubject.class.getInterfaces(), handler);
        proxy.request();
    }
}
```

> cglib动态代理

通过继承父类的方法。

![在这里插入图片描述](https://img-blog.csdnimg.cn/b27f3e9628414f12a55dfb399d22ec01.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)


1. 实现一个方法拦截器`TargetMethodInterceptor `拦截原本实现类的方法。
2. 实现一个增强类`enhancer`，使其父类是实现类。
3. 给增强类添加拦截器。
4. 生成增强类的实例 代理类`proxy`
```java
// 拦截器
public class TargetMethodInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object obj, Method method, Object[] args, 
                            MethodProxy proxy) throws Throwable {
        System.out.println("方法拦截增强逻辑-前置处理执行");
        Object result = proxy.invokeSuper(obj, args);
        System.out.println("方法拦截增强逻辑-后置处理执行");
        return result;
    }
}

public class CglibDynamicProxyTest {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        // 设置生成代理类的父类class对象
        enhancer.setSuperclass(Target.class);
        // 设置增强目标类的方法拦截器
        MethodInterceptor methodInterceptor = new TargetMethodInterceptor();
        enhancer.setCallback(methodInterceptor);
        // 生成代理类并实例化
        Target proxy = (Target) enhancer.create();
        // 用代理类调用方法
        proxy.request();
    }
}
```

### cglib和JDK代理的对比
- 字节码创建方式：JDK动态代理通过JVM实现代理类字节码的创建，cglib通过ASM创建字节码。
- JDK动态代理强制要求目标类必须实现了某一接口，否则无法进行代理。而CGLIB则要求目标类和目标方法不能是final的，因为CGLIB通过继承的方式实现代理。
- JDK快于CGLIB。


## 工厂模式
定义：定义一个用于创建对象的接口，让子类决定实例化哪个类。将一个类的实例化延迟到其子类。

适用场景：类并不清楚自己需要创建哪个对象，使用子类来指定创建哪个对象，客户端需要清楚创建哪个对象。

源码体现：spring的bean工厂

### 普通工厂
![](https://img-blog.csdn.net/20180217161953740)

存在一个抽象工厂，具体的工厂类实现这个抽象类。存在一个抽象产品类，具体的产品类实现这个类。

在具体的工厂内部判断具体的实例方法，去执行不同的操作。
[工厂模式](https://liuwangshu.blog.csdn.net/article/details/52326959)

简单工厂模式中，如果我们需要添加工厂就需要在工厂类中添加case，破坏了封闭开放原则。而在工厂模式中，我们可以通过继承抽象工厂的方法实现拓展。




### 抽象工厂
![](https://img-blog.csdn.net/20180217162019196)

本质上就是将不同工厂都进一步抽象成了一个抽象工厂。比如我们需要连接数据库，可能有不同的数据库需要操作（mysql或者olrcal），我们可以采用反射的方法得到具体的类，然后再产生相应的工厂。

使用就是`ApplicationContext context = new FileSystemXmlApplicationContext(`

# Java11，17
Java11版本的重大特点：
1. 引入var本地变量类型推断
2. collection的方法升级List.of()、Set.of()、Map.of() 
3. GC升级，G1，ZCG，Epsilon

Java17：
- 更好的支持switch
- 封闭类可以是封闭类和或者封闭接口，用来增强 Java 编程语言，防止其他类或接口扩展或实现它们。

# 创建对象的过程
- Step1：类加载检查：首先它会检查这个指令是否能在常量池中能否定位到一个类的符号引用。
接着会检查这个符号引用代表的类是否已经被加载、解析、初始化。如果没有会进行一个**类加载**
- Step2：分配内存：主要有两种分配方式：指针碰撞；空闲列表。多线程保证安全：将分配内存空间的动作进行同步处理（虚拟机底层的实现逻辑就是CAS + 失败重试）来保证分配内存空间的原子性。或者本地线程分配缓冲 Thread Local Allocation Buffer TLAB
- Step3：初始零值：当分配完内存后，虚拟机必须将分配到的内存空间（不包含对象头）都初始化为零值。
- Step4：设置对象头：va 虚拟机还需要对这些对象进行必要的设置，例如这些对象是哪些类的实例、以及如何才能找到类的元信息、对象的哈希码（实际对象的哈希码会延期到真正调用 Object::hashCode()方法时才计算）、对象 GC 的分代年龄等信息，这些信息都会保存在对象头中（Object Header）之中。 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式
- Step5：执行 init：对于 Java 视角来说，对象的创建才刚刚开始，还没有执行init方法。所有的字段还都为零。对象中需要的其它资源和状态信息还没有按照原有的意图去构造好。所以一般来说，new指令之后就会执行init方法，按照 Java 程序员的意图去对对象做一个初始化，这样之后一个真正完整可用的对象才构造出来