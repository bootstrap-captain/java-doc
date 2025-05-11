# 简介

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.11.4</version>
    <scope>test</scope>
</dependency>
```

![image-20250509093913976](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250509093913976.png)

## 入门案例

### SRC

```java
package com.nike;

public class AwsService {
    public String getTopicName(String topicName) {
        return topicName.toUpperCase();
    }
}
```

### UT

- 在test目录下，同包，注意类名，方法名

```java
package com.nike;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

public class AwsServiceTest {
    private AwsService awsService;

    @BeforeEach
    public void init() {
        /*进行类的初始化*/
        awsService = new AwsService();
    }

    @Test
    public void getTopicNameTest() {
        String input = "xian";
        String result = awsService.getTopicName(input);
        Assertions.assertEquals("XIAN", result);
    }
}
```

## 断言机制

- org.junit.jupiter.api.Assertions提供的断言机制

```bash
# 相等
- Assertions.assertEquals(Object expected, Object actual)        
- Assertions.assertNotEquals(Object unexpected, Object actual)   
- Assertions. assertNotEquals(Object unexpected, Object actual, String message)   # 不相等时候的报错信息

# 为空
- Assertions.assertNull(Object actual)           
- Assertions.assertNotNull(Object actual)      

# 同一个对象
- Assertions.assertSame(Object expected, Object actual)
- Assertions.assertNotSame(Object unexpected, Object actual)

# true or false
- Assertions.assertTrue(boolean condition)
- Assertions.assertFalse(boolean condition)

# 数组元素顺序是否相同，可以为数组的任意类型
- Assertions.assertArrayEquals(boolean[] expected, boolean[] actual)

# 组合断言， 所有断言都成功才算成功
- Assertions.assertAll(String heading, Executable... executables)

# 异常断言
- Assertions.assertThrows(Class<T> expectedType, Executable executable)
```

## 常见注解

```bash
# @BeforeAll   @AfterAll  
- 该类中的所有测试方法运行前或者后运行
- 必须是static修饰

# @Disabled
- 禁用掉(跳过)某个测试方法

# @Timeout(value = 2, unit = TimeUnit.SECONDS)
- 测试方法是否超时： 如果超时则报错
```

## 案例精进

### BeforeAll(Each)

- BeforeAll: 静态方法上，初始化整个类都会使用到的资源，只会初始化一次。适用于各个方法对该资源使用不会冲突
- BeforeEach： 初始化一些，各个方法使用时候，都会初始化。适用于可能会冲突的资源

#### SRC

```java
package com.nike;

public class AwsService {
    public String getTopicName(String topicName) {
        return topicName.toUpperCase();
    }
}
```

#### UT

```java
package com.nike;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

public class AwsServiceTest {
    private AwsService awsService;

    private static String password;

    @BeforeAll
    public static void setUp() {
        password = "123456";
        System.out.println("once setup");
    }

    @BeforeEach
    public void init() {
        awsService = new AwsService();
    }

    @Test
    public void getTopicNameTest() {
        String input = "xian";
        String result = awsService.getTopicName(input);
        Assertions.assertEquals("XIAN", result);
    }
}
```

### 组合断言

#### SRC

```java
package com.nike;

import java.util.HashMap;
import java.util.Map;

public class AwsService {
    public Map<String, String> getSnsInfo(String name, String region) {
        Map<String, String> info = new HashMap<>();
        info.put("name", name);
        info.put("region", region);
        return info;
    }
}
```

#### UT

- 部分组合断言，可拆分为单个断言

```java
package com.nike;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.util.Map;

public class AwsServiceTest {
    private AwsService awsService;

    @BeforeEach
    public void init() {
        awsService = new AwsService();
    }

    @Test
    public void getSnsInfoTest() {
        String nameInput = "sqs";
        String regionInput = "xian";
        Map<String, String> result = awsService.getSnsInfo(nameInput, regionInput);

        Assertions.assertAll("组合测试",
                () -> Assertions.assertEquals(2, result.size()),
                () -> Assertions.assertEquals("sqs", result.get("name")),
                () -> Assertions.assertEquals("xian", result.get("region")));
    }
}
```

### 超时断言

#### SRC

```java
package com.nike;

import java.util.concurrent.TimeUnit;

public class AwsService {
    public void heavyWork() throws InterruptedException {
        System.out.println("coming");
        TimeUnit.SECONDS.sleep(3);
        System.out.println("out");
    }
}
```

#### UT

```java
package com.nike;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.Timeout;

import java.util.concurrent.TimeUnit;

public class AwsServiceTest {
    private AwsService awsService;

    @BeforeEach
    public void init() {
        awsService = new AwsService();
    }

    @Test
    @Timeout(value = 2, unit = TimeUnit.SECONDS)
    public void heavyWorkTest() throws InterruptedException {
        awsService.heavyWork();
    }
}
```

### 异常断言

#### SRC

```java
package com.nike;

import java.util.NoSuchElementException;

public class AwsService {
    public int validateId(int id) {
        if (id == 1) {
            throw new IllegalArgumentException();
        }

        if (id == 2) {
            throw new NoSuchElementException();
        }

        if (id == 3) {
            throw new ArrayIndexOutOfBoundsException();
        }
        return id;
    }
}
```

#### UT

```java
package com.nike;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.util.NoSuchElementException;

public class AwsServiceTest {
    private AwsService service;

    @BeforeEach
    public void init() {
        service = new AwsService();
    }

    @Test
    public void correctIdTest() {
        int result = service.validateId(5);
        Assertions.assertEquals(5, result);
    }

    @Test
    public void illegalIdTest() {
        int id = 1;
        Assertions.assertThrows(IllegalArgumentException.class,
                () -> service.validateId(id));
    }

    @Test
    public void noSuchIdTest() {
        int id = 2;
        Assertions.assertThrows(NoSuchElementException.class,
                () -> service.validateId(id));
    }

    @Test
    public void outOfIndexdTest() {
        int id = 3;
        Assertions.assertThrows(ArrayIndexOutOfBoundsException.class,
                () -> service.validateId(id));
    }
}
```

# Mockito-@Mock

```bash
# 场景
- 1.  A类方法中   --->   调用外部client/connection，不希望去调用真实的client/connection
- 2.  A类方法中   --->   调用B类方法 (不希望去调用B类代码真实运行，因为B类代码已经自测)

# B类可以是本项目其他的类，也可以是第三方jar包里面的类
- 单纯使用junit不能满足以上需求，因此引入Mockito来增强             
```

```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.11.4</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>5.14.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## 默认行为-零值

- 被Mock的对象，执行方法时，不会走实际的调用，默认返回结果为该方法的'零值'
- 依赖的服务，可以通过setter或construtor来注入

### SRC

```java
package com.nike;

public class SqsService {
    public String getSqsTopic(String productName) {
        return "nike" + productName;
    }
}
```

```java
package com.nike;

import lombok.Setter;

public class AwsService {

    @Setter
    private SqsService sqsService;

    public String awsGetTopicName() {
        String appleTopic = sqsService.getSqsTopic("apple");
        String huaweiTopic = sqsService.getSqsTopic("huawei");
        return (appleTopic + huaweiTopic).toUpperCase();
    }
}
```

### UT

```java
package com.nike;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.MockitoAnnotations;

public class AwsServiceTest {
    private AwsService service;

    /*Mockito会对该类实例化，构造一个假的类*/
    @Mock
    private SqsService mockSqsService;

    @BeforeEach
    public void init() {
        /*代表本类开启了Mockito功能*/
        MockitoAnnotations.openMocks(this);
        service = new AwsService();
        service.setSqsService(mockSqsService);
    }

    @Test
    public void awsGetTopicNameTest() {
        /*调用三方类时，默认返回NULL*/
        String result = service.awsGetTopicName();
        /*验证一：结果*/
        Assertions.assertNotNull(result);
        /*验证二：调用了两次*/
        Mockito.verify(mockSqsService, Mockito.times(2)).getSqsTopic(Mockito.anyString());
    }
}
```

## 打桩处理

- 被Mock的对象，执行方法时，不会走实际的调用，可以给指定返回值

### SRC

```java
package com.nike;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class SqsService {

    /*无参无返回值*/
    public void work() {
        log.info("work");
    }

    /*有参有返回值*/
    public String sleep(String bed, int seconds) {
        log.info("sleep={}, seconds={}", bed, seconds);
        return "nice";
    }
}
```

```java
package com.nike;

import lombok.Setter;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class AwsService {

    @Setter
    private SqsService sqsService;


    public void awsWork() {
        sqsService.work();
    }

    public String awsSleep() {
        return sqsService.sleep("xian", 12);
    }
}
```

### UT

```java
package com.nike;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.MockitoAnnotations;

public class AwsServiceTest {
    private AwsService service;
    @Mock
    private SqsService mockSqsService;

    @BeforeEach
    public void init() {
        MockitoAnnotations.openMocks(this);
        service = new AwsService();
        service.setSqsService(mockSqsService);
    }

    /*无参无返回值：不需要打桩，默认走0值即可*/
    @Test
    public void workTest() {
        service.awsWork();
        /*验证方法被调用了即可*/
        Mockito.verify(mockSqsService, Mockito.times(1)).work();
    }

    @Test
    public void sleepTest() {
        Mockito.when(mockSqsService.sleep(Mockito.anyString(), Mockito.anyInt())).thenReturn("北京");
        String result = service.awsSleep();
        Assertions.assertEquals("北京", result);
    }
}
```

## ❤️换个写法

### SRC

```java
package com.nike;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class SqsService {
    public void work() {
        log.info("work");
    }
}
```

```java
package com.nike;

import lombok.Setter;

public class AwsService {

    @Setter
    private SqsService sqsService;

    public void awsWork() {
        sqsService.work();
    }
}
```

### UT

```java
package com.nike;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

public class AwsServiceTest {
    private AwsService service;

    private SqsService mockSqsService;

    @BeforeEach
    public void init() {
        service = new AwsService();
        /*手动new，不需要开启了*/
        mockSqsService = Mockito.mock(SqsService.class);
        service.setSqsService(mockSqsService);
    }

    @Test
    public void workTest() {
        service.awsWork();
        Mockito.verify(mockSqsService, Mockito.times(1)).work();
    }
}
```

# Mockito-静态/构造器

- 静态方法，构造器的Mock，现在已经被集成到了Mockito-core中

## 构造器

- 如果A类的方法中，new出B类的对象，可以对这些类进行打桩

### SRC

```java
package com.nike;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class SqsService {
    public SqsService() {
        log.info("SqsService 真实的");
    }

    public String work() {
        return "SQS";
    }
}
```

```java
package com.nike;

public class AwsService {

    public String awsWork() {
        SqsService sqsService = new SqsService();
        String result = sqsService.work();
        return result + "AWS";
    }
}
```

### UT

```java
package com.nike;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.MockedConstruction;
import org.mockito.Mockito;

public class AwsServiceTest {
    private AwsService service;

    @BeforeEach
    public void init() {
        service = new AwsService();
    }

    @Test
    public void workTest() {
        /*mock的值: try里面，用完后就会释放掉，不会影响其他的，用到SqsService的类*/
        try (MockedConstruction<SqsService> sqsService =
                     Mockito.mockConstruction(SqsService.class, (mock, context) -> {
                         // mock模拟的值
                         Mockito.when(mock.work()).thenReturn("MOCK-SQS");
                     })) {
            String result = service.awsWork();
            Assertions.assertEquals("MOCK-SQSAWS", result);
        }
    }
}
```

## 静态方法

- A类方法，调用了B类中的静态类
- mock对应的对象，是全局的，结束后应该进行mock的释放，否则会影响到其他类

### SRC

```java
package com.nike;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class EncryptUtil {

    public static void work() {
        log.info("无参-无返回值");
    }

    public static void sleep(String bed) {
        log.info("有参-无返回值");
    }

    public static String getName() {
        log.info("无参-有返回值");
        return "NIKE";
    }

    public static String getInfo(String info) {
        log.info("有参-有返回值");
        return info.toUpperCase();
    }
}
```

```java
package com.nike;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class AwsService {
    public void awsWork() {
        EncryptUtil.work();
        EncryptUtil.work();
    }

    public void awsSleep() {
        EncryptUtil.sleep("nike");
    }

    public String awsGetName() {
        return EncryptUtil.getName();
    }

    public String awsGetInfo() {
        return EncryptUtil.getInfo("nike");
    }
}
```

### UT

```java
package com.nike;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.MockedStatic;
import org.mockito.Mockito;

public class AwsServiceTest {
    private AwsService service;

    @BeforeEach
    public void init() {
        service = new AwsService();
    }

    /*无参-无返回值*/
    @Test
    public void awsWorkTest() {
        try (MockedStatic<EncryptUtil> mockedEncryptUtil = Mockito.mockStatic(EncryptUtil.class)) {
            // 1. 跳过static的方法调用
            mockedEncryptUtil.when(EncryptUtil::work).then(invocationOnMock -> null);
            service.awsWork();

            /*验证调用次数*/
            mockedEncryptUtil.verify(EncryptUtil::work, Mockito.times(2));
        }
    }

    /*有参-无返回值*/
    @Test
    public void awsSleepTest() {
        try (MockedStatic<EncryptUtil> mockedEncryptUtil = Mockito.mockStatic(EncryptUtil.class)) {
            mockedEncryptUtil.when(() -> EncryptUtil.sleep(Mockito.anyString())).then(invocationOnMock -> null);
            service.awsSleep();
            /*验证调用次数*/
            mockedEncryptUtil.verify(() -> EncryptUtil.sleep(Mockito.anyString()), Mockito.times(1));
        }
    }

    @Test
    public void awsGetNameTest() {
        try (MockedStatic<EncryptUtil> mockedEncryptUtil = Mockito.mockStatic(EncryptUtil.class)) {
            mockedEncryptUtil.when(EncryptUtil::getName).thenReturn("MOCK");
            String result = service.awsGetName();
            Assertions.assertEquals("MOCK", result);
        }
    }

    @Test
    public void awsGetInfoTest() {
        try (MockedStatic<EncryptUtil> mockedEncryptUtil = Mockito.mockStatic(EncryptUtil.class)) {
            mockedEncryptUtil.when(() -> EncryptUtil.getInfo(Mockito.anyString())).thenReturn("MOCK");
            String result = service.awsGetInfo();
            Assertions.assertEquals("MOCK", result);
        }
    }
}
```

# Mockito-@Spy

```bash
# 场景
- A类中： 方法a ---> 方法b，   b方法已经测过，因此在测试a方法时跳过方法b

# 用法
- 默认行为-零值： 被spy的测试对象，默认走实际调用
- 打桩： 就会跳过实际方法，mock一个假的结果

# 限制
- 打桩方法，如果本类中被打桩的方式是私有的，则无法实现，可以改为protected
```

## 默认行为-零值

### SRC

```java
package com.nike;

public class MessageService {
    public String convertMessage(String message) {
        return message.toUpperCase();
    }
}
```

### UT

```java
package com.nike;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.MockitoAnnotations;
import org.mockito.Spy;

public class MessageServiceTest {
    /*会初始化对象*/
    @Spy
    private MessageService service;

    @BeforeEach
    public void init() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void convertMessageTest() {
        String result = service.convertMessage("haha");
        Assertions.assertEquals("HAHA", result);
    }
}
```

## 打桩

### SRC

```java
package com.nike;

public class MessageService {

    public String processMessage(String message) {
        String converted = convertMessage(message);
        return converted + "NIKE";
    }

    protected String convertMessage(String message) {
        return message.toUpperCase();
    }
}
```

### UT

```java
package com.nike;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import org.mockito.MockitoAnnotations;
import org.mockito.Spy;

public class MessageServiceTest {
    /*会初始化对象*/
    @Spy
    private MessageService service;

    @BeforeEach
    public void init() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void processMessageTest() {
        /*打桩*/
        Mockito.doReturn("mock").when(service).convertMessage(Mockito.anyString());
        String result = service.processMessage("hello");
        Assertions.assertEquals("mockNIKE", result);
    }
}
```

## ♥️换个写法

### SRC

```java
package com.nike;

public class MessageService {

    public String processMessage(String message) {
        String converted = convertMessage(message);
        return converted + "NIKE";
    }

    protected String convertMessage(String message) {
        return message.toUpperCase();
    }
}
```

### UT

```java
package com.nike;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

public class MessageServiceTest {
    private MessageService service;

    @BeforeEach
    public void init() {
        /*初始化*/
        service = Mockito.spy(MessageService.class);
    }

    @Test
    public void processMessageTest() {
        /*打桩*/
        Mockito.doReturn("mock").when(service).convertMessage(Mockito.anyString());
        String result = service.processMessage("hello");
        Assertions.assertEquals("mockNIKE", result);
    }
}
```

# SpringBoot-3.4.5

- 已经封装好了Junit-5和Mockito

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

## SRC

```java
package com.nike.service;

import org.springframework.stereotype.Service;

@Service
public class HomeService {

    public String getHomeTag() {
        return "NIKE";
    }
}
```

```java
package com.nike.controller;

import com.nike.service.HomeService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/home")
@RequiredArgsConstructor
public class HomeController {

    private final HomeService homeService;

    @GetMapping("/info")
    public String getHomeInfo(String name) {
        String result = processName(name);
        String tag = homeService.getHomeTag();
        return result + tag;
    }

    protected String processName(String name) {
        return name.toUpperCase();
    }
}
```

## UT-1-不启动主启动类

```java
package com.nike.controller;

import com.nike.service.HomeService;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mockito;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.bean.override.mockito.MockitoBean;
import org.springframework.test.context.bean.override.mockito.MockitoSpyBean;
import org.springframework.test.context.junit.jupiter.SpringExtension;

/*激活Mockito的注解*/
@ExtendWith(SpringExtension.class)
/*该测试类HomeController的对应的bean放在容器中*/
@ContextConfiguration(classes = HomeController.class)
public class HomeControllerTest {

    /*类似@Autowired*/
    @MockitoSpyBean
    private HomeController homeController;

    /*依赖的类：放在容器中*/
    @MockitoBean
    private HomeService homeService;

    /*上面的步骤，就已经处理好了对应的DI关系*/

    @Test
    public void getHomeInfoTest() {
        /*类中自己打自己方法的桩*/
        Mockito.doReturn("ERICK").when(homeController).processName(Mockito.anyString());

        /*类中跳过其他类的方法执行*/
        Mockito.when(homeService.getHomeTag()).thenReturn("MOCK");
        String result = homeController.getHomeInfo("va");
        Assertions.assertEquals("ERICKMOCK", result);
    }
}
```

## UT-2-启动主启动类

```java
package com.nike.controller;

import com.nike.service.HomeService;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.bean.override.mockito.MockitoBean;
import org.springframework.test.context.bean.override.mockito.MockitoSpyBean;

@SpringBootTest
/**
 * 直接在被测试类上加， 这样就会加载主启动类，同时完成测试
 * 使用spring容器功能， 检验项目是否能够启动， 时间稍微长
 */
public class HomeControllerTest {
    @MockitoSpyBean
    private HomeController homeController;
    @MockitoBean
    private HomeService homeService;

    @Test
    public void getHomeInfoTest() {
        Mockito.doReturn("ERICK").when(homeController).processName(Mockito.anyString());
        Mockito.when(homeService.getHomeTag()).thenReturn("MOCK");
        String result = homeController.getHomeInfo("va");
        Assertions.assertEquals("ERICKMOCK", result);
    }
}

```

