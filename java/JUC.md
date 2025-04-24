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

