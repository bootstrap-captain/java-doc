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
- 删除target目录

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

```xml

```

### Effective Pom

- 一个项目的pom.xml，经过超级pom，父pom，当前pom，根据一定的优先级组合后，形成的最终的pom