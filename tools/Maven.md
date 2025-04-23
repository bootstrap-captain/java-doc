# 引入

## Dependencies Management

-  jar包下载，依赖，冲突等问题的解决

```bash
# jar包规模
- 项目中手动逐个加入jar包
- maven拉取jar包，并最终打包到对应项目的发布jar包中

# jar包来源
- 手动下载jar包，可能存在奇奇怪怪的问题
- maven拉取jar包，方便快捷规范

# jar包之间依赖关系
- 手动添加，数量庞大，彼此间存在错综复杂的依赖关系，jar包间冲突
- maven来管理冲突
```

## Build Management

- 上传到repo的是.java，通过Maven进行清理原来.class，重新compile，test，package，install，deploy

```bash
# 传统开发
- 将单个.java文件通过 javac 编译成 .class文件
- 执行 测试代码
- 将代码打包上传到服务器部署
- 测试功能
   如此步骤，遇到bug，重复进行

# Maven
 - 自动化进行上述的所有的 批量编译，打包，部署
 - jar包依赖管理
```

# 下载-3.9.9

- [官网下载](https://archive.apache.org/dist/maven/maven-3/3.9.9/binaries/)
- [官网介绍](https://maven.apache.org/index.html)
- Maven依赖Java，Maven环境必须先安装Java
- Mac/Linux选用对应的tar.gz文件，windows选用对应的bin.zip文件

## Centos7.6

```bash
# 上传到usr/local目录下并解压
put /Users/shuzhan/Desktop/apache-maven-3.9.9-bin.tar.gz /usr/local
tar -zxvf apache-maven-3.9.9-bin.tar.gz

# 查看版本， 出现下面代表安装成功
/usr/local/apache-maven-3.9.9/bin/mvn -v


Maven home: /usr/local/apache-maven-3.9.9
Java version: 17.0.4.1, vendor: Oracle Corporation, runtime: /usr/local/java17/jdk-17.0.4.1
Default locale: en_US, platform encoding: ANSI_X3.4-1968
OS name: "linux", version: "3.10.0-957.21.3.el7.x86_64", arch: "amd64", family: "unix"
```

## 参数配置

-  核心配置文件： maven/conf/setting.xml

### repo

- 保存本地下载的jar包

```xml
<!-- 本地的repo仓库-->
<localRepository>/Users/shuzhan/Documents/mvn_repo</localRepository>
```

### 镜像

- 默认去中央仓库拉取jar包，需要对中央仓库配置一个镜像仓库
- 默认的maven，对http类型的中央仓库进行了block，因此需要先删除掉用来的mirror

```xml
<mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
 </mirror>
```

## IDEA配置

- 项目中注意全局配置和项目配置
- Maven配置为刚才下载的maven路径，settings file使用刚才配置的文件，也可以自定义另外的setting文件

### 当前项目

![image-20250420115210685](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250420115210685.png)

### 全局配置

![image-20250420115514310](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250420115514310.png)

### 导入多个项目

![image-20250420120805281](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250420120805281.png)

# 介绍

## 仓库

- 存储资源，包含各种jar包，一般是三类jar

```bash
- 安装到仓库的个人/公司项目

- 第三方包和框架包

- maven插件
```

```bash
# 中央仓库
- 由maven团队维护
- 默认是去国外网站拿
- 一般会配置中央仓库的镜像，建议不要配置多个镜像和中央仓库混用，可能获取到的jar包不一样

# 私服
- 公司内部的maven仓库,私服会从中央仓库(或中央仓库的镜像)拿，获取后并保存到私服
- 加快本地拉取jar包的速度
- 部分自主研发的产品具有知识产权（不应该上传到中央仓库）
  
# 本地仓库
- 本地local的仓库，拉取jar包后会保存到本地的repo中
```

## GAV

- groupId: 域名反写，公司或者组织的ID，一般会再加上项目
- artifactId: 项目中对应的某个工程名称
- version：项目版本号
- 使用唯一坐标，让maven能够找到本地repo中对应的jar包，从而将指定包打包到项目最终的jar包中

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.38</version>
    <scope>provided</scope>
</dependency>
```

## 目录结构

- Maven工程必须严格遵守约定的目录结构，否则无法运行
- 这种目录结构是在超级目录里面配置的(相当于所有POM的父POM)

### 编译前

![image-20250420121651250](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250420121651250.png)

### 编译后

![image-20250420121748643](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250420121748643.png)

## 常用命令/生命周期

- 生命周期： compile, test, package, install, 执行后面命令时，会执行前面的
- 一般都是先clean再去执行后面的动作

```bash
# Jenkins服务器上
- 拉取到对应的java源码后，必须先进入到 pom.xml所在的目录，才能执行对应的  mvn -***

# 如果没进入到对应的目录
The goal you specified requires a project to execute but there is no pom in this directory
```

```bash
# mvn clean
- 删除target目录,删除本地install的jar包

# mvn compile
- 将src下面的main/test从 .java文件编译成 .class文件

# mvn test-compile
- 将测试代码进行编译，放在test-classes中

# mvn test
- 编译src下的业务代码和测试代码，并且执行test，并在surefire-reports中生成测试报告

# mvn package
- 执行上面所有步骤，并打包成预先定义好的jar包： artifactId-version-SNAPSHOT.jar
- 内容不会包含测试代码

# mvn install
- 执行上面所有步骤，并打包存入到本地仓库
- 将本项目的pom文件，重命名为jar包相同的，并存入同样的目录中

# mvn deploy
- 执行上面的步骤，并将对应的jar包上传到maven的linux私服上

# 可以使用组合命令
mvn clean install
```

## 插件

- lifecycle：定义的是生命周期，是一个逻辑上的概念
- plugins：定义的是执行对应生命周期时候的具体操作底层

![image-20250420122842482](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250420122842482.png)

## pom.xml

### 基本标签

- Project Object Model: 项目对象模型
- maven项目必须包含一个pom.xml文件，从而获得整个项目的信息

```xml
<!--当前pom所采用的标签结构，从Maven2开始就固定是4.0.0-->
<modelVersion>4.0.0</modelVersion>

<!--jar： 默认，说明是Java工程
    war： 说明是war工程
    pom： 说明这个工程是用来管理其他工程的工程    
-->
<packaging>jar</packaging>

<!--在构建过程中读取源码时使用的字符集-->
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
```

### super-pom

- 所有的pom.xml如果没有父pom，则都会使用maven本身提供的super pom，类似与Java的Object类
- 超级pom路径： apache-maven-3.9.9/lib/maven-model-builder-3.9.9.jar/   org/apache/maven/model/pom-4.0.0.xml

### Effective Pom

- 一个项目的pom.xml，经过超级pom，父pom，当前pom，根据一定的优先级组合后，形成的最终的pom

# 依赖管理

## scope

- 默认的jar包依赖，为compile级别，在任何地方都可以使用，可以通过scope标签来设定其作用范围

| scope            | 主代码 | 测试代码 | 打包包含该jar包                       | 示例                |
| ---------------- | ------ | -------- | ------------------------------------- | ------------------- |
| complie(default) | Y      | Y        | Y                                     | log4j               |
| test             | N      | Y        | N(测试依赖的jar包不进入最终项目jar包) | junit               |
| provided         | Y      | Y        | N                                     | servant-api, lombok |
| import           |        |          |                                       | 解决依赖单继承问题  |

```bash
# provided：
- 场景一：开发时需要用该jar包，但是部署时，服务器已经有了，所以打包的时候不能带，不然可能冲突，比如jakarta.servlet-api
- 场景二：开发时的注解，然后编译时进行动态生成好的 .class文件。     jar运行时候已经不需要该依赖了，比如 lombok
```

## 依赖传递

### 本地项目

- 本地的多个项目(不管是不是父子项目), A项目依赖B项目时，B项目的包可传递到A项目中
- IDEA中不install时，编译不会出现问题
- A项目在package时候，就会去找对应的B项目的包，所以B项目必须先install或者deploy才可以保证B项目的jar存在

![image-20250423111511484](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250423111511484.png)

### 仅传递compile

- B项目中依赖的第三方包，只有compile级别的，才会打包到B项目中，才会传递到A项目中
- B项目中的所有的源代码，会全部传递到A项目中

## version conflict

- 一个项目中使用了同一个jar包的不同版本，导致冲突

```bash
# 抛异常
- java.lang.ClassNotFoundException:   编译过程找不到类
- java.lang.NoClassDefFoundError：    运行过程找不到类
- java.lang.LinkageError:             不同类加载器分别加载的多个类有相同的全限定类名

# 抛异常
- java.lang.NoSuchMethodError:  程序找不到预期的方法，多见于通过反射调用方法

# 没报错但结果不对
- 两个jar包中的类分别实现了同一个接口，但是因为没有注意命名规范，两个不同的类刚好是同一个名字
```

### 仲裁机制

- Maven提供了对应的仲裁机制，决定项目到底使用哪个对应的jar的版本

#### 最短路径

- 依赖中的相同的GAV，资源层级越深，优先级越低
- 最终会使用lombok2.0

![image-20250423122801516](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250423122801516.png)

#### 路径相同，声明优先

- A和B，在MyProject中先声明哪个，则哪个优先
- 在MyProject的pom中，先声明A，则选用A的lombok1.0

![image-20250423123056853](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250423123056853.png)

#### 覆盖优先

- 在MyProject中，配置了一个jar的不同版本，后配置的覆盖前配置的

![image-20250423125851952](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250423125851952.png)

### 手动指定

- 经过maven仲裁后，会选定一个版本
- 但是在项目运行时，希望能够不使用maven的仲裁版本，而是手动指定一个版本

```bash
# JVM在加载一个jar包的时候，如果加载了其中一个jar包，则其他版本的该jar，就不会再次被加载

# 场景一：低版本漏洞
- 需要手动指定一个高版本的

# 场景二：高版本太新
- 高版本中可能删除了某个类，但是导致某个第三方包运行不起来，因此需要手动指定到低版本
```

#### exclude

- 可以借助Maven helper插件来快速排除某个依赖，从而使该包脱离仲裁体系

![image-20250423130719023](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250423130719023.png)

#### 版本锁定-DM

- 如果某个包被很多个框架引入，那么在exclude时，需要一个个排除，比较麻烦
- 版本锁定会脱离maven仲裁机制，优先级别最高
- 也可以在父工程中控制版本锁定
- 如果父项目和子项目同时进行了版本锁定，子项目优先级最高

![image-20250423131242863](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250423131242863.png)

- DM的优先级低于直接写明依赖

![image-20250423131844440](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250423131844440.png)

#### 依赖隐藏-optional

- B项目可以隐藏自己的依赖，不传递给A项目，不传递该依赖
- 默认为false，true代表可以传递
- 被依赖方解决依赖冲突

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.36</version>
    <optional>true</optional>
</dependency>
```

# 父子项目

## 单继承-parent

- 子项目的V和G如果和父项目线通，可省略
- 父项目V一处修改，子项目处处生效。子可单独定义V，优先级比父高
- 子无需定义V，只需GA信息，再通过版本仲裁机制确定引入的深层依赖的V
- 父项目必须为pom，不写业务代码，仅仅提供依赖版本控制
- 父项目的properties也可以继承

### 自定义父

- 打包的时候，只需要在父工程一键package即可

![image-20250423212335073](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250423212335073.png)

### 第三方父

- 仅仅提供依赖管理，不能提供一键package对应的build功能

![image-20250423214034680](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250423214034680.png)

## 多继承-import

- 如果项目依赖到多个parent，可以使用DM进行多个import

![image-20250423215358590](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250423215358590.png)

![image-20250423215414270](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250423215414270.png)

## 标签属性

- 标签属性： 可以在父工程中定义，也可以在子工程中定义
- 标签在引用时候，只能存在于子父工程，不能存在于通过DependencyManagement来使用

# Build

## fileName

- 默认打包后的文件名 A-V.jar
- 可以通过该属性进行个性化名字

## 微服务打包

- maven的直接打包，不能生成直接可以运行的微服务jar包

```bash
# 微服务jar包
- 当前微服务本身jar包
- 当前微服务所依赖的jar包
- 内置可以直接运行的tomcat
- jar包可以通过 java -jar方式直接启动

# 添加对应的plugin，最终会生成两个jar
erick-customization.jar                # fat jar，即可以直接运行的jar
erick-customization.jar.original       # 去掉 .original, 就是maven默认的打包后的jar
```

```xml
    <build>
        <plugins>
            <!--如果不加该插件，则只包含当前服务的代码-->

            <!--满足微服务打包的插件
            版本信息： 1. 如果parent是springboot的版本，则不配置可以直接使用
                      2. 也可以自定义
                      3. 不能直接通过DM来进行传递，只能通过parent来传递-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>

        <!--自定义打包后的名字: erick-customization.jar -->
        <finalName>erick-customization</finalName>
    </build>
```

![image-20250423220552684](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250423220552684.png)
