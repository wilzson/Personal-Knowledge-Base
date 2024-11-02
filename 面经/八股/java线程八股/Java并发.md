# 线程创建

## 继承Thread类

```java
Thread thread1 = new Thread("t1") {
    // 匿名内部类
    public void run() {
        
    }
};

thread1.start();

```

## 实现Runnable接口

```java
Runnable r = () -> {
    System.out.println("1");
};

Thread thread1 = new Thread(r,"t1") ;
Thread thread2 = new Thread(()->{System.out.println("1");}, "t2");
thread1.start();
```

## 实现Callable接口

callable带有返回值，所以可以在线程执行完了将数据传输给其他线程。

```java
class MyCallable implements Callable<String> {
    @Override//重写线程任务类方法
    public String call() throws Exception {
        return Thread.currentThread().getName() + "->" + "Hello World";
    }
}

Callable call = new MyCallable();
FutureTask<String> task = new FutureTask<>(call);
//FutureTask<Integer> task = new FutureTask<>(new Callable<Integer>() {
   //public Integer call() throw Exception {
       //return null;
   //} 
//});
Thread t = new Thread(task);
t.start();
try {
    String s = task.get(); // 获取call方法返回的结果（正常/异常结果）
    System.out.println(s);
}  catch (Exception e) {
    e.printStackTrace();
}
```

```java
public class test {

    static int func() throws InterruptedException {
        Thread.sleep(3000);
        System.out.println(1);
        return 1;
    }
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        long startTime = System.currentTimeMillis();
        // 1. 先定义Callable
        Callable<String> callable = new Callable<String>() {
            @Override
            public String call() throws Exception {
                Thread.sleep(1000);
                return "Hello";
            }
        };
        // 2. 用Future对象包装
        FutureTask<String> f1 = new FutureTask<>(callable);
        // 3. 用线程来运行future对象
        new Thread(f1).start();

        Callable<String> callable1 = new Callable<String>() {
            @Override
            public String call() throws Exception {
                Thread.sleep(2000);
                return "world";
            }
        };
        FutureTask<String> f2 = new FutureTask<>(callable1);
        new Thread(f2).start();

        System.out.println(f1.get() + f2.get());
        long endTime = System.currentTimeMillis();
        System.out.println((endTime - startTime) / 1000 + "秒");
    }
}
```



## 常见方法

start( )，run( )

- 直接调用run方法，最终还是main( )来执行，相当于执行普通方法，没有实现多线程部分的操作。
- thread类中的start方法调用的是native方法，也就是C++方法，由操作系统来new线程

---



sleep：

- 调用 sleep 会让当前线程从 `Running` 进入 `Timed Waiting` 状态（阻塞）
- sleep() 方法的过程中，**线程不会释放对象锁**
- 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException
- 睡眠结束后的线程未必会立刻得到执行，需要抢占 CPU
- 建议用 TimeUnit 的 sleep 代替 Thread 的 sleep 来获得更好的可读性
- 在没有利用cpu计算时，不要让while(true)空转浪费cpu，这时可以用sleep 或者 yield 来让出 cpu 使用权。

yield：

- 调用 yield 会让提示线程调度器让出当前线程对 CPU 的使用
- 具体的实现依赖于操作系统的任务调度器
- **会放弃 CPU 资源，锁资源不会释放**
- 调用yield让出CPU资源后，还是会和其他线程去抢占资源，所以很有可能下一轮还会被CPU选中。

---



join( )

由于多线程机制下，各个线程都是并行计算的，所以主线程往往跑完了，子线程还没跑完。为了主线程能获得子线程的数据。join( )函数会进行阻塞，等待子线程运行完，再继续运行主线程。

join( )类似线程插队，谁调用join( )方法，谁就相当于插入到最前面执行，其他线程都要等待该线程执行完。

```java
Thread t = new Thread(task);
t.start(); // 启动子线程
t.join(); // 启动完之后，运行join()。主线程会阻塞直到子线程运行结束
```

```java
Thread t = new Thread(task);
Thread t2 = new Thread(task);
t.start();
t2.start();
t.join();
t2.join();

// 线程并行执行，同步返回
```



join ( time ) // 表示 线程最多等待time个时间结束。

### 线程打断

打断的线程会发生上下文切换，操作系统会保存线程信息，抢占到 CPU 后会从中断的地方接着运行（打断不是停止）

- sleep、wait、join 方法都会让线程进入阻塞状态，打断线程**会清空打断状态**（false）

  ```java
  public static void main(String[] args) throws InterruptedException {
      Thread t1 = new Thread(()->{
          try {
              Thread.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }, "t1");
      t1.start();
      Thread.sleep(500);
      t1.interrupt();
      System.out.println(" 打断状态: {}" + t1.isInterrupted());// 打断状态: {}false
  }
  ```

## 守护线程

用户线程：平常创建的普通线程

守护线程：服务于用户线程，只要其他非守护线程运行结束了，即使守护线程代码没有执行完，也会强制结束。守护进程是脱离于终端并且在后台运行的进程，脱离终端是为了避免在执行的过程中的信息在终端上显示。

# 线程运行原理

## 栈帧

不同线程占用不同区间的栈帧，线程之间相互独立。

jvm 内部包含有堆、虚拟机栈、方法区。每一个线程启动后，虚拟机都会为其分配一块内存。

- 每个栈由多个栈帧（Frame）组成，对应着每次方法调用时所占用的内存。
- 每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法。
- 虚拟机栈中包含多个栈帧，栈帧中有局部变量表，操作数栈，方法返回地址，动态链接。

![image-20240723153805031](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240723153805031.png)



# 共享模型

## 问题分析

两个线程并发会出现不同的数值。原因为Java中对静态变量的自增、自减并不是原子操作。

```java


public class test {
    static int counter = 0;

    public static void main(String[] args) throws InterruptedException {

        Thread t1 = new Thread(()->{
            for(int i = 0; i < 5000; i++) {
                counter++;
            }
        }, "t1");

        Thread t2 = new Thread(()->{
            for(int i = 0; i < 5000; i++) {
                counter--;
            }
        }, "t2");

        t1.start();
        t2.start();

        t2.join();
        t1.join();

        System.out.println(counter);
    }
}
```



```java
getstatic     i // 获取静态变量i的值
iconst_1      1 // 准备常量1
iadd            // 自增
putstatic     i // 将修改后的值存入静态变量i
```

由于多线程的指令交错问题，会导致出现线程并发的问题

问题出现在多个线程对共享资源读写操作时发生指令交错，就会出现线程并发问题。

## 解决方案

### 阻塞式互斥（Synchronized, lock)

#### synchonized（对象锁）

```java


public class test {
    static Integer counter = 0;
    static Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {

        Thread t1 = new Thread(()->{
            for(int i = 0; i < 5000; i++) {
                synchronized (counter.getClass()){
                    counter++;
                }

            }
        }, "t1");

        Thread t2 = new Thread(()->{
            for(int i = 0; i < 5000; i++) {
                synchronized (counter.getClass()) {
                    counter--;
                }
            }
        }, "t2");

        t1.start();
        t2.start();

        t2.join();
        t1.join();

        System.out.println(counter);
    }
}

public Test{
    // synchonized 加在方法上
	public synchonized void test() {
    	
	}
	// 等于
	public void test() {
    	synchonized (this) {
        	// 锁住的是实例对象
    	}
	}
    
    // synchonized 加在静态方法上
    public static synchonized void test() {
        
    }
    // 约等于
    public void test() {
        synchonized(Test.class) { // 锁住的是类对象
            
        }
    }
}

```

![image-20240725095249059](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240725095249059.png)

synchonized实际是用**对象锁**保证了**临界区内代码的原子性**。

```java
// 线程八锁


```



### 非阻塞式的（原子变量, voliate）





## 变量的线程安全分析

局部变量的是基本类型则不会出现线程安全问题。

如果局部变量引用的是对象的话，则会出现线程安全问题。

![image-20240725110829813](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240725110829813.png)

```java
import java.lang.reflect.Array;
import java.util.ArrayList;
// 出现线程安全问题
class ThreadUnsafe {
    ArrayList<String> list = new ArrayList<>();

    public void method1() {
        for(int i = 0; i < 200; i++) {
            method2();
            method3();
        }
    }
    private void method2() {
        list.add("1");
    }
    private void method3() {
        list.remove(0);
    }
}
public class test {

    public static void main(String[] args) throws InterruptedException {
        ThreadUnsafe test = new ThreadUnsafe();
        for (int i = 0; i < 2; i++) {
            new Thread(()->{
                test.method1();
            }).start();
        }
        
    }
}
```

![image-20240725111700433](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240725111700433.png)

```java
import java.lang.reflect.Array;
import java.util.ArrayList;
import java.util.List;
// 改成局部变量的话，不会出现线程安全问题
class ThreadUnsafe {
    public void method1() {
        ArrayList<String> list = new ArrayList<>();
        for(int i = 0; i < 200; i++) {
            method2(list);
            method3(list);
        }
    }
    private void method2(List<String> list) {
        list.add("1");
    }
    private void method3(List<String> list) {
        list.remove(0);
    }
}
public class test {

    public static void main(String[] args) throws InterruptedException {
        ThreadUnsafe test = new ThreadUnsafe();
        for (int i = 0; i < 2; i++) {
            new Thread(()->{
                test.method1();
            }).start();
        }

    }
}
```

### 引发线程安全的三要素

- 多线程并发
- 有共享资源
- 对共享资源有操作，如写操作

## Monitor（锁）

synchronized的底层实现，monitor被翻译为监视器或者管程

当线程获得synchronized对应的monitor时，后续的线程会在entrylist中阻塞。当线程结束后，操作系统会唤醒entrylist中的线程，然后竞争获取锁。

![image-20240725172657975](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240725172657975.png)



# synchronized原理进阶

## 1. 轻量级锁

轻量级锁的使用场景：（类似于乐观锁）

如果一个对象虽然有多线程访问，但多线程访问的时间时错开的（也就是没有竞争）。那么可以使用轻量级锁来优化。

对象 = 对象头（分成两部分：hashcode age bias 01 + Klass world (类型指针)） + 对象体（Object body）

## 2. 锁膨胀

当轻量级锁遇到线程并发的时候，synchronized底层会转换为重量级锁。

## 3. 自旋优化

当重量级锁竞争的时候，还可以使用自旋来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出同步块，释放锁），这时当前线程就可避免阻塞（阻塞会产生上下文切换，由用户态转内核态）。

## 4. 偏向锁

轻量级锁在没有竞争时，每次重入仍然需要执行CAS操作.

只有第一次使用CAS将线程 id 设置到对象的 mark word 头，之后发现这个线程 id 是自己的就表示没有竞争，不用重新CAS。只要不发生竞争，这个对象就归线程所有。

## wait notify

### 原理

当 Owner 线程发现条件不满足，调用 wait 方法，进入 waitset 变为 waiting 状态。

- blocked 线程会在 Owner 线程释放锁时唤醒
- waiting 线程会在 Owner 线程调用 notify 或 notifyall 时唤醒，但需要重新进入 entrylist 重新竞争。

![image-20240725182904529](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240725182904529.png)

- obj.wait( )
- obj.notify( )
- obj.notifyall( )

```java
synchronized (lock) {
    while(条件不成立) {
    	lock.wait();
	}
    // 干活
}
// 另一个线程
synchronized (lock) {
    lock.notifyAll();
    // 干活
}
```



# Park & unpark

```java
// 暂停当前线程
LockSupport.park();

// 恢复某个线程的运行
LockSupoort.unpark(暂停线程对象);
```

```java
import java.util.concurrent.locks.LockSupport;

public class test {

    // 应用于指定式的wait和notify
    public static void main(String[] args) throws InterruptedException {

        Thread t1 = new Thread(()->{
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println("park...");
            LockSupport.park();
            System.out.println("resume...");
        });
        t1.start();

        Thread.sleep(2000);
        System.out.println("unpark...");
        LockSupport.unpark(t1);
    }
}

```

- park & unpark 可以先unpark，而 wait & notify 不能先 notify



# 多种锁

活锁

多个线程互相改变对方线程结束的条件，导致线程无法结束 （顺序加锁是解决死锁的一个方法）

饥饿

线程优先级太低，导致一直无法获取资源。（ReentranLock 可解决）



# ReentranLock（可重入锁）

Reentranlock是在 juc 包中一个重要的工具类（synchronized 是 Java 的一个关键字）

- 可中断
- 可以设置超时时间
- 可以设置为公平锁
- 支持多个条件变量

Reentranlock 和 synchronized 一样都是可重入锁。

```java
class Test {
    ReentrantLock lock = new ReentrantLock();
    public void Test1() {
        lock.lock();
        try {
            System.out.println("上锁执行");
        } finally {
            lock.unlock();
        }
    }
}
```



## 可打断

在 synchronized 中，等待序列中是不可被打断的，reentranlock 在等待的时候是可以被打断的

```java
class Test {
    ReentrantLock lock = new ReentrantLock();
    public void Test1() throws InterruptedException {
        // 如果没有竞争，那么此方法就会获取lock对象锁
        // 如果有竞争就进入阻塞队列，可以被其他线程用 interrupt 方法打断
        lock.lockInterruptibly();
        try {
            System.out.println("上锁执行");
        } finally {
            lock.unlock();
        }
    }
}
```



## 锁超时，主动停止等待获取锁

```java
class Test {
    ReentrantLock lock = new ReentrantLock();
    public void Test1() throws InterruptedException {
        // 尝试获取锁，获取到锁则返回 True
        if (!lock.tryLock(1, TimeUnit.SECONDS)) {
            System.out.println("获取不到锁");
            return;
        }
        try {
            System.out.println("获得到锁");
        } finally {
            lock.unlock();
        }
    }
}
```



  ## Reentranlock中条件变量



### 条件变量await( )和signal( )或signalAll( )；

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class test {

    static ReentrantLock lock = new ReentrantLock();
    static Condition a = lock.newCondition();
    static Condition b = lock.newCondition();
    static Condition c = lock.newCondition();
    public static void main(String[] args) throws InterruptedException {

        new Thread(()->{
            while (true) {
                lock.lock();
                try {
                    try {
                        a.await();
                        System.out.print("A");
                        b.signal();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                } finally {
                    lock.unlock();
                }
            }
        }, "t1").start();

        new Thread(()->{
            while (true) {
                lock.lock();
                try {
                    try {
                        b.await();
                        System.out.print("B");
                        c.signal();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                } finally {
                    lock.unlock();

                }
            }
        }, "t2").start();

        new Thread(()->{
            while (true) {
                lock.lock();
                try {
                    try {
                        c.await();
                        System.out.print("C");
                        a.signal();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                } finally {
                    lock.unlock();

                }
            }
        }, "t3").start();

        Thread.sleep(1000);
        lock.lock();
        try {
            System.out.println("开始...");
            a.signal();
        } finally {
            lock.unlock();
        }
    }
}
```

### 使用lock support的 park 和 unpark 方法来使用

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.LockSupport;
import java.util.concurrent.locks.ReentrantLock;

class TTest {
    void print(String str, Thread thread) {
        for (int i = 0; i < 5; i++) {
            LockSupport.park();
            System.out.println(str);
            LockSupport.unpark(thread);
        }
    }
}

public class test {

    static Thread t1;
    static Thread t2;
    static Thread t3;
    static TTest t = new TTest();

    public static void main(String[] args) throws InterruptedException {

        t1 = new Thread(() -> {
            t.print("a", t2);
        });
        t2 = new Thread(() -> {
            t.print("b", t3);
        });
        t3 = new Thread(() -> {
            t.print("c", t1);
        });
        t1.start();
        t2.start();
        t3.start();

        LockSupport.unpark(t1);
    }
}
```



# Java 内存模型（ JMM ）

JMM 定义了主存（所有线程都共享的数据）、工作内存（各自线程私有的部分）

JMM 体现在以下几个方面

- 原子性 - 保证指令不受线程上下文切换的影响 （synchronized）
- 可见性 - 保证指令不受 cpu 缓存的影响
- 有序性 - 保证多条指令不会受 cpu 指令并行优化的影响

![image-20240726152933974](C:\Users\wilzsn\AppData\Roaming\Typora\typora-user-images\image-20240726152933974.png)

## 可见性解决方案 （volatile）（CPU缓存优化引起）

volatile 可以用来修饰成员变量和静态成员变量，它可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取他的值，线程操作 volatile 变量都是直接操作主存。

修改后的数据必须立即写回主存；禁止指令重排序

```java
public class test {

    volatile static boolean run = true;

    public static void main(String[] args) throws InterruptedException {

        new Thread(()->{
            while(true) {
                if (!run) {
                    break;
                }
            }
        }).start();

        Thread.sleep(1000);
        System.out.println("停止 t");

        run = false;
    }
}
```



volatile 适合一写 多读的场景 。因此 volatile 只能保证可见性，但是不能保证原子性。（原子性还是需要synchronized）

## 有序性（jvm指令重排序引起）

JVM 会在不影响正确性的前提下，可以调整语句的执行顺序

```java
static int i;
static int j;

// 在某个线程内执行如下操作
i = ...;
j = ...;

// i，j的执行顺序不会影响代码的结果，所以 jvm 为提高效率，会实现指令重排的操作
// 但是在多线程情况下，会影响正确性。
```



```java
public class test {

    volatile static boolean run = true;

    public static void main(String[] args) throws InterruptedException {

        new Thread(()->{
            while(true) {
                if (!run) {
                    break;
                }
            }
        }).start();
        Thread.sleep(1000);
        System.out.println("停止 t");
        run = false;
    }
}
```



## 指令重排

volatile 关键字放在最后一个变量上，通过写屏障可以让变量以上的指令禁用重排序

```java
public class test {

    int num = 0;
    volatile boolean ready = false;
    
    public void actor1(Result r) {
        if (ready) {
            r.r1 = num + num;
        } else {
            r.r1 = 1;
        }
    }
    
    public void actor2(Result r) {
        num = 2;
        ready = false;
    }

    public static void main(String[] args) throws InterruptedException {

        
    }
}
```

### volatile 原理

volatile 的底层实现原理是内存屏障

- 对 volatile 变量的写指令后会加入写屏障（保证在该屏障之前，对共享变量的改动，都同步到主存当中）
- 对 volatile 变量的读指令后会加入读屏障（保证在读屏障之后，对共享变量的读取，加载的都是主存中最新数据）



## double-checked locking 双重检测

```java
// 当前代码在多线程情况还是有有序性问题
final class Singleton {
    private Singleton() { }
    // private static Singleton INSTANCE = null;
    private static volatile Singleton INSTANCE = null; // 加一个volatile 解决指令重排序
    public static Singleton getINSTANCE() {
        if (INSTANCE == null) {
            // 首次访问会同步，而之后的使用没有 synchronized
            synchronized (Singleton.class) {
                if (INSTANCE == null) {
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```

由于CPU发生指令重排优化，容易导致线程t2获得一个半成品的对象。

# 乐观锁

实现无锁并发

```java
class AccountCas {
    private AtomicInteger balance;

    public AccountCas(Integer balance) {
        this.balance = new AtomicInteger(balance);
    }

    public Integer getBalance() {
        return balance.get();
    }

    public void withdraw(Integer amount) {
        while(true) {
            // 获取余额的最新值
            int prev = balance.get();
            // 要修改的余额
            int next = prev - amount;
            // 真正修改
            if (balance.compareAndSet(prev, next)) {
                break;
            }
        }
    }
}
```



CAS必须借助volatile才能读取到共享变量的最新之来实现compare and swap的效果

## 原子类

```java
public class test {


    public static void main(String[] args) throws InterruptedException {

        AtomicInteger integer = new AtomicInteger(0);

        // 获取再自增
        System.out.println(integer.getAndIncrement());
        // 自增并获取值
        System.out.println(integer.incrementAndGet());

        System.out.println(integer.getAndAdd(5));
        // 参数为一个函数式接口
        System.out.println(integer.updateAndGet(x -> x * 10));

        // 原子引用
        AtomicReference<BigDecimal> balance;
        
    }
}
```



## 解决ABA问题（通过加版本号）

AtomicStampedReference  加了版本号的原子引用

```java
public class test {

    static AtomicStampedReference<String> reference = new AtomicStampedReference<>("A", 0);

    public static void main(String[] args) throws InterruptedException {

        // 获取共享变量的值
        String str = reference.getReference();
        // 获取版本号
        int stamp = reference.getStamp();
        
        while(true) {
            // 需要四个参数，值和版本号的比较
            if (reference.compareAndSet(str, str + "a", stamp, stamp + 1)) {
                break;
            }
        }
    }
}
```



```java
public class test {

    static AtomicMarkableReference<String> reference = new AtomicMarkableReference<>("A", true);

    public static void main(String[] args) throws InterruptedException {

        // 获取共享变量的值
        String str = reference.getReference();

        while(true) {
            // 需要四个参数，值和版本号的比较
            if (reference.compareAndSet(str, str + "a", true, false)) {
                break;
            }
        }
    }
}
```



## 原子数组、原子更新器、原子累加器

```java
// 原子数组对共享数组做修改
// 原子更新器对共享对象的某个成员变量做修改
// 原子累加器 专门做累加的
```



# unsafe

unsafe对象提供了非常底层，操作内存、线程的方法，unsafe对象不能直接调用，只能通过反射获得



# final

final 变量的赋值会在putfield指令之后加入写屏障，保证在其他线程读到它的值时不会出现 0 的情况。



# 线程池

```java
import java.util.ArrayDeque;
import java.util.HashSet;
import java.util.Queue;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

@FunctionalInterface // 拒绝策略
interface RejectPolicy<T> {
    void reject(BlockingQueue<T> queue, T task);
}


class ThreadPool {
    private BlockingQueue<Runnable> taskQueue;

    // 线程集合
    private HashSet<Worker> workers = new HashSet<Worker>();

    // 核心线程数
    private int coreSize;

    // 最大线程数
    private int maxCoreSize;

    // 获取任务的超时时间
    private long timeout;

    private TimeUnit unit;

    private int queueCapacity;

    private int threadSize;

    // 拒绝策略
    private RejectPolicy<Runnable> rejectPolicy;

    public ThreadPool(int coreSize, int maxCoreSize, long timeout, TimeUnit unit, int queueCapacity, RejectPolicy<Runnable> rejectPolicy) {
        this.coreSize = coreSize;
        this.maxCoreSize = maxCoreSize;
        this.timeout = timeout;
        this.unit = unit;
        this.queueCapacity = queueCapacity;
        this.threadSize = 0;
        this.taskQueue = new BlockingQueue<>(queueCapacity);
        this.rejectPolicy = rejectPolicy;
    }

    // 执行任务
    public void execute(Runnable task) {
        synchronized (workers) {
            // 当任务数没有超过coreSize时， 直接交给 worker 对象执行
            if (workers.size() < coreSize) {
                Worker worker = new Worker(task);
                workers.add(worker);
                worker.start();
            }
            // 当任务数超过 coreSize 时，加入任务队列暂存
            else {
                // 拒绝策略
                taskQueue.tryPut(rejectPolicy, task);
                // 当任务队列满了时候，则新增线程来处理任务
//                if (taskQueue.size() < queueCapacity) {
//                    taskQueue.put(task);
//                } else {

            }

        }
    }

    class Worker extends Thread {
        private Runnable task;

        public Worker(Runnable task) {
            this.task = task;
        }

        @Override
        public void run() {
            // 执行任务
            // 1) 当 task 不为空时，执行任务
            // 2) 当 task 为空时，从任务队列中获取任务并执行
            while (task != null || (task = taskQueue.poll(1, TimeUnit.SECONDS)) != null) {
                try {
                    System.out.println("执行任务");
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    task = null;
                }
            }
            synchronized (workers) {
                workers.remove(this);
            }
        }
    }
}


class BlockingQueue<T> {
    // 1. 任务队列
    private Queue<T> queue = new ArrayDeque<>();

    // 2. 锁
    private ReentrantLock lock = new ReentrantLock();

    // 3. 生产者条件变量
    private Condition producer = lock.newCondition();

    // 4. 消费者条件变量
    private Condition consumer = lock.newCondition();

    // 5. 容量
    private int capacity;

    public BlockingQueue(int capacity) {
        this.capacity = capacity;
    }

    // 带超时的阻塞获取
    public T poll(long timeout, TimeUnit unit) {
        lock.lock();
        try {
            // 将 timeout 统一转换为纳秒
            long nanos = unit.toNanos(timeout);
            while (queue.isEmpty()) {
                try {
                    if (nanos <= 0) {
                        return null;
                    }
                    // 返回的是剩余时间
                    nanos = consumer.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            T t = queue.poll();
            producer.signalAll();
            return t;
        } finally {
            lock.unlock();
        }
    }

    // 阻塞获取
    public T take() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                try {
                    consumer.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            T t = queue.poll();
            producer.signalAll();
            return t;
        } finally {
            lock.unlock();
        }
    }

    // 阻塞添加
    public void put(T element) {
        lock.lock();
        try {
            while (queue.size() >= capacity) {
                try {
                    producer.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            queue.add(element);
            consumer.signalAll();
        } finally {
            lock.unlock();
        }
    }

    // 带超时时间阻塞添加
    public boolean offer(T element, long timeout, TimeUnit unit) {
        lock.lock();
        try {
            // 将 timeout 统一转换为纳秒
            long nanos = unit.toNanos(timeout);
            while (queue.size() == capacity) {
                try {
                    if (nanos <= 0) {
                        return false;
                    }
                    // 返回的是剩余时间
                    nanos = consumer.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            boolean t = queue.add(element);
            consumer.signalAll();
            return t;
        } finally {
            lock.unlock();
        }
    }

    // 获取大小
    public int size() {
        lock.lock();
        try {
            return queue.size();
        } finally {
            lock.unlock();
        }
    }

    public void tryPut(RejectPolicy<T> rejectPolicy, T task) {
        lock.lock();
        try {
            // 判断队列是否满
            if (queue.size() == capacity) {
                rejectPolicy.reject(this, task);
            } else {
                queue.add(task);
                consumer.signalAll();
            }
        } finally {
            lock.unlock();
        }
    }
}

public class test {

    public static void main(String[] args) throws InterruptedException {

        ThreadPool threadPool = new ThreadPool(2, 5, 1000, TimeUnit.SECONDS, 10,
                (queue, task) -> {
                    // 死等
//                    queue.put(task);
                    // 带超时等待
//                    queue.offer(task, 500, TimeUnit.MILLISECONDS);
                    // 让调用者放弃任务执行
//                    System.out.println("放弃");
                    // 让调用者自己抛出异常, 后续流程不会继续进行
//                    throw new RuntimeException("任务执行失败 " + task);
                    // 让调用者自己执行任务
                    task.run();
                });
        for (int i = 0; i < 5; i++) {
            threadPool.execute(() -> {
                System.out.println("i");
            });
        }

    }
}


```



# Fork and join 线程池

适合任务能够拆分的 cpu 密集型运算。适合分治的思想

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;

public class test {

    public static void main(String[] args) {
		// forkand join 线程池创建
        ForkJoinPool pool = new ForkJoinPool();
        // 调用 使用invoke函数
        System.out.println(pool.invoke(new MyTask(5)));

    }
}

// recursive表示递归，recursiveTask是带返回值的数据，recursiveAction表示不带返回值数据
class MyTask extends RecursiveTask<Integer> { // 1) 任务必须继承recursiveTask<>,recursiveAction

    private int n;
    public MyTask(int n) {
        this.n = n;
    }

    @Override
    protected Integer compute() {
        // 终止条件
        if (n == 1) {
            return 1;
        }
        MyTask t1 = new MyTask(n-1);
        t1.fork(); // 让一个线程去执行任务 拆分线程去执行
        int result = n + t1.join(); // 获取任务结果，获得线程结果
        return result;
    }
}

```



# JUC 并发工具

## AQS 原理

AbstractQueueSynchronizer，阻塞式锁和相关的同步器工具的框架

## ReentranLock 原理

## 读写锁 ReentranReadWriteLock

当读操作远远高于写操作时，这时候用读写锁让读-读可以并发，提高性能。

```java
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class test {
    // 读写锁， 读读并发，读写互斥， 写写互斥
    public static void main(String[] args) {

        DataContainer container = new DataContainer();
        new Thread(()->{
            container.read();
        }, "t1").start();

        new Thread(()->{
            Object a = new Object();
            container.write(a);
        }, "t2").start();

    }
}

class DataContainer{
    private Object data;
    private ReentrantReadWriteLock rw = new ReentrantReadWriteLock();
    // 获取读锁
    private ReentrantReadWriteLock.ReadLock readLock = rw.readLock();
    // 获取写锁
    private ReentrantReadWriteLock.WriteLock writeLock = rw.writeLock();

    public void write(Object data) {
        writeLock.lock();
        try {
            System.out.println("写入");
            this.data =data;
        } finally {
            writeLock.unlock();
        }
    }
    public Object read() {
        readLock.lock();
        try {
            System.out.println("读取");
            return data;
        } finally {
            readLock.unlock();
        }

    }
}
```

### 读写锁的注意事项

读锁不支持条件变量， 写锁才支持条件变量

读锁不能升级写锁，以获取读锁后，不能继续获取写锁。 写锁可以获取读锁

## 缓存更新策略

先更新数据库，再更新缓存

## StampedLock 特殊的读写锁

进一步优化读性能，它的特点是再使用读锁，写锁时都必须配合戳使用

乐观锁 + 读写锁。读取完毕后用一次戳校验，如果校验通过，表示这期间没有写操作，数据可以安全使用。如果没通过，需要重新获取读锁，保证数据安全。

读方法的时候由原来的悲观锁转为乐观锁

## Semaphore 信号量

用来限制能同时访问共享资源的线程上限

```java
public class test {
    // Semaphore
    public static void main(String[] args) {

        Semaphore s = new Semaphore(3); // 共享资源数量为3
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                try {
                    s.acquire(); // 获取信号量 获取不到的则会阻塞
                    System.out.println("我是线程" + Thread.currentThread().getName());
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                } finally {
                    s.release(); // 释放信号量
                }
            }).start();
        }

    }
}
```



## CountdownLatch 倒计时锁

用来进行线程同步协作，等待所有线程完成倒计时

```java
public class test {

    public static void main(String[] args) throws InterruptedException {

        CountDownLatch countDownLatch = new CountDownLatch(3);
        new Thread(()->{
            System.out.println("begin...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            countDownLatch.countDown();
            System.out.println("finish...");
        }).start();
        new Thread(()->{
            System.out.println("begin...");
            try {
                Thread.sleep(1500);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            countDownLatch.countDown();
            System.out.println("finish...");
        }).start();
        new Thread(()->{
            System.out.println("begin...");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            countDownLatch.countDown();
            System.out.println("finish...");
        }).start();

        System.out.println("waiting...");
        countDownLatch.await();
        System.out.println("waiting end...");

    }
}
```

线程池版countdownlatch

```java
public class test {

    public static void main(String[] args) throws InterruptedException {

        CountDownLatch countDownLatch = new CountDownLatch(3);
        ExecutorService service = Executors.newFixedThreadPool(4);
        service.submit(()->{
            System.out.println("begin...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            countDownLatch.countDown();
            System.out.println("finish...");
        });
        service.submit(()->{
            System.out.println("begin...");
            try {
                Thread.sleep(1500);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            countDownLatch.countDown();
            System.out.println("finish...");
        });
        service.submit(()->{
            System.out.println("begin...");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            countDownLatch.countDown();
            System.out.println("finish...");
        });

        service.submit(()->{
            try {
                System.out.println("waiting...");
                countDownLatch.await();
                System.out.println("waiting end...");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        });

    }
}
```

```java
import java.util.Arrays;
import java.util.Random;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class test {

    public static void main(String[] args) throws InterruptedException {

        ExecutorService service = Executors.newFixedThreadPool(10);
        CountDownLatch countDownLatch = new CountDownLatch(10);
        String[] all = new String[10];
        Random random = new Random();
        for (int j = 0; j < 10; j++) {
            int k = j; // lambda表达式只收局部常量，不受局部变量
            service.submit(() -> {
                for (int i = 0; i <= 100; i++) {
                    try {
                        Thread.sleep(random.nextInt(100));
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                    all[k] = i + "%";
                    System.out.print("\r" + Arrays.toString(all));
                }
                countDownLatch.countDown();
            });
        }

        countDownLatch.await();
        System.out.print("\r 游戏开始...");
        service.shutdown();
    }
}

```



## cyclicbarrier 循环栅栏

与countdown launch相比，可以重用，当等待的线程数满足计数个数时，继续执行

```java
public class test {

    public static void main(String[] args) throws InterruptedException {

        ExecutorService service = Executors.newFixedThreadPool(2);
        CyclicBarrier barrier = new CyclicBarrier(2, ()->{
            System.out.println("task1, task2 finish");
        });

        for (int j = 0; j < 3; j++) {
            service.submit(()->{
                System.out.println("task1 begin ...");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                try {
                    barrier.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                } catch (BrokenBarrierException e) {
                    throw new RuntimeException(e);
                }
            });
            service.submit(()->{
                System.out.println("task2 begin ...");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                try {
                    barrier.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                } catch (BrokenBarrierException e) {
                    throw new RuntimeException(e);
                }
            });
        }
        service.shutdown();
    }
}
```



# concurrenthashmap







