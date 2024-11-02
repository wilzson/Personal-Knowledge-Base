```java
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.*;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;


public class B implements Runnable{
    // 打印次数
    private static final int PRINT_COUNT = 10;
    private final ReentrantLock reentrantLock;
    private final Condition thisCondtion;
    private final Condition nextCondtion;
    private final char printChar;
    public B(ReentrantLock reentrantLock, Condition thisCondtion, Condition nextCondition, char printChar) {
        this.reentrantLock = reentrantLock;
        this.nextCondtion = nextCondition;
        this.thisCondtion = thisCondtion;
        this.printChar = printChar;
    }

    @Override
    public void run() {
        // 获取打印锁，进入临界区
        reentrantLock.lock();
        try {
            for(int i = 0; i < PRINT_COUNT; i++) {
                System.out.println(printChar);
                nextCondtion.signal(); // 唤醒其他线程
                if (i < PRINT_COUNT - 1) {
                    try {
                        thisCondtion.await();// 当前线程暂停，并放弃锁
                    }catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }finally {
            reentrantLock.unlock();
        }
    }
    public static void main(String[] args) throws InterruptedException {
        ReentrantLock lock = new ReentrantLock();
        Condition conditionA = lock.newCondition();
        Condition conditionB = lock.newCondition();
        Condition conditionC = lock.newCondition();
        Thread printA = new Thread(new B(lock, conditionA, conditionB,'A'));
        Thread printB = new Thread(new B(lock, conditionB, conditionC,'B'));
        Thread printC = new Thread(new B(lock, conditionC, conditionA,'C'));
        printA.start();
//        Thread.sleep(100);
        printB.start();
//        Thread.sleep(100);
        printC.start();
    }
}

```

