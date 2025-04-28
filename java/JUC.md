# 基本方法

- 使用logback和lombok作为日志打印

```xml
<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.36</version>
    </dependency>

    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.5.18</version>
    </dependency>
</dependencies>
```

## 创建线程

- 启动JVM(main方法)，即开启一个JVM进程
- JVM进程内包含一个主线程，主线程可以派生出多个其他线程，同时还有一些守护线程(比如垃圾回收线程)
- 主线程，守护线程，派生线程等，cpu随机分配时间片，交替随机执行

### 继承Thread

- 继承Thread,重写run()，start()启动线程

```java
package com.citi;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Demo01 {
    public static void main(String[] args) {
        ErickThread erickThread = new ErickThread();
        erickThread.setName("t-1");
        erickThread.start();
        log.info("main running...");
    }
}

@Slf4j
class ErickThread extends Thread {
    @Override
    public void run() {
        log.info("slave-thread running");
    }
}
```

```java
package com.citi;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Demo02 {
    public static void main(String[] args) {
        /*匿名内部类*/
        Thread erickThread = new Thread("t-1") {
            @Override
            public void run() {
                log.info("slave-running");
            }
        };
        erickThread.start();
        log.info("main thread running");
    }
}
```

### Runnable接口

- 实现Runnable接口，重写Runnable的run方法，并将该对象作为Thread构造方法的参数传递

```bash
# Runnable优点 
- 线程和任务分开了，更加灵活
- 容易和线程池相关的API结合
- 让任务脱离了继承体系，更加灵活，避免单继承
```

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Demo03 {
    public static void main(String[] args) {
        Thread thread = new Thread(new ErickTask());
        thread.setName("t-1");
        thread.start();
        log.info("main-thread");
    }
}

@Slf4j
class ErickTask implements Runnable {

    @Override
    public void run() {
        log.info("slave-thread");
    }
}
```

```java
// 匿名内部类
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Demo04 {
    public static void main(String[] args) {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                log.info("slave-threa");
            }
        }, "t-1");

        thread.start();
        log.info("main-thread");
    }
}
```

- Runnable vs Thread

```bash
# 策略模式
- 实际执行是调用的Runnable接口的run方法
- 因为将Runnable实现类传递到了Thread的构造参数里面
```

- Runnable接口

```java
// @FunctionalInterface修饰的接口，可以用lambda来创建
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

- Thread 类

```java
public class Thread implements Runnable {

    private Runnable target;
    
   // 如果是Runnable接口，则重写的是接口中的run方法
    public Thread(Runnable target) {
        this(null, target, "Thread-" + nextThreadNum(), 0);
    }
    // 如果是继承Thread,则重新的是这个run方法
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
}
```

### FutureTask

```java
package com.citi;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
import java.util.concurrent.TimeUnit;

@Slf4j
public class Demo03 {
    public static void main(String[] args) {
        FutureTask<String> task = new FutureTask<>(new Callable<String>() {
            @Override
            public String call() throws Exception {
                TimeUnit.SECONDS.sleep(3);
                return "erick";
            }
        });

        Thread thread = new Thread(task);
        thread.start();

        /*获取结果时，会阻塞主线程*/
        try {
            String result = task.get();
            log.info("result={}", result);
        } catch (InterruptedException | ExecutionException e) {
            throw new RuntimeException(e);
        }
        log.info("main end");
    }
}
```

## start/run

```bash
#  public synchronized void start() 
- 线程从new-->runnable,等待cpu分片从何如有机会转换到running
- 在主线程之外，再开启了一个线程
- 已经start的线程，不能再次start。     IllegalThreadStateException

# public void run()
- 线程内部调用start后，实际执行的一个普通方法
- 线程如果直接调用run()，只是在主线程内，调用了一个普通方法
```

## sleep/yield

### sleep

- 放弃cpu时间片，不释放锁

```bash
# public static native void sleep(long millis) throws InterruptedException;
# Thread的方法
- 线程放弃cpu，RUNNABLE --> TIMED_WAITED
- slee的线程，可以自己去打断自己的休眠
- sleep结束后，会变为RUNNABLE,并不会立即执行，而是等待cpu
```

```java
package com.citi;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

@Slf4j
public class Demo04 {
    public static void main(String[] args) throws InterruptedException {

        Thread thread = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        });

        log.info("state={}", thread.getState()); // NEW
        thread.start();
        log.info("state={}", thread.getState());  // RUNNABLE

        TimeUnit.SECONDS.sleep(2);
        log.info("state={}", thread.getState());  // TIMED_WAITING
        /*自己打断自己休眠*/
        thread.interrupt();
    }
}
```

### yield

- 线程让出cpu，让其他高优先级的线程先去执行
- 当前线程从RUNNING-->RUNNABLE
- 假如其他线程不用cpu，那么cpu又会立即分配到当前线程，可能压根就没停下来

### priority

- 只是一个CPU参考值，高优先级的会被分配到更多cpu时间片

```java
package com.citi;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Demo05 {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            int i = 0;
            while (true) {
                i++;
                log.info("t-1:{}", i);
            }
        });

        Thread t2 = new Thread(() -> {
            int i = 0;
            while (true) {
                i++;
                log.info("===================================t-2:{}", i);
            }
        });
        t1.setPriority(Thread.MAX_PRIORITY);
        t2.setPriority(Thread.MIN_PRIORITY);
        t1.start();
        t2.start();
    }
}

```

### sleep应用

- 程序一直执行，浪费cpu，可以让其间歇性休眠，让出cpu，避免空转

```java
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

@Slf4j
public class Demo08 {
    public void method01() {
        while (true) {
            log.info("do...");
        }
    }

    public void method02() throws InterruptedException {
        while (true) {
            TimeUnit.SECONDS.sleep(1);
            log.info("do..");
        }
    }
}
```

## join

- B中开启了A线程，A调用join，等待A执行结束后再去运行B线程

### 无限等待

```java
// 等某个线程执行完毕后
public final void join() throws InterruptedException 
```

```java
package com.citi;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

@Slf4j
public class Demo07 {

    public static void main(String[] args) throws InterruptedException {
        Thread first = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        });

        Thread second = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        });

        long start = System.currentTimeMillis();
        first.start();
        second.start();
        /*两个线程同时加入进来: 3s*/
        first.join();
        second.join();
        log.info("time:{}ms", System.currentTimeMillis() - start);
    }
}
```

### 超时等待

- 等待线程执行，超时后不在等待该线程

```java
// 最多等待时间
public final synchronized void join(long millis, int nanos)
```

```java
package com.citi;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

@Slf4j
public class Demo07 {

    private static int num = 0;

    public static void main(String[] args) {
        Thread first = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            num = 10;
            log.info("first finished={}", num);
        });

        first.start();
        /*first 等待1s后，继续向下执行原来的线程，同时first继续执行*/
        try {
            first.join(1000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        log.info("num:{}", num);
    }
}
```

## interrupt

```java
# 谁调用：打断谁
public void interrupt();

# 判断当前线程是否被打断： 默认为false，不会清楚打断标记
 public boolean isInterrupted();

# 会将打断标记重置为true
 public static boolean interrupted();
```

### 阻塞线程

- 打断阻塞线程，抛出InterruptedException，将打断标记从true重制为false(需要一点缓冲时间)
- sleep，join，wait的线程

```java
package com.citi;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

@Slf4j
public class Demo08 {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                log.error("t1 is interrupt");
                throw new RuntimeException(e);
            }
        });

        t1.start();
        log.info("t1={}", t1.isInterrupted()); // false
        TimeUnit.SECONDS.sleep(1);
        /*打断sleep后，标记就是true，但是interrupt并将true重置为false*/
        t1.interrupt();
        log.info("t1={}", t1.isInterrupted());  // true

        TimeUnit.SECONDS.sleep(1);
        log.info("t1={}", t1.isInterrupted()); // false
    }
}
```

### 正常线程

- 收到打断，正常执行，但是打断信号标为true

```java
package com.citi;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

@Slf4j
public class Demo08 {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            while (true) {

            }
        });

        t1.start();
        log.info("erick={}", t1.isInterrupted()); // false
        t1.interrupt();
        log.info("erick={}", t1.isInterrupted()); // true
        TimeUnit.SECONDS.sleep(1);
        log.info("erick={}", t1.isInterrupted()); // true
    }
}
```

- 可以通过打断标记来进行普通线程的打断控制

```java
package com.citi;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

@Slf4j
public class Demo08 {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            while (true) {
                if (Thread.currentThread().isInterrupted()) {
                    break;
                }
            }
        });

        t1.start();
        log.info("erick={}", t1.isInterrupted()); // false
        TimeUnit.SECONDS.sleep(1);
        t1.interrupt();
        log.info("erick={}", t1.isInterrupted()); // true
        TimeUnit.SECONDS.sleep(1);
        log.info("erick={}", t1.isInterrupted()); // true
    }
}
```

### Two Phase Termination

- 终止正常执行的线程：需要后续clean
- 两阶段：正常执行和阻塞线程
- 业务场景：定时任务线程，如果被打断，则终止任务

```java
package com.citi;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

public class Demo09 {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new ScheduleTask());
        thread.start();
        TimeUnit.SECONDS.sleep(3);
        thread.interrupt();
    }
}

@Slf4j
class ScheduleTask implements Runnable {
    @Override
    public void run() {
        /*没有被打断时，不断执行*/
        while (!Thread.currentThread().isInterrupted()) {
            interval();
            job();
        }
        /*被打断后，进行clear*/
        clear();
    }

    private void job() {
        log.info("doing job");
    }

    private void clear() {
        log.info("doing clear");
    }

    private void interval() {
        try {
            log.info("doing interval");
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            // 被打断时，会将标记重置为false，需要调用该方法将打断标记标志为true
            Thread.currentThread().interrupt();
        }
    }
}
```

## daemon

- 普通线程：只有当所有线程结束后，JVM才会退出
- 守护线程：只要普通线程结束了，即使守护线程的代码还没执行完，也会强制退出(垃圾回收器)

```java
package com.citi.d01;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

@Slf4j
public class Demo01 {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            log.info("demon start");
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.info("demon end"); // 不会执行
        });

        thread.setDaemon(true);
        thread.start();
        TimeUnit.SECONDS.sleep(1);
        log.info("main end");
    }
}
```

## 其他方法

|              | 方法                                                         | 备注                                                         |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 名字         | public final synchronized void setName(String name);<br/>public final String getName(); |                                                              |
| 优先级       | public final void setPriority(int newPriority);<br/>public final int getPriority(); | - 最小为1，最大为10，默认为5<br/>- cpu执行目标线程的顺序  建议设置<br/>- 任务调度器可以忽略它进行分配资源 |
| 线程id       | public long getId();                                         | 13                                                           |
| 是否存活     | public final native boolean isAlive();                       |                                                              |
| 是否后台线程 | public final boolean isDaemon();                             |                                                              |
| 获取线程状态 | public State getState()                                      | NEW  RUNNABLE  BLOCKED    <br/>WAITING   TIMED_WAITING   TERMINATED |
| 获取当前线程 | public static native Thread currentThread()                  |                                                              |

## 生命周期

### 操作系统

- CPU时间片只会分给可运行状态的线程，阻塞状态的需要其本身阻塞结束

![image-20240405161947365](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240405161947365.png)

**NEW** 

- new 出一个Thread的对象时，线程并没有创建，只是一个普通的对象
- 调用start，new 状态进入runnable状态

**RUNNABLE**

- 调用start，jvm中创建新线程，但并不立即执行，只是处于就绪状态
- 等待cpu分配时间片，轮到它时，才真正执行

**RUNNING**

- 一旦cpu调度到了该线程，该线程真正执行

```bash
# 该状态的线程可以转换为其他状态
- 进入BLOCK：        sleep， wait，阻塞IO如网络数据读写， 获取某个锁资源
- 进入RUNNABLE：     cpu轮询到其他线程，线程主动yield放弃cpu
```

**BLOCK**

```bash
# 该状态的线程可以转换为其他状态
- 进入RUNNABLE：      线程阻塞结束，完成了指定时间的休眠
                     wait中的线程被其他线程notify/notifyall
                     获取到了锁资源
```

**TERMINATED**

-  线程正常结束
-  线程运行出错意外结束
-  jvm crash,导致所有的线程都结束

### JAVA层面

- 根据Thread类中内部State枚举，分为六种状态

```
NEW     RUNNABLE    BLOCKED    WAITING    TIMED_WAITING   TERMINATED
```

# 线程安全

## 不安全

### 场景

- 多线程对共享变量的并发修改

```java
package com.citi.d01;

import lombok.Getter;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Demo02 {
    public static void main(String[] args) throws InterruptedException {
        ErickService erickService = new ErickService();
        Thread first = new Thread(erickService::incr);
        Thread second = new Thread(erickService::decr);
        first.start();
        second.start();
        first.join();
        second.join();
        log.info("num={}", erickService.getNum());
    }
}

class ErickService {
    @Getter
    private int num;

    public void incr() {
        for (int i = 0; i < 100000; i++) {
            num++;
        }
    }

    public void decr() {
        for (int i = 0; i < 100000; i++) {
            num--;
        }
    }
}
```

### 原因

#### 字节码

- 线程拥有自己的stack内存，读取数据时会从heap内存中拿，写完后会将数据回写到heap中
- i++在执行时，是一系列指令，一系列指令就会导致指令交错

![image-20250425123721235](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250425123721235.png)

#### 指令交错

- 多线程之间，上下文切换时，不同线程内指令交错，最终导致结果不为0

![image-20250425124713719](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250425124713719.png)

#### 发生场景

- 只存在于多个线程共享资源
- 临界区：对共享资源的，多线程的读写的代码块

```bash
多线程     读        没问题
          读写      可能不安全(指令交错)
```

## synchronized

- 对象锁：同一个不可变对象，引用数据类型

### 同步代码块

```bash
- 阻塞式，可在不同代码粒度控制
- 保证临界区代码的原子性(字节码)，不会因为线程上下文切换而被打断

# 原理
1. a线程获取锁，执行临界区代码
2. b线程尝试获取锁，失败后被block，进入等待队列，同时进入上下文切换
3. a线程执行完毕后，释放锁资源。 唤醒其他线程，进行cpu的争夺
```

```java
class LucyService {
    @Getter
    private int number;

    /*final：保证引用不变，确保是同一个锁*/
    private final Object lock = new Object();

    /*可在不同粒度进行控制,一般会用this作为锁对象*/
    public void incr() {
        for (int i = 0; i < 100000; i++) {
            synchronized (lock) {
                number++;
            }
        }
    }

    public void decr() {
        for (int i = 0; i < 100000; i++) {
            synchronized (lock) {
                number--;
            }
        }
    }
}
```

### 同步方法

#### 普通方法

- 同步方法的锁对象：this，即为当前对象
- 同步方法和同步代码块类似，可能在锁粒度上不太一样

```java
class LucyService {
    @Getter
    private int number;


    public synchronized void incr() {
        for (int i = 0; i < 100000; i++) {
            number++;
        }
    }

    public void decr() {
        for (int i = 0; i < 100000; i++) {
            synchronized (this) {
                number--;
            }
        }
    }
}
```

#### 静态方法

- 锁对象：当前类的字节码对象

```java
class LucyService {
    @Getter
    private static int number;

    public static synchronized void incr() {
        for (int i = 0; i < 100000; i++) {
            number++;
        }
    }

    /*只要锁的是同一个对象即可*/
    public static void decr() {
        for (int i = 0; i < 100000; i++) {
            synchronized (LucyService.class) {
                number--;
            }
        }
    }
}
```

## 变量安全

### (静态)成员变量

```bash
# 不被多线程共享：                    ✅
# 被多线程共享：
                      只读          ✅
                      读写          ❌
```

### 局部变量

- 局部变量是线程私有。每个线程，都会创建自己单独的栈内存，局部变量保存在自己当前线程的栈内存中

```bash
# 基本数据类型                                                  ✅
# 引用数据类型 
                        对象没有逃离方法作用域                    ✅
                        对象逃离方法作用域(引用逃离)               ❌
```

#### 引用逃逸

- 类被继承时并且方法重写时，重写方法创建新的线程，就可能发生引用逃逸类型的问题
- 解决方法：类被final修饰或private方法不让子类去改变

```java
// 安全
package com.erick.multithread.d03;

import java.util.List;

class SafeCounter {
    public void handle(List<String> data) {
        for (int i = 0; i < 100000; i++) {
            add(data);
            remove(data);
        }
    }

    public void add(List<String> data) {
        data.add("hello");
    }

    public void remove(List<String> data) {
        data.remove(0);
    }
}
```

```java
// 不安全
package com.erick.multithread.d03;

import java.util.List;

public class UnsafeCounter extends SafeCounter {

    @Override
    public void remove(List<String> data) {
        new Thread() {
            @Override
            public void run() {
                data.remove(0);
            }
        }.start();
    }
}
```

### 线程安全类

#### JDK

- 多线程同时调用他们同一个实例的方法时，线程安全
- 线程安全类中的方法的组合，不一定线程安全
- 如果要线程安全，必须将组合方法也设置成 synchronized

```bash
- String
- Integer
- StringBuffer
- Random
- Vector
- Hashtable
- java.util.concurrent包下的类
```

#### 不可变类

- 类中属性都是final，不能修改。如 String，Integer

```bash
#  实现线程安全类方式
- 无共享变量
- 共享变量不可变
- synchronized互斥修饰
```

# Synchronized锁

## 对象头

- 堆内存中的Java对象中，对象头包含如下信息
- Mark Word:     锁信息，hashcode，垃圾回收器标志
- Klass Word:     指针，包含当前对象的Class对象的地址

### 普通对象

![image-20250427111914144](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427111914144.png)

### 数组对象

![image-20250427112017672](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427112017672.png)

## Mark Word

- 如果加锁时用到了该对象，该对象的Mark Word的状态会发生变化

![image-20250427112154080](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427112154080.png)

```bash
# hashcode            :java对象的hashcode
# age                 :对象的分代年龄，为垃圾回收器进行标记
# biased_lock         :是否是偏向锁

# 锁状态： 01/00/11
- 代表是否加锁
- 01: 无锁
- 00: 轻量级锁
- 10: 重量级锁
```

## 轻量级锁

- 锁对象被多个线程访问，但访问时间错开，不存在竞争
- 对使用者是透明的，Synchronized语法
- 加锁失败，则升级为重量级锁
- 轻量级锁和monitor无关，不阻塞

### 加锁流程

#### 创建锁记录

- 每个线程的栈帧都会包含一个锁记录的结构，存储锁对象的MarkWord

![image-20250427113155038](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427113155038.png)

#### CAS

- 锁记录中的Object reference指向锁对象，并尝试使用CAS替换Object中的Mark Word，将Mark Word的值存入锁记录
- CAS成功了，则表示加上了锁
- 可能CAS失败

![image-20250427114246373](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427114246373.png)

#### 解锁

- 执行完临界代码块后，再次交换，并从当前栈帧中删除Lock Record

![image-20250427114509246](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427114509246.png)

### 锁重入

#### 创建锁记录

![image-20250427114825540](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427114825540.png)

#### CAS

![image-20250427114926002](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427114926002.png)

#### 锁重入

- 在当前栈帧中，创建第二个锁记录
- 让第二个Lock Record中的Object Reference指向锁地址
- 尝试CAS时，发现该锁的Mark Word就是当前线程的另外一个Lock Record的地址
- 则第二个Lock Record会存一个null，计数器加一

![image-20250427115535875](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427115535875.png)

#### 解锁

- 解锁时候，如果发现了null，则表示有重入，则直接清空Lock Record即可
- 直到完全CAS并删除Lock Record为止

## 锁膨胀

### 轻量级锁加锁失败

- thread-0进行CAS交换时，发现Object已经是加锁状态了，因此加锁失败
- 引入monitor锁

![image-20250427120428988](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427120428988.png)

### monitor

- 监视器(管程)： 操作系统提供，会有多个不同的monitor

![image-20250427120551150](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427120551150.png)

```bash
# Owner    
- 保存当前获取到java锁的，线程指针
# EntryList
- 保存被java锁阻塞的，线程指针
# WaitSet:    
- 保存被java锁等待的，线程指针
```

### 加锁流程

#### 申请monitor

- thread-0进行CAS时失败，就进行锁膨胀
- 为锁对象Object申请Monitor锁，Java锁对象的Mark Word保存monitor地址，后两位同时变为10
- 该Monitor的EntryList保存thread-0的指针，同时thread-0进入阻塞状态

![image-20250427120843578](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427120843578.png)

#### 原CAS解锁

- Thread-1执行完临界代码块，解锁时，发现锁对象的Mark Word已经变成重量级锁的地址，进入重量级锁结束流程
- 清空monitor的Owner，并唤醒EntryList中的线程，随机唤醒一个线程，成为新的Owner

## 重量级锁

- 锁一旦成为重量级锁后，就不能再降级成为轻量级锁

### 锁竞争

- 如果当前锁已经是重量级锁
- thread-0已经成为monitor的owner，其他线程过来后，就会进入monitor的EntryList来进行阻塞等待

![image-20250427121025875](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427121025875.png)

### 解锁

- thread-0执行完毕后，会将monitor中的Owner清空，同时通知该monitor的EntryList中的线程来抢占锁(非公平锁)
- 抢占到的新的thread，成为Owner中新的主人

### 😎 自旋优化

- 只属于重量级锁

```bash
- 一个线程的重量级锁被其他线程持有时，该线程并不会直接进入阻塞
- 先本身自旋，同时查看锁资源在自旋优化期间是否能够释放   《避免阻塞时候的上下文切换》
- 若当前线程自旋成功(即此时持有锁的线程已经退出了同步块，释放了锁)，这时线程就避免了阻塞
- 若自旋失败，则进入EntryList中
```

```bash
智能自旋： 
-  自适应的: Java6之后，对象刚刚的一次自旋成功，就认为自旋成功的概率大，就会多自旋几次
            反之，就少自旋几次甚至不自旋
- java7之后不能控制是否开启自旋功能
- 自旋会占用cpu时间，单核自旋就是浪费，多核自旋才有意义
```

### 锁重入

![image-20250427121230511](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427121230511.png)

```bash
# 创建锁记录
- 线程在自己的工作内存内，创建栈帧，并在活动栈帧创建一个  《锁记录》  的结构
- 锁记录： lock record address: 加锁的信息，用来保存当前线程ID等信息, 同时后续会保存对像锁的Mark Word
          Object Reference:  用来保存锁对象的地址
          00: 表示轻量级锁， 01代表无锁
          
- 锁记录对象：是在JVM层面的，对用户无感知         
- Object Body: 该锁对象的成员变量

# 加锁cas-- compare and set
# cas成功
- 尝试cas交换Object中的 Mark Word和栈帧中的锁记录

# cas失败
- 情况一：锁膨胀，若其他线程持有该obj对象的轻量级锁，表明有竞争，进入锁膨胀过程，加重量级锁
- 情况二：锁重入，若本线程再次synchronized锁，再添加一个Lock Record作为重入计数
- 两种情况区分： 根据obj中保存线程的lock record地址来进行判断
- null： 表示重入了几次

# 解锁cas
- 退出synchronized代码块时，若为null的锁记录，表示有重入，这时清除锁记录（null清除）
- 退出synchronized代码块时，锁记录不为null，cas将Mark Word的值恢复给对象头
  同时obj头变为01无锁状态
- 成功则代表解锁成功； 失败说明轻量级锁进入了锁膨胀
```



# Wait/Notify

- 线程在加锁执行时，发现条件不满足，则进行wait，同时释放锁
- 条件满足后，通过notify唤醒，同时继续竞争锁

## 原理

- Owner线程发现条件不满足，调用当前锁的wait，进入WaitSet变成WAITING状态，同时释放锁

```bash
- BLOCK和WAITING的线程， 处于阻塞状态，   不占用cpu
- BLOCK线程会在Owner线程释放锁时竞争锁
- WAITING线程会在Owner线程调用notify时唤醒，但唤醒后进入EntryList进行阻塞
```

![image-20250426171608243](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250426171608243.png)

## API

- Object类的方法，必须成为锁的owner才能使用，否则：java.lang.IllegalMonitorStateException: current thread is not owner
- 调用当前锁对象的lock，notify方法

```bash
# 当前持有锁的线程进入WaitSet, 一直等待
public final void wait() throws InterruptedException

# 当前持有锁的线程只等待一定时间，然后从 WaitSet 重新进入EntryList来竞争锁资源
# 也可以被提前唤醒
public final native void wait(long timeoutMillis) throws InterruptedException

# 随便唤醒一个线程，进入到EntryList
public final native void notify();

# 唤醒所有的线程，进入到EntryList
public final native void notifyAll()
```

### 基本使用

```java
package com.citi.d01;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

public class Demo04 {
    public static void main(String[] args) throws InterruptedException {
        WorkService workService = new WorkService();
        new Thread(workService::job).start();
        new Thread(workService::job).start();

        TimeUnit.SECONDS.sleep(1);
        new Thread(workService::wakeUp).start();
    }
}

@Slf4j
class WorkService {

    public synchronized void job() {
        String name = Thread.currentThread().getName();
        log.info("{} job start", name);
        try {
            /*调用锁的this*/
            this.wait();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        log.info("{} job end", name);
    }

    public synchronized void wakeUp() {
        String name = Thread.currentThread().getName();
        log.info("{} wakeup start", name);
        /*调用锁的this*/
        this.notifyAll();
        log.info("{} wakeup end", name);
    }
}
```

### Wait vs Sleep

```bash
- Wait 是Object的方法，调用lock的方法                     Sleep 是Thread 的静态方法
- Wait 必须和synchronized结合使用                        Sleep 不需要
- Wait 会释放当前线程的锁                                 Sleep 不会释放锁（如果工作时候带锁）

# 都会让出cpu资源，状态都是Timed-Waiting
```

## 虚假唤醒

- 循环wait： 线程一定是执行完毕任务后才会结束

```java
// 工作线程
synchronized(lock){
  while(条件不成立){
    lock.wait();
  }
  executeBusiness();
};

//其他线程唤醒
synchronized(lock){
  // 实现上述条件
  lock.notifyAll();
}
```

```java
package com.citi.d01;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

public class Demo05 {
    public static void main(String[] args) throws InterruptedException {
        JobService service = new JobService();
        new Thread(service::manWork).start();

        TimeUnit.SECONDS.sleep(1);
        new Thread(service::deliverCigarette).start();
    }
}

@Slf4j
class JobService {
    private boolean hasCigarette = false;

    public synchronized void manWork() {
        while (!hasCigarette) {
            log.info("没有烟，休息会");
            try {
                this.wait();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        executeJob();
    }

    public synchronized void deliverCigarette() {
        log.info("送烟来了");
        hasCigarette = false; // 设置为false，可以进行虚假唤醒
        this.notifyAll();
    }

    private void executeJob() {
        log.info("烟来了，开始干活");
    }
}
```

## 保护性暂停

- GuardedSuspension：A线程等待B线程的一个执行结果，即同步模式

```bash
- 一个结果需要从A线程传递到B线程，让两个线程关联同一个GuardedObject
- JDK中， join的实现，Future的实现，就是采用保护性暂停
```

![image-20250426175550094](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250426175550094.png)

```java
@Slf4j
class GuardObject {
    private Object result;

    public synchronized void setObject() {
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        result = new Object();
        log.info("result completed");
        this.notifyAll();
    }

    /*无限等待*/
    public synchronized Object getObject() {
        while (result == null) {
            log.info("get waiting");
            try {
                this.wait();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        return result;
    }

    public synchronized Object getObject(long timeout, TimeUnit timeUnit) {
        long startTime = System.currentTimeMillis();
        long passedTime = 0;
        while (result == null) {
            long leftTime = timeUnit.toMillis(timeout) - passedTime;
            if (leftTime <= 0) {
                return null;
            }

            try {
                this.wait(leftTime); /*动态等待*/
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }

            passedTime = System.currentTimeMillis() - startTime;
        }
        return result;
    }
}
```

## 消息队列

- 结果从A类线程不断传递到B类线程
- 多个生产者/消费者，阻塞队列，异步消费
- 消息队列：先入先得，容量限制，满时不再生产消息，空时不再消费消息
- JDK中各种阻塞队列，就是用的这种方式

![image-20250427110307029](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427110307029.png)

```java
@Slf4j
class MessageBroker {

    private int capacity;

    private LinkedList<Object> blockingQueue = new LinkedList<>();

    public MessageBroker(int capacity) {
        this.capacity = capacity;
    }

    public synchronized void produce(Object data) {
        while (blockingQueue.size() >= capacity) {
            log.info("{}: full queue, producer wait", Thread.currentThread().getName());
            try {
                this.wait();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        blockingQueue.addFirst(data);
        this.notifyAll();
    }

    public synchronized Object consume() {
        while (blockingQueue.isEmpty()) {
            log.info("{}: empty result, consumer wait", Thread.currentThread().getName());
            try {
                this.wait();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        Object result = blockingQueue.removeLast();
        this.notifyAll();
        return result;
    }
}
```

# 锁特性

## 多锁

- 如果不同方法访问的不是同一个共享资源，则尽可能提供多把锁，否则会降低并发度
- 共享同一个资源的不同方法，才让他们去使用同一把锁

### 一把锁

```java
class BigRoom {
    private int firstNumber = 0;
    private int secondNumber = 0;

    public void firstJob() {
        synchronized (this) {
            firstNumber++;
            sleep();
        }
    }

    public void secondJob() {
        synchronized (this) {
            secondNumber++;
            sleep();
        }
    }

    private void sleep() {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

### 多把锁

```java
class BigRoom {
    private int firstNumber = 0;
    private final Object firstLock = new Object();
    private int secondNumber = 0;

    private final Object secondLock = new Object();

    public void firstJob() {
        synchronized (firstLock) {
            firstNumber++;
            sleep();
        }
    }

    public void secondJob() {
        synchronized (secondLock) {
            secondNumber++;
            sleep();
        }
    }

    private void sleep() {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## 活跃性

- 一个线程中的代码，因为某种原因，一直不能执行完毕

### 死锁

- 如果一个线程需要同时获取多把锁，就容易发生死锁

```bash
- 线程一：持有a锁，等待b锁
- 线程二：持有b锁，等待a锁
- 互相等待引发的死锁问题
- 哲学家就餐问题
- 定位死锁: 可以借助jconsole来定位死锁
- 解决方法： 按照相同顺序加锁就可以，但可能引发饥饿问题
```

```java
@Slf4j
class DeadLock {
    private final Object firstLock = new Object();
    private final Object secondLock = new Object();

    public void firstMethod() {
        synchronized (firstLock) {
            log.info("{} first lock coming", Thread.currentThread().getName());
            sleep();
            synchronized (secondLock) {
                log.info("{} second lock coming", Thread.currentThread().getName());
            }
        }
    }

    public void secondMethod() {
        synchronized (secondLock) {
            log.info("{} second lock coming", Thread.currentThread().getName());
            sleep();
            synchronized (firstLock) {
                log.info("{} first lock coming", Thread.currentThread().getName());
            }
        }
    }

    private void sleep() {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

### 饥饿

```bash
# 场景一：
- 某些线程，因为优先级别低，一直抢占不到cpu的资源，导致一直不能运行

# 场景二：
- 在加锁情况下，有些线程，一直不能成为锁的Owner，任务无法执行完毕
```

### 活锁

- 并没有加锁
- 两个线程中互相改变对方结束的条件，导致两个线程一直运行下去
- 可能会结束，但是二者可能会交替进行

```java
package com.erick.multithread.d03;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

@Slf4j
public class Demo03 {
    static volatile int number = 10;

    public static void main(String[] args) {
        new Thread(() -> {
            while (number < 20) {
                number++;
                log.info("+++++++ {}", number);
                try {
                    TimeUnit.MILLISECONDS.sleep(200);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }).start();

        new Thread(() -> {
            while (number > 0) {
                number--;
                try {
                    TimeUnit.MILLISECONDS.sleep(200);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.info("                         --------- {}", number);
            }
        }).start();
    }
}
```

# 内存模型

```bash
# 主存
- 所有线程都共享的数据，比如堆内存(成员变量)，静态成员变量(方法区)
# 工作内存
- 线程私有的数据，比如局部变量(栈帧中的局部变量表) 
```

## 可见性

- 一个线程对主存中数据写操作，对其他线程的读操作可见性

### 不可见

```java
package com.erick.multithread.d03;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

public class Demo04 {
    public static void main(String[] args) throws InterruptedException {
        // 主存的数据
        ErickService service = new ErickService();
        new Thread(() -> service.first(), "t-1").start();

        // 必须缓存一段时间，让JIT做从主存到工作内存的缓存优化
        TimeUnit.SECONDS.sleep(2);

        /*2s后，t-2 线程更新了flag的值，但是t-1线程并没有停下来*/
        new Thread(() -> service.updateFlag(), "t-2").start();
    }
}

@Slf4j
class ErickService {
    private boolean flag = true;

    public void first() {
        while (flag) {
        }
    }

    public void updateFlag() {
        log.info("update the flag");
        flag = !flag;
    }
}
```

### 原因

```bash
# 初始状态
- 初始状态，变量flag被加载到主存中
- t1线程从主存中读取到了flag
- t1线程要频繁从主存中读取数据,经过多次的循环后，JIT进行优化
- JIT编译器会将flag的值缓存到当前线程的栈帧工作内存中的高速缓存中(栈内存)，减少对主存的访问，提高性能
 
 # 2秒后
- 主存中的数据被t2线程修改了，但是t1线程还是读取自己工作内存的数据，并不会去主存中去拿
```

![image-20250427190550109](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427190550109.png)

### 解决方案

#### volatile

- 轻量级锁

```bash
- 修饰成员变量或者静态成员变量
- 线程获取该变量时，不会从自己的工作内存中去读取，每次都是去主存中读取
- 牺牲了性能，保证了一个线程改变主存中某个值时，对于其他线程不可见的问题
```

```java
@Slf4j
class ErickService {
    private volatile boolean flag = true;

    public void first() {
        while (flag) {
        }
    }

    public void updateFlag() {
        log.info("update the flag");
        flag = !flag;
    }
}
```

#### synchronized

- 重量级锁

```bash
# java内存模式中，synchronized规定，线程在加锁时
1. 先清空工作内存
2. 在主存中拷贝最新变量到工作内存中
3. 执行代码
4. 将更改后的共享变量的值刷新到主存中
5. 释放互斥锁
```

```java
@Slf4j
class ErickService {
    private boolean flag = true;

    public synchronized void first() {
        while (true) {
            synchronized (this) {
                if (!flag) {
                    break;
                }
            }
        }
    }

    public void updateFlag() {
        log.info("update the flag");
        flag = !flag;
    }
}
```

## 原子性

- 多线程访问共享资源，指令交错引发的原子性问题
- volatile：不保证
- synchronized：能解决

## 有序性

- JVM会在不影响正确性的前提下，可以调整语句的执行顺序
- 在单线程下没有问题，多线程下可能发生问题

### 计组思想

```bash
- 每条指令划分为    取指令 --- 指令译码 --- 执行指令 --- 内存访问 --- 数据回写
- 计组思想 并不能提高单条指令的执行时间，但是变相的提高了吞吐量
- 同一时间，对不同指令的不同步骤进行处理，提高吞吐量
```

![image-20250427191237748](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427191237748.png)

![image-20250427191253531](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427191253531.png)

### 指令重排

- 一行代码对应字节码可能分为若干个指令
- JVM在不影响性能的情况下，会对代码对应的字节码指令执行顺序进行重排

#### 单线程

- 指令重排不会对结果产生任何影响

#### 多线程

- 因为指令重排，可能引发诡异的问题

```bash
# 场景一：正常结果：10
- 线程2执行num=2时，线程1进入flag判断，result为10

# 场景二：正常结果：2
- 线程2执行完flag，线程1才开始，result为2

# 场景三：异常：0
- 线程2在执行时候指令重排：     num = 2; flag = true;---> flag = true; num = 2; 
- 线程2执行到flag = true，线程1开始，result为0
```

```java
class OrderServer {
    private int num = 0;
    private boolean flag = false;

    private int result;

    /*线程1执行该方法*/
    public void first() {
        if (flag) {
            result = num;
        } else {
            result = 10;
        }
    }

    /*线程2执行该方法*/
    public void second() {
        num = 2;
        flag = true;
    }
}
```

## 读写屏障-voliate

- Memory Barrier(Memory Fence):     底层实现是内存屏障

```bash
# 1. 写屏障
- voliate修饰的变量，会在写操作后，加上写屏障(代码块)
- 可见性 ：       在该屏障之前的所有代码的改动，同步到主存中去
- 有序性 ：       确保指令重排时，不会将写屏障之前的代码排在写屏障之后

# 2. 读屏障
- volatile修饰的变量，会在读取到该变量的前面，加上读屏障
- 可见性 ：      读屏障后面的数据，都会在主存中获取
- 有序性：       不会将读屏障之后的代码排在读屏障之前
```

### 可见性

```java
package com.erick.multithread.d05;

import java.util.concurrent.TimeUnit;

public class Demo04 {
    public static void main(String[] args) throws InterruptedException {
        TomService service = new TomService();
        new Thread(() -> service.work()).start();
        TimeUnit.SECONDS.sleep(2);
        new Thread(() -> service.changeValue()).start();
    }
}

class TomService {

    private boolean firstFlag = false;
    private volatile int age = 0;

    private boolean secondFlag = false;

    /**
     * 写屏障： age是volatile修饰，在age下面加上写屏障
     * age以及前面的firstFlag，secondFlag等写操作都会同步到主存中
     */
    public void changeValue() {
        firstFlag = true;
        secondFlag = true;
        age = 10;
        /*--------写屏障-------*/
    }


    /**
     * 读屏障： age是volatile修饰，在age上加上读屏障
     * 读屏障下面的所有变量，都是从主存中加载
     */
    public void work() {
        while (true) {
            /*--------读屏障--------*/
            int w_age = age;
            boolean w_firstFlag = firstFlag;
            boolean w_secondFlag = secondFlag;
            if (w_age == 10 && w_firstFlag && w_secondFlag) {
                break;
            }
        }
    }
}
```

### 有序性

-  单例模式: 双重加锁检查机制

```bash
# volatile作用：防止指令重排
# instance = new Singleton();   具体执行的字节码指令可以分为三步
1. 给instance分配内存空间
2. 调用构造函数来对内存空间进行初始化
3. 将构造好的对象的指针指向instance(这样对象就不为null了)

# 假如不加volatile
- 线程一执行时候，假如发生指令重排，按照132来构造，当执行到3的时候，其实对象还没初始化好
- 线程二在判断的时候，就发现instance已经不是null了，就返回了一个没有初始化完全的对象
```

```java
package com.erick.multithread.d02;

import com.erick.multithread.util.Sleep;

public class Demo05 {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(() -> System.out.println(Singleton.getInstance())).start();
        }
    }
}

class Singleton {
    /**
     * volatile作用：防止指令重排
     * instance = new Singleton();
     */
    private volatile static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            /*第一批的若干个线程都能到这里，
             * 但instance实例一旦初始化完毕，后续就不会进入这里了*/
            Sleep.sleep(2);

            synchronized (Singleton.class) {
                /*第一个线程初始化完毕后，第一批的后面所有，就不会初始化了，但是还是加锁判断的*/
                if (instance == null) {
                    instance = new Singleton();
                }
                return instance;
            }
        }

        return instance;
    }
}
```

## synchronized/volatile

|                        | synchronized         | volatile                 |
| ---------------------- | -------------------- | ------------------------ |
| 锁级别                 | 重量级锁             | 轻量级锁                 |
| 可见性                 | 可解决               | 可解决                   |
| 原子性(多线程指令交错) | 可解决               | 不保证                   |
| 有序性(指令重排)       | 可以重排，但不会出错 | 可以禁止重排             |
| 场景                   | 多线程并发修改       | 一个线程修改，其他线程读 |

## happen-before

- 对共享变量的读写操作，代码的可见性和有序性的总结

### synchronized

- 线程解锁时候，对变量的写，会从工作内存同步到主内存中，其他线程加锁的读，会从主内存中取

```java
package com.dreamer.multithread.day02;

public class Demo04 {
    private static int x = 0;

    private static Object lock = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (lock) {
                x = 10;
            }
        }).start();

        new Thread(() -> {
            synchronized (lock) {
                System.out.println(x);
            }
        }).start();
    }
}
```

### volatile

- 变量用volatile修饰，一个线程对其的写操作，对于其他线程来说是可见的
- 写屏障结合读屏障，保证是从主存中读取变量

### 先写先得

- 线程start前对变量的写操作，对该线程开始后的读操作是可见的

```java
package com.dreamer.multithread.day02;

public class Demo06 {
    private static int x = 0;

    public static void main(String[] args) {

        x = 10;

        new Thread(() -> System.out.println(x)).start();
    }
}
```

### 通知准则

- 线程结束前对变量的写操作，会将操作结果同步到主存中(join())
- 主要原因：public final synchronized void join(long millis)，加锁了

```java
package com.erick.multithread.d05;

import java.util.concurrent.TimeUnit;

public class Demo07 {

    private static boolean flag = true;

    public static void main(String[] args) throws InterruptedException {

        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                flag = false;
            }
        });
        t1.start();
        t1.join(); // 其实就是t1先执行，执行完毕后，已经将最新结果同步到主存中了

        while (flag) {

        }
    }
}
```

### 打断规则

- 线程t1 打断t2前，t1对变量的写，对于其他线程得知得知t2被打断后，对变量的读可见

```java
package com.erick.multithread.d05;

import lombok.Setter;

import java.util.concurrent.TimeUnit;

public class Demo08 {
    public static void main(String[] args) {
        PhoneServer server = new PhoneServer();

        Thread t2 = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(10000);
            } catch (InterruptedException e) {
            }
        });
        t2.start();

        Thread t1 = new Thread(() -> {
            // 写的455会生效
            server.setNumber(123);
            // 2秒后打断t2的休眠线程
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            t2.interrupt();
        });
        t1.start();

        new Thread(() -> server.firstMethod(t2)).start();
    }
}

class PhoneServer {
    @Setter
    private int number = 2;

    public void firstMethod(Thread interThread) {
        while (!interThread.isInterrupted() || number == 456) {

        }
    }
}
```

### 传递性

- 其实就是volatile的读屏障和写屏障

```java
package com.erick.multithread.d05;

import java.util.concurrent.TimeUnit;

public class Demo04 {
    public static void main(String[] args) throws InterruptedException {
        TomService service = new TomService();
        new Thread(() -> service.work()).start();
        TimeUnit.SECONDS.sleep(2);
        new Thread(() -> service.changeValue()).start();
    }
}

class TomService {

    private boolean firstFlag = false;
    private volatile int age = 0;

    private boolean secondFlag = false;

    /**
     * 写屏障： age是volatile修饰，在age下面加上写屏障
     * age以及前面的firstFlag，secondFlag等写操作都会同步到主存中
     */
    public void changeValue() {
        firstFlag = true;
        secondFlag = true;
        age = 10;
        /*--------写屏障-------*/
    }


    /**
     * 读屏障： age是volatile修饰，在age上加上读屏障
     * 读屏障下面的所有变量，都是从主存中加载
     */
    public void work() {
        while (true) {
            /*--------读屏障--------*/
            int w_age = age;
            boolean w_firstFlag = firstFlag;
            boolean w_secondFlag = secondFlag;
            if (w_age == 10 && w_firstFlag && w_secondFlag) {
                break;
            }
        }
    }
}
```

# CAS-无锁并发

- CAS需要voliate支持

## 线程不安全

- 可以通过 悲观锁-synchronized，来实现线程间的互斥，实现线程安全
- 也可以通过无锁并发CAS实现

```java
@Slf4j
class BankService {
    public int total = 10;

    public void drawMoney() {
        if (total <= 0) {
            log.info("no money left");
            return;
        }

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }

        total--;
        log.info("get money");
    }
}
```

## 乐观锁-CAS 

- 不加锁实现共享资源的保护
- Compare And Set
- JDK提供了对应的CAS类来实现不加锁

### API

```bash
AtomicInteger

private volatile int value;

# 构造方法
public AtomicInteger(int initialValue) {
    value = initialValue;
}

# 1. 获取最新值
public final int get();

# 2. 比较，交换
public final boolean compareAndSet(int expectedValue, int newValue)
```

```java
@Slf4j
class BankService {
    public AtomicInteger total = new AtomicInteger(10);

    public void drawMoney() {
        while (true) {
            int value = total.get(); // 获取最新值
            if (value <= 0) {
                log.info("no money left");
                return;
            }

            Sleep.sleep(1);

            /*参数一：原来的值
             * 参数二：变换后的值
             * 原来的值，一旦被其他线程修改了，是一个volatile修饰的，本线程立刻就会获取到，CAS就会失败，
             * 就会继续下次循环*/
            boolean result = total.compareAndSet(value, value - 1);
            if (result) {
                break;
            }
            log.info("CAS failure");
        }
    }
}
```

### 原理

- CAS 必须和 volatile结合使用
- get()方法获取到的是类的value，被volatile修饰。其他线程修改该变量后，会立刻同步到主存中，方便其他线程的cas操作
- compareAndSet内部，是通过系统架构实现的，也是一种锁

```java
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

![image-20250427193309965](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250427193309965.png)

### CAS vs synchronized

```bash
# 1. 解决方式
- CAS:             无锁并发(基于内存可见性，不加锁)，无阻塞并发(不会阻塞，上下文切换少)
- synchronized:    悲观锁，阻塞并发

# 2. 上下文切换
- CAS即使重试失败，线程一直高速运行
- synchronized会让线程在没有获取到锁时，进入EntryList，同时发生上下文切换，进入阻塞，影响性能

# 3. cpu核心
- CAS时，线程一直在运行，如果cpu不够多且当前时间片用完，虽然不会进入阻塞，但依然会发生上下文切换，从而进入可运行状态
- 最好是线程数少于cpu的核心数目

# 4. 乐观锁适应场景
- 如果竞争激烈，重试机制必然频发触发，反而性能会收到影响
```

## 原子整数

- 能够保证修改数据的结果，是线程安全的包装类型

```bash
# 功能类似
java.util.concurrent.atomic.AtomicInteger
java.util.concurrent.atomic.AtomicLong
java.util.concurrent.atomic.AtomicBoolean
```

### 常用方法

- AtomicInteger的下面方法，都是原子性的，利用了CAS思想，简化代码

```bash
# 1. 自增，自减等操作： 可以基于compareAndSet来实现
# 自增并返回最新结果: 最新结果可能不一定是自增后的值，比如方法执行前值是2，自增只保证增加了1，但是最新结果可能是14
public final int incrementAndGet();
# 获取最新结果并自增: 
public final int getAndIncrement();

public final int decrementAndGet();
public final int getAndDecrement();
public final int getAndAdd(int delta);
public final int addAndGet(int delta);
public final int getAndSet(int newValue);

# 2. 复杂操作，自定义操作： 自定义实现cas，要配合while(true)来使用
public final boolean compareAndSet(int expect, int update);

# 3. 自定义操作的函数式接口，也是基于compareAndSet实现
public final int updateAndGet(IntUnaryOperator updateFunction);
public final int getAndUpdate(IntUnaryOperator updateFunction);
```

### 自增/自减

- 如果没有判读条件，那么多个线程访问同一个资源，也是线程安全的

```java
package com.erick.multithread.d04;

import lombok.extern.slf4j.Slf4j;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;

@Slf4j
public class Demo01 {
    public static void main(String[] args) throws InterruptedException {
        List<Thread> threads = new ArrayList<>();
        BankService service = new BankService();
        for (int i = 0; i < 1000000; i++) {
            Thread thread = new Thread(() -> service.drawMoneyFirst());
            thread.start();
            threads.add(thread);
        }

        for (Thread thread : threads) {
            thread.join();
        }
        log.info("money={}", service.total.get()); // 结果=0
    }
}

@Slf4j
class BankService {

    /**
     * 内部维护了一个private volatile int value;
     * 无参构造为0
     */
    public AtomicInteger total = new AtomicInteger(1000000);

    /*每次取一块钱*/
    public void drawMoneyFirst() {
        total.getAndDecrement();
    }
}
```

### 自定义操作

- 如果要控制超额支出问题，必须自定义CAS的while true

#### 普通运算

- 手写while true循环

```java
@Slf4j
class BankService {

    public AtomicInteger total = new AtomicInteger(50);

    /*每次取指定钱，带条件判断*/
    public void drawMoneyFirst(int bucks) {
        while (true) {
            int value = total.get();
            if (value < bucks) {
                log.info("no money left");
                break;
            }

            Sleep.sleep(1);

            boolean result = total.compareAndSet(value, value - bucks);
            if (result) {
                break;
            }
            log.info("CAS failure");
        }
    }
}
```

#### 函数式接口

```java
@Slf4j
class BankService {

    public AtomicInteger total = new AtomicInteger(50);

    public void drawMoneyFirst() {
        process(total, new Calculate());
    }

    private void process(AtomicInteger count, IntUnaryOperator operator) {
        while (true) {
            int currentValue = count.get();
            if (currentValue <= 0) {
                log.info("zero money");
                break;
            }
            int afterValue = operator.applyAsInt(currentValue);
            if (afterValue <= 0) {
                log.info("not enough money");
                break;
            }
            if (count.compareAndSet(currentValue, afterValue)) {
                log.info("success");
                break;
            }
            log.info("CAS failure");
        }
    }
}

class Calculate implements IntUnaryOperator {

    @Override
    public int applyAsInt(int operand) {
        /*可能涉及到复杂运算*/
        int afterValue = operand - 6;
        return afterValue;
    }
}
```

## 原子引用

- 基本数据类型满足不了的一些其他计算场景，就要用到原子引用
- 用法和上面原子整数类似

```bash
# 比如大数BigDecimal， 商业计算

# 比如其他引用类型，如String
```

### AtomicReference

```java
@Slf4j
class AccountService {
    public AtomicReference<BigDecimal> account = new AtomicReference<>(new BigDecimal("100"));

    public void drawMoney(BigDecimal bucks) {
        while (true) {
            BigDecimal prev = account.get();
            if (prev.compareTo(new BigDecimal("0")) <= 0) {
                log.info("no money left");
                break;
            }
            BigDecimal next = prev.subtract(bucks);
            if (next.compareTo(new BigDecimal(0)) <= 0) {
                log.info("not enough money");
                break;
            }
            if (account.compareAndSet(prev, next)) {
                log.info("success");
                break;
            }
            log.info("cas failure");
        }
    }
}
```

### AtomicStampedReference

#### ABA场景

- 线程-1在CAS中要将A->B，线程-2在此期间进行A->B->A
- 线程-1的CAS会成功，但是对比的值，其实已经被改过
- 一般情况下，ABA并不会影响具体的业务

```java
package com.erick.multithread.d04;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.atomic.AtomicReference;

@Slf4j
public class Demo03 {
    public static void main(String[] args) throws InterruptedException {
        JackService jackService = new JackService();
        Thread firstThread = new Thread(() -> jackService.job());
        Thread secondThread = new Thread(() -> jackService.work());
        firstThread.start();
        secondThread.start();
        firstThread.join();
        secondThread.join();
        log.info(jackService.ref.get());
    }
}

@Slf4j
class JackService {
    public AtomicReference<String> ref = new AtomicReference<>("A");

    // 第一次就会交换成功
    public void job() {
        while (true) {
            String prev = ref.get();
            Sleep.sleep(1);
            if (ref.compareAndSet(prev, "B")) {
                log.info("success");
                break;
            }
            log.info("cas failure");
        }
    }

    public void work() {
        ref.compareAndSet("A", "B");
        ref.compareAndSet("B", "A");
    }
}
```

#### ABA解决

- 具体的值和版本号(版本号或者时间戳)
- 只要其他线程动过了共享变量(通过值和版本号)，就算cas失败
- 可以通过版本号，得到该值前前后后被改动了多少次

```java
@Slf4j
class JackService {
    /*初试版本号可以设置为0 */
    public AtomicStampedReference<String> ref = new AtomicStampedReference<>("A", 0);

    // 第一次会交换成功
    public void job() {
        while (true) {

            int prevStamp = ref.getStamp();
            String prevValue = ref.getReference();
            Sleep.sleep(1);
            boolean result = ref.compareAndSet(prevValue, "B", prevStamp, prevStamp + 1);
            if (result) {
                log.info("success");
                break;
            }
            log.info("cas failure");
        }
    }

    public void work() {
        ref.compareAndSet("A", "B", 0, 1);
        ref.compareAndSet("B", "A", 1, 2);
    }
}
```

### AtomicMarkableReference

- 线程a在执行CAS操作时，其他线程反复修改数据，但是a线程只关心最终的结果是否变化了
- ABABA场景

```java
@Slf4j
class JackService {

    private boolean initialFlag = true;
    public AtomicMarkableReference<String> ref = new AtomicMarkableReference<>("A", true);

    public void job() {
        while (true) {
            String prevValue = ref.getReference();

            Sleep.sleep(1);
            boolean result = ref.compareAndSet(prevValue, "B", true, false);
            if (result) {
                log.info("success");
                break;
            }
            log.info("cas failure");
        }
    }

    public void work() {
        ref.compareAndSet("A", "B", true, false);
        ref.compareAndSet("B", "A", false, true);
        ref.compareAndSet("A", "B", true, false);
        // ref.compareAndSet("B", "A", false, true);
    }
}
```

## 原子数组

- 上述原子整数和原子引用，只是针对一个对象的
- 原子数组，可以存放上面的数组

### 不安全

- 可以将数组中的每个元素单独拿出来，通过原子整数来解决

```java
package com.erick.multithread.d04;

import lombok.extern.slf4j.Slf4j;

import java.util.ArrayList;
import java.util.List;

public class Demo04 {
    public static void main(String[] args) throws InterruptedException {
        BankingService service = new BankingService();
        List<Thread> threads = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(() -> service.change());
            thread.start();
            threads.add(thread);
        }

        for (Thread thread : threads) {
            thread.join();
        }

        service.log();
    }
}

@Slf4j
class BankingService {
    private int[] accounts = new int[2];

    public void change() {
        for (int i = 0; i < 1000; i++) {
            accounts[0] = accounts[0] - 1;
            accounts[1] = accounts[1] + 1;
        }
    }

    public void log() {
        log.info("arr-0={}", accounts[0]);
        log.info("arr-1={}", accounts[1]);
    }
}
```

### 原子数组

- AtomicIntegerArray
- AtomicLongArray
- AtomicReferenceArray

```java
@Slf4j
class BankingService {
    private AtomicIntegerArray accounts = new AtomicIntegerArray(2);

    public void change() {
        for (int i = 0; i < 1000; i++) {
            /*参数一： 索引
             * 参数二： 改变的值
              也可以自定义while true循环
             */
            accounts.addAndGet(0, 2);
            accounts.addAndGet(1, -2);
        }
    }

    public void log() {
        log.info("arr-0={}", accounts.get(0));
        log.info("arr-1={}", accounts.get(1));
    }
}
```

## 原子Filed更新器

- 用来原子更新对象中的字段，该字段必须和volatile结合使用

```bash
- AtomicReferenceFieldUpdater      # 引用类型字段
- AtomicLongFieldUpdater           # Long类型字段
- AtomicIntegerFieldUpdater        # Integer类型字段
```

```java
package com.erick.multithread.d04;

import com.erick.multithread.Sleep;
import lombok.Getter;
import lombok.Setter;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.atomic.AtomicReferenceFieldUpdater;

@Slf4j
public class Demo05 {
    public static void main(String[] args) throws InterruptedException {
        Student student = new Student();
        student.setAge(11);
        student.setName("lucy");

        StudentService service = new StudentService();
        Thread firstThread = new Thread(() -> service.updateName(student));
        firstThread.start();

        Sleep.sleep(1);

        Thread secondThread = new Thread(() -> student.setName("nancy"));
        secondThread.start();

        firstThread.join();
        secondThread.join();

        log.info(student.name);
    }
}

@Slf4j
class StudentService {
    private AtomicReferenceFieldUpdater<Student, String> nameField =
            AtomicReferenceFieldUpdater.newUpdater(Student.class, String.class, "name");

    public void updateName(Student student) {
        while (true) {
            /*从lucy改成erick*/
            String prev = nameField.get(student);
            String next = "erick";

            Sleep.sleep(2);

            boolean result = nameField.compareAndSet(student, prev, next);
            if (result) {
                log.info("success");
                break;
            }
            log.info("cas failure");
        }
    }
}

@Setter
@Getter
class Student {
    private int age;
    /*是基于反射的，字段必须是public volatile*/
    public volatile String name;
}
```

## 原子累加器

- JDK 8 以后提供了专门的做累加的类，用来提高性能
- 任务拆分的理念

```bash
# 原理： 在有竞争的时候，设置多个累加单元， Thread-0 累加 Cell[0], Thread-1累加Cell[1]
#       累加结束后，将结果进行汇总，这样他们在累加时操作的不同的Cell变量，因此减少了CAS重试失败，从而提高性能
- LongAdder(long的累加)        ----          AtomicLong(原始long)
- DoubleAdder
```

```java
package com.erick.multithread.d04;

import lombok.Getter;
import lombok.extern.slf4j.Slf4j;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicLong;
import java.util.concurrent.atomic.LongAdder;

@Slf4j
public class Demo06 {
    public static void main(String[] args) throws InterruptedException {
        AdderService adderService = new AdderService();
        adderService.firstMethod();
        adderService.secondMethod();
        log.info("{}", adderService.getFirstLong().get());        // 20000000
        log.info("{}", adderService.getSecondLong().longValue()); // 20000000
    }
}

@Slf4j
class AdderService {
    @Getter
    private AtomicLong firstLong = new AtomicLong(0L);
    @Getter
    private LongAdder secondLong = new LongAdder();

    public void firstMethod() throws InterruptedException {
        long start = System.currentTimeMillis();
        List<Thread> threads = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(() -> {
                for (int j = 0; j < 1000000; j++) {
                    firstLong.addAndGet(2);
                }
            });
            thread.start();
            threads.add(thread);
        }

        for (Thread thread : threads) {
            thread.join();
        }
        log.info("AtomicLong={}", (System.currentTimeMillis() - start)); // 507ms
    }

    public void secondMethod() throws InterruptedException {
        long start = System.currentTimeMillis();
        List<Thread> threads = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(() -> {
                for (int j = 0; j < 1000000; j++) {
                    secondLong.add(2);
                }
            });
            thread.start();
            threads.add(thread);
        }

        for (Thread thread : threads) {
            thread.join();
        }
        log.info("LongAdder={}", (System.currentTimeMillis() - start)); // 46ms
    }
}
```

# 不可变类

## 日期类

### SimpleDateFormat

#### 线程不安全

```java
package com.erick.multithread.d04;

import lombok.extern.slf4j.Slf4j;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

@Slf4j
public class Demo07 {
    public static void main(String[] args) {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    Date parse = sdf.parse("2022-03-12");
                    log.info(parse.toString()); // 最终结果不一致，或者出现异常
                } catch (ParseException e) {
                    throw new RuntimeException(e);
                }
            }).start();
        }
    }
}
```

#### 加锁解决

- 性能会受到影响

```java
package com.erick.multithread.d04;

import lombok.extern.slf4j.Slf4j;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

@Slf4j
public class Demo07 {
    private static Object lock = new Object();

    public static void main(String[] args) {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    synchronized (lock) {
                        Date parse = sdf.parse("2022-03-12");
                        log.info(parse.toString());
                    }
                } catch (ParseException e) {
                    throw new RuntimeException(e);
                }
            }).start();
        }
    }
}
```

### DateTimeFormatter

- JDK8之后提供了线程安全的类

```java
package com.erick.multithread.d04;

import lombok.extern.slf4j.Slf4j;

import java.time.format.DateTimeFormatter;
import java.time.temporal.TemporalAccessor;

@Slf4j
public class Demo07 {
    private static Object lock = new Object();

    public static void main(String[] args) {
        DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                TemporalAccessor parse = dtf.parse("2022-09-12");
                log.info(parse.toString());
            }).start();
        }
    }
}
```

## 不可变类

- 不可变类是线程安全的
- 类中所有成员变量都是final修饰，保证不可变，保证只能读不能写
- 类是final修饰，不会因为错误的继承来重写方法，导致了可变

```bash
# String类型： 不可变类
- 里面所有的Field都是final修饰的，保证了不可变，不可被修改：  private final byte[] value;
- 类被final修饰，保证了String类不会被继承

# 数组保护性拷贝： 如果String的构造方法是传递了一个数组，实际上是内部创建了一个新的数组
- 数组类型也是final修饰，如果通过构造传递，实际上是创建了新的数组和对应的String [保护性拷贝]
```

## 享元模式

- 对于不可变类，可能需要系统频繁的创建和销毁，对于同一个对象，可以使用享元模式来进行获取和使用

### 包装类

- Long: 维护了一个静态内部类LongCache, 将一些Long对象进行缓存
- 热点数据，提前预热

```java
// 获取的时候，先从对应的Cache中去找，然后才会创建新的对象
public static Long valueOf(long l) {
    final int offset = 128;
    if (l >= -128 && l <= 127) { // will cache
        return LongCache.cache[(int)l + offset];
    }
    return new Long(l);
}


private static class LongCache {
    private LongCache(){}

    static final Long cache[] = new Long[-(-128) + 127 + 1];

    static {
        for(int i = 0; i < cache.length; i++)
            cache[i] = new Long(i - 128);
    }
}
```

```bash
# 缓存范围
Byte, Short, Long:   -128～127
Character:           0~127
Integer:             -128~127
Boolean:             TRUE和FALSE
```

### 自定义连接池

```java
package com.erick.multithread.d07;

import lombok.Data;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicIntegerArray;

public class Demo06 {
    public static void main(String[] args) {
        ConnectionPool pool = new ConnectionPool(2);
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                ErickConnection connection = pool.getConnection();
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                pool.release(connection);
            }).start();
        }
    }
}

class ConnectionPool {
    private int pooSize;

    private ErickConnection[] pool;

    /* 1: 当前Connection正在使用
     * 0: 当前Connection空闲*/
    private AtomicIntegerArray states;

    public ConnectionPool(int pooSize) {
        this.pooSize = pooSize;
        pool = new ErickConnection[pooSize];
        states = new AtomicIntegerArray(pooSize);
        for (int i = 0; i < pooSize; i++) {
            pool[i] = new ErickConnection();
        }
    }

    public ErickConnection getConnection() {
        while (true) {
            for (int i = 0; i < pooSize; i++) {
                if (states.get(i) == 0) {
                    states.compareAndSet(i, 0, 1);
                    System.out.println(Thread.currentThread().getName() + "获取连接");
                    return pool[i];
                }
            }
            /*如果没有空闲连接，当前线程进入等待: 如果不加下面，就会造成CPU资源*/
            System.out.println(Thread.currentThread().getName() + "等待连接");
            synchronized (this) {
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }

    public void release(ErickConnection erickConnection) {
        for (int i = 0; i < pooSize; i++) {
            if (pool[i] == erickConnection) {
                states.set(i, 0);
                System.out.println(Thread.currentThread().getName() + "释放连接");
                synchronized (this) {
                    this.notifyAll();
                }
                break;
            }
        }
    }

}

@Data
class ErickConnection {
    private String username;
    private String password;
}
```

# 线程池

## 基本介绍

### 引入原因

```bash
1. 一个任务过来，一个线程去做。频繁创建新线程，性能低且比较耗费内存
2. 线程数 > cpu核心: 线程切换，要保存原来线程的状态，运行现在的线程，势必会更加耗费资源
   线程数 < cpu核心: 不能很好的利用cpu性能
   
3. 充分利用已有线程，去处理原来的任务
```

### 线程池组件

```bash
消费者(线程池)：                 保存一定数量线程来处理任务
生产者：                        客户端源源不断产生的新任务
阻塞队列(blocking queue)：      平衡消费者和生产者之间，用来保存任务(线程任务) 的一个等待队列

- 生产者：生产任务速度较快，多余的任务要等
- 消费者：消费任务快，那么线程池中存活的线程等
```


![image-20250428104423496](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250428104423496.png)

## 自定义线程池

```bash
# 生产者：拒绝策略
- 当核心线程已经达到最大值，那么生产者产生的新的任务

- 让生产者自己决定该如何执行当前任务
       - 不带超时，加入到阻塞队列
       - 带超时等待，加入到阻塞队列
       - 让生产者放弃执行任务
       - 让生产者抛出异常
       - 让生产者自己执行任务
       
# 消费者超时：
- 线程池的核心线程消费，从阻塞队列中拉取时，超时后则kill当前核心线程
```

### 阻塞队列

```java
package com.erick.multithread.poll;

import lombok.extern.slf4j.Slf4j;

import java.util.LinkedList;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

@Slf4j
public class BlockingQueue {
    private int capacity; // 阻塞队列大小：有界队列
    private LinkedList<Runnable> queue = new LinkedList<>(); // 保存任务
    private ReentrantLock lock = new ReentrantLock();
    private Condition producerRoom = lock.newCondition(); // 生产者等待
    private Condition consumerRoom = lock.newCondition();  // 线程池消费者等待

    public BlockingQueue(int capacity) {
        this.capacity = capacity;
    }

    /*offer: 添加任务的方法，不会和线程池直接作用，而是通过rejectPolicy交互*/
    public boolean offer(Runnable task) {
        try {
            lock.lock();
            while (queue.size() == capacity) {
                try {
                    producerRoom.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }

            queue.addFirst(task);
            consumerRoom.signalAll();
            return false;
        } finally {
            lock.unlock();
        }
    }

    public boolean offer(Runnable task, long timeout, TimeUnit timeUnit) {
        if (timeout <= 0 || timeUnit == null) {
            return offer(task);
        }

        try {
            lock.lock();
            long nanos = timeUnit.toNanos(timeout); // 时间转换
            while (queue.size() == capacity) {
                if (nanos <= 0) {
                    return false;
                }
                try {
                    nanos = producerRoom.awaitNanos(nanos); // 等待并且重新赋值
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }

            queue.addFirst(task);
            consumerRoom.signalAll();
            return true;
        } finally {
            lock.unlock();
        }
    }

    /*线程池消费者： 从阻塞队列中获取任务，一直死等*/
    public Runnable poll() {
        try {
            lock.lock();
            while (queue.isEmpty()) {
                try {
                    consumerRoom.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            Runnable task = queue.removeLast();
            producerRoom.signalAll();
            return task;
        } finally {
            lock.unlock();
        }
    }

    /*线程池消费者： 从阻塞队列中获取任务，超时则返回null, 如果获取到的是null,则kill线程池的线程并从池中释放*/
    public Runnable poll(long timeout, TimeUnit timeUnit) {
        if (timeout <= 0 || timeUnit == null) {
            return poll();
        }

        try {
            lock.lock();
            long nanos = timeUnit.toNanos(timeout);
            while (queue.isEmpty()) {
                if (nanos <= 0) {
                    return null;
                }
                try {
                    nanos = consumerRoom.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }

            Runnable task = queue.removeLast();
            producerRoom.signalAll();
            return task;
        } finally {
            lock.unlock();
        }
    }

    public int getSize() {
        try {
            lock.lock();
            return queue.size();
        } finally {
            lock.unlock();
        }
    }
}
```

### 线程池

```java
package com.erick.multithread.poll;

import lombok.extern.slf4j.Slf4j;

import java.util.HashSet;
import java.util.Set;
import java.util.concurrent.TimeUnit;

@Slf4j
public class ThreadPool {
    private int capacity; // 阻塞队列大小
    private BlockingQueue blockingQueue;
    private int coreThreadSize; // 线程池核心线程数大小
    private long timeout; // 消费任务时，超时时间, 超时则kill掉当前线程池的当前线程
    private TimeUnit timeUnit;
    private RejectPolicy rejectPolicy; // 拒绝策略

    /*线程池中保存核心线程的*/
    private final Set<Worker> threadPool = new HashSet<>();

    public ThreadPool(int capacity, int coreThreadSize, long timeout, TimeUnit timeUnit, RejectPolicy rejectPolicy) {
        blockingQueue = new BlockingQueue(capacity);
        this.coreThreadSize = coreThreadSize;
        this.timeout = timeout;
        this.timeUnit = timeUnit;
        this.rejectPolicy = rejectPolicy;
    }

    /**
     * 任务执行：
     * 如果核心线程数<池子中线程数量，
     * 1. 创建新的工作线程
     * 2. 开启工作线程(执行当前task，执行完毕后去阻塞队列中获取)
     * 3. 当前工作线程加入到线程池中
     * 如果核心线程数=池子中线程数量，则将当前任务添加到阻塞队列中
     */
    public synchronized void executeTask(Runnable task) {
        if (threadPool.size() >= coreThreadSize) {
            /*根据一定的拒绝策略，会将任务加在阻塞队列中*/
            log.info("maximum core thread");
            rejectPolicy.reject(task, blockingQueue);
        } else {
            Worker worker = new Worker(task);
            log.info("create new thread={}", worker);
            threadPool.add(worker); // 加入到线程池中
            worker.start(); // 开始运行线程
        }
    }

    class Worker extends Thread {
        private Runnable task;

        public Worker(Runnable task) {
            this.task = task;
        }

        @Override
        public void run() {
            while (task != null) {
                task.run(); // 执行完当前任务
                task = blockingQueue.poll(timeout, timeUnit); // 继续从阻塞队列中去取
            }

            /*执行完毕后，将该线程从线程池中移除*/
            synchronized (threadPool) {
                System.out.println("thread-destroyed");
                threadPool.remove(this);
            }
        }
    }
}
```

### 拒绝策略

```java
package com.erick.multithread.poll;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

public interface RejectPolicy {
    void reject(Runnable task, BlockingQueue blockingQueue);
}

/*生产者死等：一定要把任务添加到阻塞队列中*/
class WaitForeverAddToQueuePolicy implements RejectPolicy {

    @Override
    public void reject(Runnable task, BlockingQueue blockingQueue) {
        blockingQueue.offer(task);
    }
}

/*生产者超时等待：等待把任务添加到阻塞队列中，超时则放弃该任务*/
class WaitWithTimeoutAddToQueuePolicy implements RejectPolicy {

    private TimeUnit timeUnit;
    private long timeout;

    @Override
    public void reject(Runnable task, BlockingQueue blockingQueue) {
        blockingQueue.offer(task, timeout, timeUnit);
    }
}

/*生产者放弃任务*/
@Slf4j
class AbortPolicy implements RejectPolicy {

    @Override
    public void reject(Runnable task, BlockingQueue blockingQueue) {
        log.info("abort the task");
    }
}

/*生产者自己执行任务*/
class ProducerExecutePolicy implements RejectPolicy {

    @Override
    public void reject(Runnable task, BlockingQueue blockingQueue) {
        new Thread(task).start();
    }
}

/*生产者异常:后续其他任务不能再进来了*/
class ProducerExceptionPolicy implements RejectPolicy {

    @Override
    public void reject(Runnable task, BlockingQueue blockingQueue) {
        throw new RuntimeException("core thread is working, current task can not work");
    }
}
```

### 测试代码

```java
package com.erick.multithread.poll;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

@Slf4j
public class Test {
    public static void main(String[] args) {
        ThreadPool pool =
                new ThreadPool(5, 3, 4, TimeUnit.SECONDS, new WaitForeverAddToQueuePolicy());

        for (int i = 0; i < 10; i++) {
            int j = i;
            pool.executeTask(() -> {
                Sleep.sleep(2);
                log.info("{}---{}号---任务被执行", Thread.currentThread().getName(), j);
            });
        }
    }
}
```

## JDK线程池

### 类图

![image-20250428104649151](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250428104649151.png)

### 线程状态

- ThreadPoolExecutor 使用int的高3位来表示线程池状态，低29位表示线程数量

| 状态       | 高3位 | 接受新任务 | 处理阻塞任务        | 说明                                |
| ---------- | ----- | ---------- | ------------------- | ----------------------------------- |
| RUNNING    | 111   | Y          | Y                   |                                     |
| SHUTDOWN   | 000   | N          | Y                   |                                     |
| STOP       | 001   | N          | N(抛弃阻塞队列任务) |                                     |
| TIDYING    | 010   | -          | -                   | 任务执行完毕，活动线程为0，即将终结 |
| TERMINATED | 011   | -          | -                   | 终结状态                            |

### 线程数量

- 过小：CPU不能充分利用
- 过大：线程上下文切换浪费性能，每个线程也要占用内存导致占用内存过多

#### CPU密集型

- 线程的任务主要是和CPU打交道，比如大数据运算
- 线程数量： 核心数+1
- +1: 保证某线程由于某些原因(操作系统方面)导致暂停时，额外线程可以启动，不浪费CPU资源

#### IO密集型

- IO操作，RPC调用，数据库访问时，CPU是空闲的，称为IO密集型
- 更加常见： IO操作，远程RPC调用，数据库操作
- 线程数 = 核数 * 期望cpu利用率 *  (CPU计算时间 + CPU等待时间) / CPU 计算时间

![image-20221018104629282](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20221018104629282.png)

## ThreadPoolExecutor

- 常见的线程池

### 构造方法

```java
int corePoolSize:                     // 核心线程数
int maximumPoolSize：                 // 最大线程数
long keepAliveTime：                  // 救急线程数执行任务完后存活时间
TimeUnit unit：                       
BlockingQueue<Runnable> workQueue：   // 阻塞队列: 有界或者无界
ThreadFactory threadFactory：         // 线程生产工厂，为线程起名字
RejectedExecutionHandler handler：    // 拒绝策略 

public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler){
}
```

### 核心线程和救急线程

```bash
# 核心线程
- 执行完任务后，会继续保留在线程池中

# 救急线程：
- 如果阻塞队列已满，并且没有空余的核心线程。那么会创建救急线程来执行任务
- 任务执行完毕后，这个线程就会被销毁(临时工)
- 必须是有界阻塞，如果是无界队列，则不需要创建救急线程
```

### 拒绝策略

- 核心线程满负荷，阻塞队列(有界队列)已满，无空余救急线程，才会执行拒绝策略，JDK 提供了4种拒绝策略

```bash
- RejectedExecutionHandler     # 拒绝策略接口

- AbortPolicy:                 # 调用者抛出RejectedExecutionException,  默认策略
- CallerRunsPolicy:            # 调用者运行任务
- DiscardPolicy:               # 放弃本次任务
- DiscardOldestPolicy:         # 放弃阻塞队列中最早的任务，本任务取而代之

# 第三方框架的技术实现
- Dubbo: 在抛出异常之前，记录日志，并dump线程栈信息，方便定位问题
- Netty: 创建一个新的线程来执行任务
- ActiveMQ: 带超时等待(60s)， 尝试放入阻塞队列
```

![image-20250428105009099](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250428105009099.png)

## Executors

- 默认的构造方法来创建线程池，参数过多，JDK提供了工厂方法，来创建线程池

### newFixedThreadPool

- 固定大小
- 核心线程数 = 最大线程数，救急线程数为0
- 阻塞队列：无界，可以存放任意数量的任务

```bash
# 应用场景
任务量已知，但是线程执行时间较长
执行任务后，线程并不会结束
```

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

```java
package com.erick.multithread.d05;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;

@Slf4j
public class Demo01 {
    public static void main(String[] args) {
        ThreadPoolExecutor pool = (ThreadPoolExecutor) Executors.newFixedThreadPool(3);

        for (int i = 0; i < 5; i++) {
            pool.execute(() -> {
                Sleep.sleep(3);
                log.info("{} execute task", Thread.currentThread().getName());
            });
        }
    }
}
```

### newCachedThreadPool

- SynchronousQueue: 没有容量，没有线程来取的时候是放不进去的
- 整个线程池数会随着任务数目增长，1分钟后没有其他活动会消亡

```bash
# 应用场景
1. 时间较短的线程
2. 任务数量大，任务执行时间长，会造成  OutOfMmeory问题
```

```java
// 1.核心线程数为0, 最大线程数为Integer的无限大
// 2. 全部是救急线程，等待时间是60s，60s后就会消亡
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

```java
package com.erick.multithread.d05;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;

@Slf4j
public class Demo01 {
    public static void main(String[] args) {
        ThreadPoolExecutor pool = (ThreadPoolExecutor) Executors.newCachedThreadPool();
         
        // 任务执行完成后60s，线程死亡 
        for (int i = 0; i < 5; i++) {
            pool.execute(() -> {
                Sleep.sleep(5);
                log.info("{} execute task", Thread.currentThread().getName());
            });
        }
    }
}
```

### newSingleThreadExecutor

- 线程池大小始终为1个，不能改变线程数
- 相比自定义一个线程来执行，线程池可以保证前面任务的失败，不会影响到后续任务
- 线程不会死亡

```bash
# 1. 和自定义线程的区别
自定义线程：  执行多个任务时，一个出错，后续都能不能执行了
单线程池：    一个任务失败后，会结束出错线程。重新new一个线程来执行下面的任务

# 2. 执行顺序
单线程池： 保证所有任务都是串行

# 3. 和newFixedThreadPool的区别
newFixedThreadPool:          初始化后，还可以修改线程大小
newSingleThreadExecutor:     不可以修改
```

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

```java
package com.erick.multithread.d05;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

@Slf4j
public class Demo01 {
    static ExecutorService pool = Executors.newSingleThreadExecutor();

    public static void main(String[] args) {
        method2();
    }

    public static void method1() {
        for (int i = 0; i < 3; i++) {
            pool.execute(() -> {
                Sleep.sleep(1);
                // 每次都是创建一个新的线程
                log.info("{} execute task", Thread.currentThread().getName());
                int num = 1 / 0;
            });
        }
    }

    /*同一个线程执行：任务串行*/
    public static void method2() {
        ExecutorService pool = Executors.newSingleThreadExecutor();
        for (int i = 0; i < 3; i++) {
            pool.execute(() -> {
                Sleep.sleep(1);
                log.info("{} execute task", Thread.currentThread().getName());
            });
        }
    }
}
```

### newScheduledThreadPool

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

#### 延时执行

```java
package com.erick.multithread.d05;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

@Slf4j
public class Demo01 {

    public static void main(String[] args) {
        log.info("coming");
        method3();
    }

    /*2个核心线程：分别延迟1s和3s后执行*/
    private static void method1() {
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(2);

        scheduledExecutorService.schedule(() -> log.info("{} execute task", Thread.currentThread().getName()),
                1, TimeUnit.SECONDS);

        scheduledExecutorService.schedule(() -> log.info("{} execute task", Thread.currentThread().getName()),
                3, TimeUnit.SECONDS);
    }

    /*1个核心线程，串行*/
    private static void method2() {
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(1);

        scheduledExecutorService.schedule(() -> log.info("{} execute task", Thread.currentThread().getName()),
                1, TimeUnit.SECONDS);

        scheduledExecutorService.schedule(() -> log.info("{} execute task", Thread.currentThread().getName()),
                3, TimeUnit.SECONDS);
    }

    /*1个核心线程：其中一个出错，会开启一个新的核心线程来处理*/
    private static void method3() {
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(1);

        scheduledExecutorService.schedule(() -> log.info("{} execute task", Thread.currentThread().getName()),
                1, TimeUnit.SECONDS);

        scheduledExecutorService.schedule(() -> {
            int a = 1 / 0;
        }, 1, TimeUnit.SECONDS);

        scheduledExecutorService.schedule(() -> log.info("{} execute task", Thread.currentThread().getName()),
                3, TimeUnit.SECONDS);
    }
}
```

#### 定时执行

```bash
# scheduleAtFixedRate
- 如果任务的执行时间大于时间间隔，就会紧接着立刻执行

public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                              long initialDelay,
                                              long period,
                                              TimeUnit unit);

# scheduleWithFixedDelay
- 上一个任务执行完毕后，再延迟一定的时间才会执行

public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                 long initialDelay,
                                                 long delay,
                                                 TimeUnit unit);
```

```java
package com.dreamer.multithread.day09;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class Demo07 {
    public static void main(String[] args) {
        ScheduledExecutorService pool = Executors.newScheduledThreadPool(2);

        // 定时执行任务
        pool.scheduleWithFixedDelay(new Runnable() {
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("task is running");
            }
        }, 3, 2, TimeUnit.SECONDS);
        //  初始延时，   任务间隔时间，    任务间隔时间单位
    }
}
```

## 提交任务

```bash
# 1. execute
void execute(Runnable command);

# 2. submit: 可以从 Future 对象中获取一些执行任务的最终结果
Future<?> submit(Runnable task);
<T> Future<T> submit(Runnable task, T result);
<T> Future<T> submit(Callable<T> task);

# 3. invokeAll: 执行集合中的所有的任务
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
    throws InterruptedException;
    
# 4. invokeAny: 集合中之要有一个任务执行完毕，其他任务就不再执行
 <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
```

```java
package com.nike.erick.d07;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;

public class Demo02 {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService pool = Executors.newFixedThreadPool(10);
        method05(pool);
    }

    /* void execute(Runnable command) */
    public static void method01(ExecutorService pool) {
        pool.execute(() -> System.out.println(Thread.currentThread().getName() + " running"));
    }

    /*  <T> Future<T> submit(Runnable task, T result)
     * Future<?> submit(Runnable task) */
    public static void method02(ExecutorService pool) throws InterruptedException {
        Future<?> result = pool.submit(new Thread(() -> System.out.println(Thread.currentThread().getName() + " running")));
        TimeUnit.SECONDS.sleep(1);
        System.out.println(result.isDone());
        System.out.println(result.isCancelled());
    }

    /*
     * <T> Future<T> submit(Callable<T> task)*/
    public static void method03(ExecutorService pool) throws InterruptedException, ExecutionException {
        Future<String> submit = pool.submit(() -> "success");
        TimeUnit.SECONDS.sleep(1);
        System.out.println(submit.isDone());
        System.out.println(submit.isCancelled());
        System.out.println(submit.get()); // 返回结果是success
    }

    /* <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;*/
    public static void method04(ExecutorService pool) throws InterruptedException {
        Collection tasks = new ArrayList();
        for (int i = 0; i < 10; i++) {
            int round = i;
            tasks.add((Callable) () -> {
                System.out.println(Thread.currentThread().getName() + " running");
                return "success:" + round;
            });
        }
        List results = pool.invokeAll(tasks);

        TimeUnit.SECONDS.sleep(1);
        System.out.println(results);
    }

    /*
     *     <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;
     * */
    public static void method05(ExecutorService pool) throws InterruptedException, ExecutionException {
        ExecutorService service = Executors.newFixedThreadPool(1);
        Collection<Callable<String>> tasks = new ArrayList<>();

        tasks.add(() -> {
            System.out.println("first task");
            TimeUnit.SECONDS.sleep(1);
            return "success";
        });

        tasks.add(() -> {
            System.out.println("second task");
            TimeUnit.SECONDS.sleep(2);
            return "success";
        });


        tasks.add(() -> {
            System.out.println("third task");
            TimeUnit.SECONDS.sleep(3);
            return "success";
        });
        // 任何一个任务执行完后，就会返回结果
        String result = pool.invokeAny(tasks);
        System.out.println(result);
    }
}
```

## 异常处理

```bash
# 不处理
- 任务执行过程中，业务中的异常并不会抛出

# 任务执行者处理

# 线程池处理
```

```java
package com.erick.multithread.d05;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

@Slf4j
public class Demo01 {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        log.info("coming");
        method04();
    }

    /*会抛出异常，终止原来的线程，开启新的线程*/
    public static void method01() {
        ExecutorService pool = Executors.newFixedThreadPool(1);
        pool.execute(() -> {
            log.info("{} executing", Thread.currentThread().getName());
            int i = 1 / 0;
        });

        pool.execute(() -> {
            log.info("{} executing", Thread.currentThread().getName());
        });
    }

    /*不处理异常*/
    public static void method02() {
        ExecutorService pool = Executors.newFixedThreadPool(1);
        pool.submit(() -> {
            int i = 1 / 0;
        });
    }

    /*调用者自己处理*/
    public static void method03() {
        ExecutorService pool = Executors.newFixedThreadPool(1);
        pool.submit(() -> {
            try {
                int i = 1 / 0;
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
    }

    /*线程池处理*/
    public static void method04() throws ExecutionException, InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(1);
        Future<?> result = pool.submit(() -> {
            int i = 1 / 0;
        });
        Sleep.sleep(1);
        // 获取结果的时候，就可以把线程执行任务过程中的异常报出来
        log.info("{}", result.get());
    }
}
```

## 关闭线程池

```bash
# shutdown:      void shutdown();
- 将线程池的状态改变为SHUTDOWN状态
- 不会接受新任务，已经提交的任务不会停止
- 不会阻塞调用线程的执行

# shutdownNow:    List<Runnable> shutdownNow();
-  不会接受新任务
-  没执行的任务会打断
-  将等待队列中的任务返回
```

```java
package com.erick.multithread.d02;

import com.erick.multithread.util.Sleep;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Demo01 {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(1);
        pool.execute(() -> {
            System.out.println("first");
            Sleep.sleep(2);
        });

        pool.execute(() -> {
            System.out.println("second");
            Sleep.sleep(2);
        });

        pool.execute(() -> {
            System.out.println("third");
            Sleep.sleep(2);
        });
        pool.shutdown();//任务都会执行完毕
        System.out.println("main ending"); // 不阻塞调用线程的执行
    }
}
```

```java
package com.erick.multithread.d02;

import com.erick.multithread.util.Sleep;

import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Demo01 {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(1);
        pool.execute(() -> {
            System.out.println("first");
            Sleep.sleep(2);
        });

        pool.execute(() -> {
            System.out.println("second");
            Sleep.sleep(2);
        });

        pool.execute(() -> {
            System.out.println("third");
            Sleep.sleep(2);
        });
        List<Runnable> leftOver = pool.shutdownNow();
        System.out.println(leftOver.size());// 剩余任务为2，正在执行的任务被打断
        System.out.println("main ending"); // 不阻塞调用线程的执行
    }
}
```

## 异步模式-工作线程

### Worker Thread 

- 让有限的工作线程来轮流异步处理无限多的任务
- 不同的任务类型应该使用不同的线程池

### 饥饿问题

- 单个固定大小线程池，处理不同类型的任务时，会有饥饿现象

```bash
# 假设线程池包含2个线程
- 一个线程负责洗菜，一个线程负责做饭， 
- 线程任务：洗菜-->再做饭(同步等待结果)，才算结束
- 1个菜，   刚好2个线程都可以运行
- 2个菜，   那么两个线程都洗菜，但没线程做饭，就会出现线程饥饿
```

```java
package com.erick.multithread.d05;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.*;

public class Demo02 {
    public static void main(String[] args) {
        CookingService cookingService = new CookingService();
        cookingService.hungryDeal();
    }
}

@Slf4j
class CookingService {

    /*正常情况*/
    public void normalDeal() {
        ExecutorService pool = Executors.newFixedThreadPool(2);
        // 只有一道菜
        prepareFood(pool);
    }

    /*线程饥饿*/
    public void hungryDeal() {
        ExecutorService pool = Executors.newFixedThreadPool(2);
        //  多道菜
        for (int i = 0; i < 5; i++) {
            prepareFood(pool);
        }
    }

    public void prepareFood(ExecutorService pool) {
        pool.execute(new Runnable() {
            @Override
            public void run() {
                log.info("{} prepare", Thread.currentThread().getName());
                Future<String> firstJob = pool.submit(new Callable<String>() {
                    @Override
                    public String call() throws Exception {
                        washVegetables();
                        return "干净的菜";
                    }
                });
                /*同步等待结果*/
                String washResult;
                try {
                    washResult = firstJob.get();
                } catch (InterruptedException | ExecutionException e) {
                    throw new RuntimeException(e);
                }
                cookVegetables(washResult);
                log.info("food is ready");
            }
        });
    }

    private void washVegetables() {
        log.info("{}---洗菜中", Thread.currentThread().getName());
        Sleep.sleep(2);
    }

    private void cookVegetables(String washedVegetable) {
        log.info("{}----做饭中", Thread.currentThread().getName());
        Sleep.sleep(2);
    }
}

```

### 饥饿解决

- 增加线程池的线程数量，但是不能从根本解决问题
- 不同的任务类型，使用不同的线程池

```java
@Slf4j
class CookingService {
    public void process() {
        ExecutorService washPool = Executors.newFixedThreadPool(2);
        ExecutorService cookPool = Executors.newFixedThreadPool(2);
        //  多道菜
        for (int i = 0; i < 5; i++) {
            prepareFood(washPool, cookPool);
        }
    }

    public void prepareFood(ExecutorService washPool, ExecutorService cookPool) {
        cookPool.execute(new Runnable() {
            @Override
            public void run() {
                log.info("{} prepare", Thread.currentThread().getName());
                Future<String> firstJob = washPool.submit(new Callable<String>() {
                    @Override
                    public String call() throws Exception {
                        washVegetables();
                        return "干净的菜";
                    }
                });
                /*同步等待结果*/
                String washResult;
                try {
                    washResult = firstJob.get();
                } catch (InterruptedException | ExecutionException e) {
                    throw new RuntimeException(e);
                }
                cookVegetables(washResult);
                log.info("food is ready");
            }
        });
    }

    private void washVegetables() {
        log.info("{}---洗菜中", Thread.currentThread().getName());
        Sleep.sleep(2);
    }

    private void cookVegetables(String washedVegetable) {
        log.info("{}----做饭中", Thread.currentThread().getName());
        Sleep.sleep(2);
    }
}
```

# 
