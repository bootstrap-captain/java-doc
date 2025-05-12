# 基本介绍

## 简介

- 开源，高性能，无模式的文档型数据库
- NoSQL数据库的一种，是最像关系型数据库的NoSQL
- 数据类型：BSON，类似JSON

|        | MySQL    | MongoDB    | MongoDB example           |
| ------ | -------- | ---------- | ------------------------- |
| 数据库 | database | database   |                           |
| 表     | table    | collection |                           |
| 行     | row      | document   | {"name":"Erick","age",26} |
| 列     | column   | field      |                           |
| 索引   | index    | index      |                           |
| 表连接 |          | NA         | 通过嵌入文档来代替表连接  |
| 主键   |          | _id        | 自动生成                  |

## 场景

- 数据量大
- 读写频繁
- 数据价值低，对事务要求不高

# 安装

- MongoDB Community Server

## 单机版

- [官方安装文档](https://www.mongodb.com/try/download/community)

![image-20250512102422236](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250512102422236.png)

### 上传

```bash
# 上传
put /Users/shuzhan/Desktop/mongodb-linux-x86_64-rhel70-7.0.20.tgz /opt
# 解压
tar -xvf mongodb-linux-x86_64-rhel70-7.0.20.tgz

# 解压后的目录
-rw-r--r-- 1 root root  30608 Apr 26 06:59 LICENSE-Community.txt
-rw-r--r-- 1 root root  16726 Apr 26 06:59 MPL-2
-rw-r--r-- 1 root root   2094 Apr 26 06:59 README
-rw-r--r-- 1 root root 122512 Apr 26 06:59 THIRD-PARTY-NOTICES
drwxr-xr-x 2 root root   4096 May 12 10:29 bin
-rw-r--r-- 1 root root      0 Apr 26 07:01 sbom_from_silk.json

# 移动解压后的文件夹到指定目录
# 所有的mongo的文件，都移动到这个目录中
mv mongodb-linux-x86_64-rhel70-7.0.20 /usr/local/mymongo

# 新建几个目录，用来存储日志和数据
cd /usr/local/mymongo
mkdir -p data/db                # 存储数据
mkdir -p log                    # 存储日志
```

### mongoldb.conf

```bash
systemLog:
   destination: file                                  # 以文件的形式输出日志
   path: "/usr/local/mymongo/log/mongod.log"         # 具体的绝对路径
   logAppend: true                                    # mongo重启时，日志附加到原文件末尾

storage:
   dbPath: "/usr/local/mymongo/data/db"                # 存储数据的目录

processManagement:
   fork: true                                          # 后台启动

net:
   bindIp: 172.17.56.24                               # 局域网ip，非公网IP
   port: 27017
```

```bash
put /Users/shuzhan/Desktop/mongodb.conf /opt
```

### 启动

```bash
# 启动: 在bin目录中
/usr/local/mymongo/bin/mongod -f /opt/mongodb.conf 

- 如果启动失败，一般是配置文件出错，去对应的log里面查找即可

# 查看
ps -ef | grep mongod

# 远程Compass连接
- 连接的IP为公网IP
```

![image-20250512103544562](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250512103544562.png)

### 安全认证

- 默认的mongodb启动后，是不需要安全认证的
- mongodb提供了基于用户的权限认证
- 通过Compass登陆进去

```bash
use admin;                 #  切换到admin
show collections;          #  包含 system.version 的这个collection
db.system.version.find();   

# 结果
{
  _id: 'featureCompatibilityVersion',
  version: '7.0'
}
```

#### 管理员账户

```bash
# root: 超级权限
db.createUser({user:"erickroot", pwd:"123456", roles:[{"role":"root", "db":"admin"}]});
show collections;           # 在admin库创建了一个system.users的collection
db.system.users.find();

# userAdminAnyDatabase: 在指定数据库创建和修改用户(除了config和local库存)
db.createUser({user:"erickadmin", pwd:"123456", roles:[{"role":"userAdminAnyDatabase","db":"admin"}]});
show collections;           # 在admin库的system.users添加用户
db.system.users.find();

# 删除用户
db.dropUser("erickadmin");

# 修改密码
db.changeUserPassword("erickadmin","123456");
```

#### 关闭服务端

```bash
ps -ef |grep mongo         # 查看端口

kill -9 1741               # 杀死进程

# 在配置文件中：开启认证
```

#### 配置文件

```bash
systemLog:
   destination: file                                  # 以文件的形式输出日志
   path: "/usr/local/mymongo/log/mongod.log"         # 具体的绝对路径
   logAppend: true                                    # mongo重启时，日志附加到原文件末尾

storage:
   dbPath: "/usr/local/mymongo/data/db"                # 存储数据的目录

processManagement:
   fork: true                                          # 后台启动

net:
   bindIp: 172.17.56.24                               # 局域网ip，非公网IP
   port: 27017
   
security:
  authorization: enabled                             # 开启权限认证   
```

#### 重启服务端

```bash
# 上传配置文件
put /Users/shuzhan/Desktop/mongodb.conf /opt

# 启动
/usr/local/mymongo/bin/mongod -f /opt/mongodb.conf 
```

# 基本指令

- 区分大小写

## database

```bash
# 查看有权限的数据库
show dbs;
show databases;

# 创建数据库: 如果不存在，则创建新，并切换到这个db中
# 暂时存放在内存中，还没做持久化， 因此这个时候查看不到
# 插入一条数据库后，就会持久化
use erickdb;

# 查看当前在哪个库
db;

# 删除数据库： 删除已经持久化的数据库
db.dropDatabase();
```

```bash
# 本身自带的库

# admin
- root数据库，负责权限管理的

# local
- 数据永远不会被复制
- 用来存储仅限于本地单台服务器的数据

# config
- 设置了mongo的分片的时候，config数据库在内部使用， 用于保存分片的相关信息
```

## collection

- 类似MySQL的表

```bash
# 创建
# 显示创建 ： 创建一个名字为 ‘good’的collection
db.createCollection('good');

# 查看表
show collections;
show tables;

# 删除表 : 删除‘good’表
db.good.drop();
```

## document

- 插入一条文档后，该文档所在的库，才会持久化到硬盘上

```bash
# 插入单个: 会按照长度最长的filed来组建
# ‘good’可以不存在于databse里面，执行插入的时候，会自动创建一个collection
db.good.insertOne({"name":"erick","age":20});
db.good.insertOne({"name":"lucy","age":18,"address":"changsha"});

# 插入多条：一旦其中一个失败，将会终止插入，但是之前插入的不会回滚
db.good.insertMany([{"name":"jack"},{"name":"tom","age":10}]);

# 查找全部
db.good.find();

# 查看某个: 属性匹配
db.good.find({"name":"jack"});
db.good.findOne({"name":"jack"})     # 只查看其中一条

# 也可以比较，模糊，in等

# 查看时候，显示address和name字段，_id是自带的
db.good.find({"name":"lucy"},{"address":1, "name":1});

# 分页
db.good.find().limit(3).skip(1);

# 排序: 1为升序， -1为降序
db.good.find().sort({"age":1});
```

```bash
# 更新: 参数一：匹配条件        参数二：更新其中的字段
db.good.updateOne({"name":"erick"},{$set:{"address":"xian"}});

db.good.updateMany({"name":"lucy"},{$set:{"address":"beijig"}});

# 删除
db.good.deleteOne({"name":"erick"});
db.good.deleteMany({"name":"erick"});
```

```bash
# 统计
db.good.countDocuments();
```

## 索引

```bash
# 查看当前索引： 默认_id是索引
db.student.getIndexes();

# name：索引的字段
# v：版本号
# key：具体的那个字段，1代表升序，-1代表降序
[
  {
    "key": {
      "_id": 1
    },
    "name": "_id_",
    "v": 2
  }
]

# 创建索引
db.student.createIndex({name:1});              # 单个索引
db.student.createIndex({name:1,hobby:-1});     # 多个索引

# 删除索引
db.student.dropIndex({name:1});    # 具体指定索引类型
db.student.dropIndex('name_1');    # 根据索引名称
db.student.dropIndexes();          # 删除所有自定义的index，不会删除_id
```

# Java

- springdata

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

```yaml
spring:
  data:
    mongodb:
      host: 39.105.210.163
      port: 27017
      database: erickdb
```

## 插入

# 查询

## 字段类型为数组

- 数组中的数据，只要有其中一条匹配当前条件，就返回当前文档

![image-20250512145307910](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250512145307910.png)

```java
package com.citi.erick.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
@Slf4j
public class QueryArrayService {
    @Autowired
    private MongoTemplate mongoTemplate;

    private String collectionName = "replay";

    public void queryArray() {
        Query query = new Query();
        // action: 一级字段
        query.addCriteria(Criteria.where("action").elemMatch(
                // comment：数组中的二级字段
                Criteria.where("comment").is("拒绝")
        ));

        List<Object> results = mongoTemplate.find(query, Object.class, collectionName);
        log.info("results={}", results);
        log.info("size={}", results.size());
    }
}
```

## 聚合查询

### count

#### root-level

- country为一级字段，根据country进行分组后统计count

![image-20250512145358220](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250512145358220.png)

```java
package com.citi.erick.service;

import com.citi.erick.entity.CountryCountResult;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.aggregation.Aggregation;
import org.springframework.data.mongodb.core.aggregation.AggregationResults;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
@Slf4j
public class QueryService {

    @Autowired
    private MongoTemplate mongoTemplate;

    private String collectionName = "people";

    public List<CountryCountResult> aggregateByCountry() {
        Aggregation agg = Aggregation.newAggregation(
                /*1.添加过滤条件*/
                Aggregation.match(Criteria.where("email").exists(true)),
                /*2. 对country进行group后，统计count为totalCount*/
                Aggregation.group("country").count().as("totalCount")
        );

        /*映射的类型*/
        AggregationResults<CountryCountResult> aggregateResult = mongoTemplate.aggregate(agg, collectionName, CountryCountResult.class);
        List<CountryCountResult> results = aggregateResult.getMappedResults();
        return results;
    }
}
```

```java
package com.citi.erick.entity;

import lombok.Data;
import org.springframework.data.mongodb.core.mapping.Field;

@Data
public class CountryCountResult {
    /*原始的字段是_id，需要将其映射到country上*/
    @Field("_id")
    private String country;
    private long totalCount;
}
```

```json
[
  {
    "country": "美国",
    "totalCount": 1
  },
  {
    "country": "中国",
    "totalCount": 2
  },
  {
    "country": "非洲",
    "totalCount": 1
  },
  {
    "country": "韩国",
    "totalCount": 1
  }
]
```

#### second-level

- 某个字段为二级字段，根据二级字段的某个值

![image-20250512145419242](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250512145419242.png)

```java
package com.citi.erick.service;

import com.citi.erick.entity.CountryCountResult;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.aggregation.Aggregation;
import org.springframework.data.mongodb.core.aggregation.AggregationResults;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
@Slf4j
public class AggregateService {

    @Autowired
    private MongoTemplate mongoTemplate;

    private String collectionName = "student";

    public List<CountryCountResult> aggregateByCountry() {
        Aggregation agg = Aggregation.newAggregation(
                /*1.添加过滤条件: 二级字段*/
                Aggregation.match(Criteria.where("order.email").exists(true)),
                /*2. 二级字段: 对country进行group后，统计count为totalCount*/
                Aggregation.group("order.country").count().as("totalCount")
        );

        /*映射的类型*/
        AggregationResults<CountryCountResult> aggregateResult = mongoTemplate.aggregate(agg, collectionName, CountryCountResult.class);
        List<CountryCountResult> results = aggregateResult.getMappedResults();
        return results;
    }
}
```

#### unwind

- 是MongoTemplate中聚合管道的一个步骤，将文档中的一个数组字段拆分为多个文档，并将原始文档的其他字段复制到新文档中

![image-20250512145438047](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250512145438047.png)

```java
package com.citi.erick.service;

import com.citi.erick.entity.CountryCountResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.aggregation.Aggregation;
import org.springframework.data.mongodb.core.aggregation.AggregationResults;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UnwindService {
    @Autowired
    private MongoTemplate mongoTemplate;

    private String collectionName = "phone";

    public List<CountryCountResult> aggregateByCountry() {
        Aggregation agg = Aggregation.newAggregation(
                /*1.拆分数组，并将文档其他的值复制到新文档上*/
                Aggregation.unwind("products"),

                /*2. 按照一定规则匹配*/
                Aggregation.match(Criteria.where("products.country").is("中国")),

                /*3. 进行统计*/
                Aggregation.group("products.country").count().as("totalCount")
        );

        /*映射的类型*/
        AggregationResults<CountryCountResult> aggregateResult = mongoTemplate.aggregate(agg, collectionName, CountryCountResult.class);
        List<CountryCountResult> results = aggregateResult.getMappedResults();
        return results;
    }
}
```

```json
[
  {
    "country": "中国",
    "totalCount": 4
  }
]
```

