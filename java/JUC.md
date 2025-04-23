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

- 启动$JVM(main方法)，即开启一个JVM进程
- JVM进程内包含一个主线程，主线程可以派生出多个其他线程，同时还有一些守护线程(比如垃圾回收线程)
- 主线程，守护线程，派生线程等，cpu随机分配时间片，交替随机执行



