# Java基础

## 深复制 浅复制

浅复制是复制原来对象的原有的基础变量，对其他对象的引用还是指向原来的对象。

深复制是把要复制的对象所引用的对象都复制了一遍

```java
public class MyClass implements Cloneable {
    private int number;
    private List<String> list;
    MyClass() {
        this.number = 0;
        this.list = new ArrayList<>();
    }
    MyClass(int number) {
        this.number = number;
        this.list = new ArrayList<>();
    }
    void setNumber(int number) {
        this.number = number;
    }
    int getNumber() {
        return number;
    }
    @Override
    public MyClass clone() throws CloneNotSupportedException {
        MyClass copy = (MyClass) super.clone();
        // 手动进行深拷贝
        copy.list = new ArrayList<>(this.list); // 进行深拷贝
        return copy;
    }
}
public class Main{
    public static void main(String[] args) throws CloneNotSupportedException {
        MyClass myClass1 = new MyClass(3);
        MyClass myClass2 = myClass1.clone();
        myClass1.setNumber(2);
        System.out.println(myClass2.getNumber()); // 3
    }
}
```



## String的基础方法

```java
string str = "";
str.charAt();
str.split("-");
str.trim();
str.indexof(String substr);// 获取子串的第一个位置

str.toCharArray();
```



## final和static的区别

final表示是常量。

- 只能被赋值一次，赋值后不能再被改变
- final类不能被继承，没有子类，final类中的方法默认为final

static表示是静态变量

- 被static修饰的成员变量独立于该类的任何对象，static修饰的变量可以被赋值多次。
- static类不能被继承，但是**可以不用初始化而访问**。class.forName();

## native方法

native方法允许java程序调用其他非java编程语言所编写的代码，通过native关键字，Java程序可以调用底层的操作系统功能、硬件功能或者第三方库的功能。

## == 和equals

==：作用是判断两个**对象的地址**是不是相等。

equals：判断内容是否相等

## hashcode

hashcode的作用能通过“键”快速检索出对应的“值”。

hashcode()和equals()都是用于比较两个对象是否相等。再一些容器中有了hashcode，判断元素是否在对应容器中效率会更高，如果相等再使用equals()来判断是否相同。

## 反射

反射是用来根据对象来分析类能力。主要是因为它赋予了我们在运行时分析类以及执行类中方法的能力。通过反射你可以获取任意一个类的所有属性和方法，你还可以调用这些方法和属性。

在运行状态中，对于任意一个实体类，都能够知道这个类的所有属性和方法；

对于任意一个对象，都能够调用它的任意方法和属性。

```java
//1.通过getclass()获取
// 静态方法Class.forName(String className)
Employee e;
e.getclass().getName();
//2. 判断是否为某个类的实例
// 利用instanceof关键字来判断是否为某个类的实例。同时借助反射中Class对象的isInstance()方法来判断某个类的实例

// 7、调用方法
public static void main(String[] args) throws IllegalAccessException, InstantiationException, NoSuchMethodException{
        Class<?> klass = methodClass.class;
        //创建methodClass的实例
        Object obj = klass.newInstance();
        //获取methodClass类的add方法
        Method method = klass.getMethod("add",int.class,int.class);
        //调用method对应的方法 => add(1,4)
        Object result = method.invoke(obj,1,4);
        System.out.println(result);
    }

```

### 动态代理

在程序执行过程中，利用反射机制，创建代理对象，并动态的指定代理目标类。主要是实现invocationHandler接口，重写里面的invoke方法。

```java
public class DebugInvocationHandler implements InvocationHandler {
    /**
     * 代理类中的真实对象
     */
    private final Object target;

    public DebugInvocationHandler(Object target) {
        this.target = target;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws InvocationTargetException, IllegalAccessException {
        System.out.println("before method " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("after method " + method.getName());
        return result;
    }
}
```





## 抽象类和接口

抽象类和接口都是用来继承

抽象类的话抽象方法不能写，接口中抽象方法可以有默认方法

抽象类只能继承一个，接口实现可以很多个。



## String、StringBuffer、StringBuilder

String方法是不可变的

StringBuffer是线程安全的，使用了大量的同步块

StringBuilder是线程不安全。

- 操作少量的数据: 适用 `String`
- 单线程操作字符串缓冲区下操作大量数据: 适用 `StringBuilder`
- 多线程操作字符串缓冲区下操作大量数据: 适用 `StringBuffer`

# Java数据结构

### 哈希表

HahsMap默认的初始化大小为16，每次扩容都会变为原来的2倍。

当链表长度大于阈值（默认为8）时，开始调用 treeifyBin () 方法

1. 当数组长度大于或者等于64时，才会转换位红黑树，以减少搜索事件
2. 否则只是执行resize( ) 方法对数组进行扩容，然后重新进行分配



```java
import java.util.*;
public class main {
    public static void main(String args[]) {
        Map<String, Integer> umap = new HashMap<>();
        umap.put("one", 1);
        umap.put("two", 2);
//        System.out.println(umap.get("hello"));
        Set<String> s = umap.keySet();
        if(umap.containsKey("one")) {
            System.out.println("one")
        }
        for (Object key: s) {
            System.out.println(umap.get(key)); // get() 不存在返回null
        }
        Set<Integer> values = umap.values();
        // getOrDefault(key, value); 在哈希表中查找key，如果没找到则赋予默认值value
	    List<> result = umap.getOrDefault(key, new ArrayList<>());
        // containsKey(key) 判断在哈希表中是否存在该key
        if(umap.containsKey(key)) {
            return true;
        }
        // 同时获取key，value
        Map<Integer, Integer> cnt = new HashMap<Integer, Integer>();
        cnt.put(1,2);
        cnt.put(2,3);

        for (Map.Entry<Integer, Integer> x : cnt.entrySet()) {
            System.out.println(x);
        }
    }
}
```

插入 put( )， 获取 get( ) ,遍历hash表的通过keyset()取出所有的key值，再通过取出对应的value

### 动态数组

ArrayList

1. 一开始初始化赋值的是一个空数组。
2. 当开始添加元素时，才真正分配容量，将容量扩为10
3. 当超过容量时，会调用grow方法进行扩容，扩为原来容量的1.5倍。

增加add()， 

```java
public class A{
    public static void main(String args[]) {
        Scanner scanner = new Scanner(System.in);
        List<String> list = new ArrayList<String>();
        list.add("12");
        list.add("13");
        for (Object obj: list) {
            System.out.println(obj);
        }
    }
}
```

### 链表

利用class来实现

```java
import java.util.*;
class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) {
        this.val = val;
    }
    ListNode(int val, ListNode next) {
        this.val = val;
        this.next = next;
    }
}

public class A{
    public static void main(String args[]) {
        Scanner scanner = new Scanner(System.in);
        
    }
}
```

### 栈

```java
public class A{
    public static void main(String args[]) {
        Stack<TreeNode> stk = new Stack<TreeNode>();
        stk.push(1);
        stk.push(2); // offer和push都可以插入元素，但是push是队尾插入，offer是队首
        int m = stk.peek(); // 取栈顶元素，peek是2
        stk.pop();
    }
}
```



### 队列

通过连接链表来实现队列

```java
public class A{
    public static void main(String args[]) {
        Queue<String> queue = new LinkedList<String>();
        queue.offer("1");
        queue.offer("2");
        System.out.println(queue.peek());
        while(!queue.isEmpty()){
            System.out.println(queue.remove());
            queue.poll();// 删除队头元素，并返回
        }
    }
}

（1）添加元素：
boolean add(E element): 将指定的元素添加到队列的末尾，如果成功则返回true，如果队列已满则抛出异常。
boolean offer(E element): 将指定的元素添加到队列的末尾，如果成功则返回true，如果队列已满则返回false。
（2）移除元素：
E remove(): 移除并返回队列头部的元素，如果队列为空则抛出异常。
E poll(): 移除并返回队列头部的元素，如果队列为空则返回null。
（3）获取头部元素：
E element(): 获取队列头部的元素，但不移除它，如果队列为空则抛出异常。
E peek(): 获取队列头部的元素，但不移除它，如果队列为空则返回null。
（4）队列大小：
int size(): 返回队列中的元素个数。
boolean isEmpty(): 判断队列是否为空。

```

### 队列

```java
ArrayDeque<Integer> arrayDeque = new ArrayDeque<>();
arrayDeque.add(1); // add添加元素
System.out.println(arrayDeque.peek()); // peek弹出队首元素

// 获取元素
arrayDeque.getFirst();
arrayDeque.pollFirst();
arrayDeque.getLast();
arrayDeque.pollLast();
arrayDeque.offerFirst();
arrayDeque.offerLast();
```





### set集合

```java
Set<String> s = new HashSet<String>();
s.add("1");
s.add("2");
s.add("2");
System.out.println(s.size());
// contains(key) 判断在集合中是否存在key 
s.contains("1")
// 删除对应key
s.remove("key");
```



### 优先队列

```java
//在Java中，优先队列是通过util.PriorityQueue类实现的。
public static void main(String[] args) {
        // 默认是小根堆
        PriorityQueue<Integer> queue = new PriorityQueue<>();
        // 不能插入null元素，会报错
        // 如果是大顶堆的话，要重写比较器
        PriorityQueue<Integer> queue1 = new PriorityQueue<>((x1, x2)-> {
            return x2 - x1;// 
        });

        List<Integer> array = new ArrayList<>(Arrays.asList(1,2,3,4,5));
        Collections.sort(array, (Integer x1, Integer x2)-> x2 - x1);


        queue1.offer(1);
        queue1.offer(2);
        queue1.offer(3);

        System.out.println(array);
    }
```







## 序列化和反序列化

序列化：将数据结构或对象转换成二进制字节流的过程

反序列化：将二进制字节流转换尾数据结构或者对象的过程

```java

```

### HashMap

JDK1.8之前的hashMap是由数组+链表组成的，数组是hashMap的主体，链表则是主要为了解决哈希冲突而存在。

1.8之后了，当链表长度大于阈值（默认为8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。



### ConcurrentHashMap和hashmap的区别



## Java8中Stream流

Stream是Java8的一大亮点，是对容器对象功能的增强，它专注于对容器对象进行各种非常便利、高效的 **聚合操作（aggregate operation）**或者大批量数据操作。同时，它提供串行和并行两种模式进行汇聚操作，并发模式能够充分利用多核处理器的优势，使用fork/join并行方式来拆分任务和加速处理过程。

```java
// 1. Individual values
Stream stream = Stream.of("a", "b", "c");

// 2. Arrays
String [] strArray = new String[] {"a", "b", "c"};
stream = Stream.of(strArray);
stream = Arrays.stream(strArray);

// 3. Collections
List<String> list = Arrays.asList(strArray);
stream = list.stream();
```



## 泛型类、泛型方法、泛型接口

```java
//泛型类：当一个类中，某个变量的数据类型不确定时，可以定义带有泛型的类
class ArrayList<E> {
    
}
// 泛型方法：当方法中形参类型不确定时，可以使用类名后面定义的泛型<E>
public <E> boolean add(E e) {
    
}
// 泛型接口
1- 方法1 实现类给出具体类型
2- 方法2 实现类延续泛型，创建对象时再确定
```



## IO流

- InputStream( )
- OutputStream( )

字节流是FileInputStream、FileOutputStream( )

字符流是

```java
// 字节流转换为字符流的桥梁
public class InputStreamReader extends Reader {
}
// 用于读取字符文件
public class FileReader extends InputStreamReader {
}
//
try (FileReader fileReader = new FileReader("input.txt");) {
    int content;
    long skip = fileReader.skip(3);
    System.out.println("The actual number of bytes skipped:" + skip);
    System.out.print("The content read from file:");
    while ((content = fileReader.read()) != -1) {
        System.out.print((char) content);
    }
} catch (IOException e) {
    e.printStackTrace();
}

```

字节缓冲流 **BufferedInputStream** ，采用装饰器模式来强化FileInputStream( )

可以有缓冲区，大幅减少IO次数，提高读取效率

字符缓冲流 BufferReader



## 随机访问流

可以支持随意跳转到文件的任意位置进行读写的 **RandomAccessFile**。

大部分用于大文件分片下载的情况



