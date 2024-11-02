## 字符串转化为数字：通过为遍历 + 判断ASCII码

### C++

```c++
string str = "abc123"
for (int i = 0; i < n; i++) {
	if(str[i] >= '0' && str[i] <= '9') {
        cout<<str[i] - '0';
    }
}
```

### Java

```java
// 方法一：如果字符串为数字类型，直接通过函数来把字符串转数字
Integer.parseInt(tokens[i]);
// 方法二：通过char来实现获取数字
String str = "123456";
ArrayList arrayList = new ArrayList<>();
for (int i = 0; i < str.length(); i++) {
    arrayList.add(str.charAt(i) - '0');
}
```

## 长度函数length，length( )，及size( )的区别

```java
// 1. length成员是在普通静态数组中
int[] nums = {1,2,3,4,5};
int n = nums.length;

// 2. length()成员函数是在String类型中
String str = "abc1234";
int m = str.length();

// 3. size()成员函数是在容器内
ArrayList<Integer> array = new ArrayList<>();
int k = array.size();
```

## 判断两个数哪个大，哪个小

```java
import java.util.*;
int n = 1, m = 2;
System.out.println(Math.min(n,m));
System.out.println(Math.max(n,m));
```

## Java输入

```java
Scanner scanner = new Scanner(System.in);
//        String str = scanner.nextLine();
//        int n = scanner.nextInt();
//        double m = scanner.nextDouble();

        int[] array = new int[5];
        for(int i = 0; i < 5; i++) {
            array[i] = scanner.nextInt();
        }
//        System.out.println(str);
//        for (int x: array) {
//            System.out.println(x);
//        }
        System.out.println(array);
```



## Java中的排序函数

```java
// 使用Arrays.sort()对数组进行排序
import java.util.Arrays;
 
public class SortExample {
    public static void main(String[] args) {
        int[] numbers = {9, 5, 2, 7, 3};
        Arrays.sort(numbers);
        System.out.println(Arrays.toString(numbers)); // 输出 [2, 3, 5, 7, 9]
    }
}
 
// 使用Collections.sort()对列表进行排序
import java.util.Arrays;
import java.util.Collections;
import java.util.List;
 
public class SortExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(9, 5, 2, 7, 3);
        Collections.sort(numbers);
        System.out.println(numbers); // 输出 [2, 3, 5, 7, 9]
    }
}
 
// 使用Comparator进行自定义排序
import java.util.Arrays;
import java.util.Comparator;
 
public class SortExample {
    public static void main(String[] args) {
        String[] names = {"Bob", "Alice", "Charlie"};
        Arrays.sort(names, Comparator.comparing(String::toString).reversed());
        System.out.println(Arrays.toString(names)); // 输出 [Charlie, Bob, Alice]
    }
}
```



## Java自定义排序

```java
Arrays.sort(intervals, new Comparator<int[]>() {
	public int compare(int[] interval1, int[] interval2) {
    	return interval1[0] - interval2[0];
	}
});
```





## Java中返回int类型的最大值和最小值

```java
int minValue = Integer.MIN_VALUE;
int maxValue = Integer.MAX_VAUE;
```



## 优先队列创建

```java
public static void main(String[] args) {
        // 默认是小根堆
        PriorityQueue<Integer> queue = new PriorityQueue<>();
        // 不能插入null元素，会报错
        // 如果是大顶堆的话，要重写比较器
        PriorityQueue<Integer> queue1 = new PriorityQueue<>((x1, x2)-> {
            return x2 - x1;
        });

        List<Integer> array = new ArrayList<>(Arrays.asList(1,2,3,4,5));
        Collections.sort(array, (Integer x1, Integer x2)-> x2 - x1);


        queue1.offer(1);
        queue1.offer(2);
        queue1.offer(3);

        System.out.println(array);


    }
```

