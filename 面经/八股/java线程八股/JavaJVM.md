# JVM

JVM的职责时运行Java字节码文件

java文件——>(通过javac编译) Java字节码文件(Hello.class)——>(通过Java运行) Java虚拟机 (虚拟机将字节码翻译成机器码)

## jvm的内部组成

![image-20240729154900374](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240729154900374.png)



## 字节码

字节码是以二进制方式存储的，无法用记事本直接打开。

## 方法区

方法区是用来存储每个类的基本信息，一般称之为 InstanceKlass 对象。在类的加载阶段完成。

### 运行时常量池

![image-20240731164439385](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240731164439385.png)

运行时常量池中存放的是字节码中常量池内容。当常量池被加载到内存之后，可以通过内存地址快速的定位到常量池中的内容，这种常量池称为**运行时常量池**。由静态常量池的符号引用转变为运行时常量池的直接引用。

### 字符串常量池

方法区除了类的元信息，运行时常量池之外，还有一块区域叫字符串常量池。

字符串常量池会存储在代码中定义的常量字符串内容。

### String.intern( )

String.intern( )可以将字符串手动放入字符串常量池中

```java

```





Java虚拟机控制内存，不容易出现内存泄漏和内存溢出问题。

![image-20240415145157040](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240415145157040.png)

线程私有的：

- 程序计数器
- 虚拟机栈
- 本地方法栈

线程共享的：

- 堆
- 方法区
- 直接内存（非运行时数据区的一部分）

## 对象创建

### 1. 类加载检查

遇到一个new指令，先检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否被加载过，解析和初始化过。如果没有，必须执行相应的类加载过程。

### 2. 分配内存

在类加载检查后，虚拟机将为新生对象分配内存，对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于一块确定大小的内存从Java堆中划分出来，分配方式有“**指针碰撞**”、“**空闲列表**”。具体选择由Java堆是否规整决定，Java堆规整又由垃圾收集器是否带有压缩整理功能决定。

#### * 内存分配的两种方式

#### * 内存分配并发问题

### 3. 初始化零值

内存分配完，jvm会将分配到的内存空间都初始化为零值

### 4. 设置对象头

初始化零值完成之后，**虚拟机要对对象进行必要的设置**

### 5. 执行init方法

等上面的方法都完成，jvm已经完成对象的生产，但是Java程序的视角，对象还没创建，所以在new指令之后，执行<init>方法。





## Java垃圾回收

需要排查各种内存溢出问题。主要针对对象内存的挥手和对象内存的分配，Java自动内存管理最核心的功能是堆内存中对象的分配和回收。

Java堆中被划分为新生代、老生代、永久代。

1. 分配内存首先在新生代分配内存，当eden区没有足够空间进行分配时，虚拟机将发起一次minor GC
2. 将新生代的对象转移到老年代中

### 方法区的回收

- 方法区中能回收的内容主要就是不再使用的类。

判断一个类可以被卸载，需要同时满足三个条件

1. 此类所有实例对象都已经被回收，在堆中不存在人和该类的实例对象和子类对象
2. 加载该类的类加载器已经被回收
3. 该类对象的 java.lang.Class 对象没有在任何地方被引用

### 判断对象是否为垃圾

#### 1. 引用计数算法

当Object的对象实例被创建出来后，计数器会被初始化为1，因为局部变量obj的指针引用了该实例对象。而后续执行过程中，又有另外一个变量引用该实例时，该对象的引用计数器会+1。而当方法执行结束，栈帧中局部变量表中引用该对象的指针随之销毁时，当前对象的引用计数器会-1。当一个对象的计数器为0时，代表当前对象已经没有指针引用它了，那么在GC发生时，该对象会被判定为“垃圾”，然后会被回收。
但是该算法无法处理两个对象相互引用的这种引用循环情况。

#### 2. 可达性分析算法

在该算法中存在一个GCRoots的概念，在GC发生时，会以这些GCRoots作为根节点，然后从上至下的方式进行搜索分析，搜索走过的路线则被称为Reference Chain引用链。当一个对象没有任何引用链相连时，则会被判定为该对象是不可达的，即代表着此对象不可用，最终该对象会被判定为“垃圾”对象等待回收。

#### 对象的finalization机制

在Java中对象可触及性分为三类

①可触及的：存在于引用链上的对象则是可触及对象，也就是指通过根节点是可以找到的对象。
②可复活的：一旦当一个对象的所有引用被释放，那么它就会处于可复活状态，因为在finalize()中可能复活该对象。
③不可触及的：在finalize()执行后，对象会进入不可触及状态，从此该对象没有机会再次复活，只能等待被GC机制回收。

当垃圾回收器发现一个对象没有引用指向时，那么在GC之前，总会先调用这个对象的`finalize()`方法。但如果该对象所属的类没有重写`finalize()`方法或已经执行过一次该方法了，最终则不会再执行`finalize()`方法。

如果一个对象执行`finalize()`方法过程中，与引用链上的任何一个对象建立了联系，那么该对象会被移出队列，然后标记为存活对象。

```java
public class Finalization {
    public static Finalization f;

    // 当对象被GC时，会在GC前调用finalize()方法执行
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("重新与GcRoots建立连接...");
        // 当执finalize方法时，重新将当前实例复活，让对象变成可触及性的
        f = this;  
    }

    @Override
    public String toString() {
        return "我是Finalization实例对象....";
    }

    public static void main(String[] args) throws
            InterruptedException {
        f = new Finalization();
        f = null;   // 置空引用(处于可复活状态)
        
        System.out.println("第一次GC被强制触发了....");
        System.gc(); // 手动触发GC
        
        Thread.sleep(1000);
        if (f == null) {
            System.out.println("f == null");
        } else {
            System.out.println("f ！= null");
        }
        
        f = null;    // 再次置空(前面复活过一次，这次为不可复活)
        System.out.println("第二次GC被强制触发了....");
        System.gc(); // 手动触发GC
        
        Thread.sleep(1000);
        if (f == null) {
            System.out.println("f == null");
        } else {
            System.out.println("f ！= null");
        }
    }
}

结果：
第一次GC被强制触发了....
重新与GcRoots建立连接...
f ！= null
第二次GC被强制触发了....
f == null

```







### 垃圾回收算法

#### 1. 标记-清除算法：

 将不需要回收的对象标记出来，在标记完成之后统一回收掉所有被标记的对象。（最基础的算法）

所带的问题为：1：效率不高，标记和清楚的效率都不高。2. 而且容易产生大量不连续的内存碎片。

#### 2. 标记-复制算法：

为了解决标记-清楚算法的效率和内存碎片问题，复制收集算法出现了， 将内存分为大小相同的两块，每次使用其中一块，当使用空间完了，就将存活对象复制到另外一块，然后清除其中的一块。

问题：1. 可用内存缩小为原来的一半。2. 不适合老年代，如果存活对象数量比较大，复制性能会变得比较差。

#### 3. 标记-整理算法

在标记阶段将不需要的标记出来，让所有存活的对象移动压缩到内存的一端，然后直接清理掉端边界以外的内存。适合于老年代这种垃圾回收频率不是很高的场景。

#### 4. 分代收集算法

根据对象存活周期的不同将内存分为几块。一般将Java堆分为新生代和老生代，根据各个年代的特点选择合适的垃圾收集算法。

新生代：一般使用复制算法

老年代：一般采用标记-整理算法或者标记-清除算法





### 垃圾收集器

#### 1. Serial收集器

单线程的交互，简单高效。多用于client模式下的虚拟机

#### 2. ParNew收集器

#### 3. G1收集器

默认就是g1收集器，优先选择回收价值最大的Region

- young GC

  回收 Eden 区和 Survivor 区中不用的对象。会导致 STW。G1 垃圾回收期会尽可能地保证暂停时间

- mixed GC

- full GC 单线程清理垃圾



### Java中的四种引用

Java中引用四种引用的目的是让程序自己决定对象的生命周期，JVM通过垃圾回收期对这四种引用做不同的处理，来实现对象生命周期的改变。

#### 1. 强引用

Java中最常见的就是强引用，把一个对象赋给一个引用变量，这个引用变量就是一个强引用。

当一个对象被强引用变量引用时，它处于可达状态，是**不可能被垃圾回收器回收**的，即使该对象永远不会被用到也不会被回收。

当内存不足，JVM 开始垃圾回收，对于强引用的对象，就算是**出现了 OOM 也不会对该对象进行回收**，打死都不收。因此强引用有时也是造成 Java 内存泄露的原因之一。

对于一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显示地将相应（强）引用赋值为 null，一般认为就是可以被垃圾收集器回收。（具体回收时机还要要看垃圾收集策略）。

```java
Object o1 = new Object();
Object o2 = o1;
o1 = null;
System.gc();
System.out.println(o1); //null
System.out.println(o2); //java.lang.Object@2503dbd3

demo 中尽管 o1已经被回收，但是 o2 强引用 o1原指向的位置，一直存在，所以不会被GC回收
```



#### 2. 软引用

软引用是一种相对强引用弱化了一些的引用，需要用 java.lang.ref.SoftReference 类来实现，可以让对象豁免一些垃圾收集。

软引用用来描述一些还有用，但并非必需的对象。对于软引用关联着的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中并进行第二次回收。如果这次回收还是没有足够的内存，才会抛出内存溢出异常。

对于只有软引用的对象来说：当系统内存充足时它不会被回收，当系统内存不足时它才会被回收。

```java
import java.lang.ref.SoftReference;

public class test {
    private static void softRefMemoryEnough() {
        Object o1 = new Object();
        SoftReference<Object> s1 = new SoftReference<Object>(o1);
        System.out.println(o1);
        System.out.println(s1.get());
        o1 = null;
        System.gc();
        System.out.println(o1);
        System.out.println(s1.get());
    }

    /**
     * JVM配置`-Xms5m -Xmx5m` ，然后故意new一个一个大对象，使内存不足产生 OOM，看软引用回收情况
     */
    private static void softRefMemoryNotEnough() {
        Object o1 = new Object();
        SoftReference<Object> s1 = new SoftReference<Object>(o1);
        System.out.println(o1);
        System.out.println(s1.get());
        o1 = null;
        byte[] bytes = new byte[10 * 1024 * 1024 * 1024 * 1024 * 1024];
        System.out.println(o1);
        System.out.println(s1.get());
    }

    public static void main(String[] args) {
        softRefMemoryEnough();
        System.out.println("------内存不够用的情况------");
        softRefMemoryNotEnough();
    }
}
```



软引用通常用在**对内存敏感的程序**中，比如高速缓存就有用到软引用，内存够用的时候就保留，不够用就回收。

#### 3. 弱引用 (Weak reference)

弱引用也是用来描述非必需对象的，但是它的强度比软引用更弱一些，**被弱引用关联的对象只能生存到下一次垃圾收集发生之前**。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。

```java
Object o1 = new Object();
WeakReference<Object> w1 = new WeakReference<Object>(o1);
System.out.println(o1);
System.out.println(w1.get());
o1 = null;
System.gc();
System.out.println(o1);
System.out.println(w1.get());
```



#### 4. 虚引用

虚引用，顾名思义，就是形同虚设，与其他几种引用都不太一样，一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。

如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收，它不能单独使用也不能通过它访问对象，虚引用必须和引用队列（RefenenceQueue）联合使用。

虚引用的主要作用是跟踪对象垃圾回收的状态。仅仅是提供了一种确保对象被 finalize 以后，做某些事情的机制。

PhantomReference 的 get 方法总是返回 null，因此无法访问对应的引用对象。其意义在于说明一个对象已经进入 finalization 阶段，可以被 GC 回收，用来实现比 finalization 机制更灵活的回收操作。

换句话说，**设置虚引用的唯一目的，就是在这个对象被回收器回收的时候收到一个系统通知或者后续添加进一步的处理**。



### 类的加载过程

#### 类的生命周期

加载——连接——初始化——使用——卸载。连接过程又包括（验证、连接、解析）

1. 加载
2. 验证
3. 准备
4. 解析
5. 初始化
6. 使用
7. 卸载

#### 加载阶段

1. **类加载器**根据类的全限定名通过不同的渠道以二进制流的方式获取字节码信息
2. 加载完类之后，虚拟机会将字节码中的信息保存到方法区内。
3. 生成一个instanceKlass对象，保存类的所有信息，里边还包含特定功能比如多态信息
4. Java虚拟机会再堆中生成一份与方法去中数据类似的java.lang.Class对象。作用是在Java代码中去获取**类的信息**以及存储**静态字段**的数据。

![image-20240729163639624](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240729163639624.png)



#### 连接阶段

1. 验证：验证内容是否满足《Java虚拟机规范》

2. 准备：给静态变量赋初值

   如果静态变量用final来修饰，且为基本数据类型，准备阶段直接会将代码的值进行赋值

3. 解析： 将常量池中的符号引用替换成指向内存的直接引用

#### 初始化阶段

- 初始化阶段会执行静态代码块中的代码，并为静态变量赋值
- 初始化阶段会执行字节码文件中clinit部分的字节码指令。

类的初始化：

- 访问一个类的静态变量或者静态方法
- 调用Class.forName(String className)
- new 一个该类的对象时。
- 执行main方法的当前类。

构造代码块比构造方法优先

### 类加载器

类加载器的主要作用就是加载Java类的字节码(.class文件)到jvm中（在内存中生成一个代表该类的Class对象）。 

- SPI机制 (Service Provider Interface)

类加载器分为两类，一类是Java代码实现的，jdk默认提供了多种处理不同渠道的类加载器。所有Java中实现的类加载器都需要继承ClassLoader这个抽象类

另一类是Java虚拟机源码提供的，用C++来实现



### 双亲委派机制

当一个类加载器接收到加载类的任务时，会**自底向上查找是否加载过，再由顶向下进行加载**。

classloader实例会在试图亲自查找类或资源之前，将搜索类或资源的任务委托给其父类加载器。

1. 通过双亲委派机制避免恶意代码替换 JDK 的核心类库，保证类加载的安全性
2. 避免重复加载，避免一个类被多次加载

![image-20240730175906464](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240730175906464.png)



### 在Java中如何使用代码的方式去主动加载一个类呢

- 使用 Class.forName方法，使用当前类的类加载器去加载指定的类
- 获取到类加载器，通过类加载器的 loadClass 方法指定某个类加载器加载。

```java
public class test {

    public static void main(String[] args) throws InterruptedException, ClassNotFoundException {
        ClassLoader classLoader = test.class.getClassLoader();

        System.out.println(classLoader.loadClass("java.lang.String").getClassLoader());
    }
}
```



### 打破双亲委派机制的三种方法

#### 自定义类加载器

自定义类加载器并且重写 loadClass 方法，就可以将双亲委派机制的代码去除

Tomcat通过这种方式实现应用之间类隔离，《面试篇》中分享他的做法

```java
//重写一个classLoader，然后重写loadclass
public class test extends ClassLoader{

    private String basePath;
    private final static String FILE_EXT = ".class";

    public void setBasePath(String basePath) {
        this.basePath = basePath;
    }

    private byte[] loadClassData(String name) {
        return null;
    }

    public Class<?> loadClass(String name) throws ClassNotFoundException {
        byte[] data = loadClassData(name);
        return defineClass(name, data, 0, data.length);
    }


    public static void main(String[] args) throws InterruptedException, ClassNotFoundException, InstantiationException, IllegalAccessException {
        test t = new test();
        t.setBasePath("D:\\lib\\");

        Class<?> class1 = t.loadClass("java.lang.String");
        class1.newInstance();
    }
}
```

![image-20240730221239502](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240730221239502.png)

#### 线程上下文类加载器

利用上下文类加载器加载类，比如 JDBC 和 JDNI

#### Osgi框架的类加载器





# 内存调优

