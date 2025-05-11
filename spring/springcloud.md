# 架构演变

## 单体

- ALL IN ONE：所有功能模块都在一个项目

```bash
# 优点：  开发部署方便
# 缺点：  无法应对高并发
```

![image-20250506164127075](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250506164127075.png)

## 集群

```bash
# 优点
- 解决了单体架构的大并发问题
# 不足
     # 模块化升级
     - 部分功能模块经常升级，每次需要整体进行打包部署
     # 多语言团队
     - 假如某个业务模块需要其他语言开发
```

![image-20250506170611722](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250506170611722.png)

## 分布式

# 简介

- 分布式微服务架构的一站式解决方案

```bash
# springboot：     3.4.3
# springcloud：    2024.0.1  
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.citi</groupId>
    <artifactId>replay-erick-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <spring-cloud.version>2024.0.1</spring-cloud.version>
        <spring-boot.version>3.4.3</spring-boot.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

</project>
```

# Nacos

- 3.0.0

## 安装

- 依赖Java17

### 单机版

- 去官网github下载对应的tar包

```bash
# 在阿里云服务器上
cd usr/local
mkdir nacos

put /Users/shuzhan/Desktop/nacos-server-3.0.0.tar.gz /usr/local/nacos

# 解压文件
tar -zxvf nacos-server-3.0.0.tar.gz

# 进入bin目录
cd /usr/local/nacos/nacos/bin

# 启动：单点模式
startup.sh -m standalone
```





