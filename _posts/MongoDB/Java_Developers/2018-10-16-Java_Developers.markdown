---
layout:     post
title:      "MongoDB for Java Developers"
subtitle:   "Overview, Design Goals, the Mongo Shell, JSON Intro, Installing Tools,Overview of Blog Project. Maven, Spark and Freemarker Intro. Mongo Shell, Query Operators, Update Operators and a Few Commands. Patterns, Case Studies & Tradeoffs. Using Indexes, Monitoring And Understanding Performance. Performance In Sharded Environments. Goals, The Use Of The Pipeline, Comparison With SQL Facilities. Drivers, Impact Of Replication And Sharding On Design And Development. Final Exam"
date:       2018-10-16 10:57:02
author:     "于磊"
header-img: "img/mongoDB/Java_Developers/bg.png"
catalog: true
tags:
    - MongoDB
    - Java

---



### Week 1: Introduction

> Overview, Design Goals, the Mongo Shell, JSON Intro, Installing Tools,Overview of Blog Project. Maven, Spark and Freemarker Intro



#### Overview of Building an App with MongoDB

![mongoDB](/img/mongoDB/Java_Developers/overview.jpg)



#### Installing MongoDB(mac)

1.下载 MongoDB Server（https://www.mongodb.com/download-center/v2/community）

2.打开终端进行安装操作


```powershell
# 进入下载目录
cd ~/Downloads/  

# 查看文件
ls -lt

# 解压缩文件
tar xvf mongodb-osx-x86_64-enterprise-4.0.3.tgz

# 进入目录，看到自述文件、许可证、bin目录等
cd mongodb-osx-x86_64-enterprise-4.0.3

# 进入 bin 目录
cd bin

# 查看 bin 目录都有什么
ls -l

# mongo 用于连接到服务器的客户端程序 [shell]
# mongod 是可以运行的服务器数据库程序 [服务器]

# 成为 root 用户
sudo bash

# 制作 dir目录 /data/db
mkdir -p /data/db

# 赋予目录公开权限,这样就可以退出 root用户，使用普通用户写入
chmod 777 /data
chmod 777 /data/db

# 确认目录是777
ls -ld /data/db

# 查看自己的身份是谁(root/yulei)
whoami

# 以自己的身份开始运行
sudo yulei

# 运行 mongoDB
./mongod

# 新开一个窗口，查看路径是什么
pwd

# 运行 mongo shell 
./mongo

# 测试插入一条记录
db.names.insert({'name':'yulei'});

# 测试查找
db.names.find()

# 按 Control-C 来 kill 掉数据库服务器

# 再次成为根
sudo bash

# 将所有 shell 命令程序复制到 usr/local/bin 
cp * /usr/local/bin

# 或者配置.bash_profile PATH变量
export PATH="/Users/yulei/software/mongodb-osx-x86_64-enterprise-4.0.3/bin:$PATH"

# 退出这个 root shell
exit

# 查看 mongod 命令在哪
which mongod

# 键入 mongod 将启动服务器再次运行
mongod
```

3.mongodb 数据库在本机安装完成



#### Installing MongoDB(windows)

1. 下载 MongoDB Server（https://www.mongodb.com/download-center/v2/community）

2. 配置环境变量，系统->高级系统设置->环境变量->追加安装路径到 Path变量上（注意加分号）

3. 打开 cmd 终端 shell 进行操作

   ```powershell
   # 创建 db目录
   md \data
   md \data\db
   
   # 输入 mongod 启动数据库服务器
   mongod
   
   # 再打开 cmd 终端 shell 连接本机27017的数据库
   mongo
   
   # 测试插入一条记录
   db.names.insert({'name':'yulei'});
   
   # 测试查找
   db.names.find()
   
   # 停止服务
   control + C
   ```



#### JSON

> 由键值对组成，键必须是字符串，键和值是由冒号分开的，对象中的字段使用逗号分隔，并使用花括号关闭，支持许多不同的值类型
>
> 访问 www.json.org 查看更多规范

```json
{
  "headline" : "Apple Reported Fourth Quarter Revenue Today",
  "date" : "2015-10-27T22:35:21.908Z",
  "views" : 1132,
  "author" : {
    "name" : "Bob Walker",
    "title" : "Lead Business Editor"
  },
  "published" : true,
  "tags" : [ 
    "AAPL", 
    { "name" : "city", "value" : "Cupertino" },
    [ "Electronics", "Computers" ]
  ]
}

```

Which of the following value types are defined by the JSON spec? Check all that apply.

- object
- array
- string
- number



#### BSON（Binary JSON）

> MongoDB实际上将数据存储为 BSON 或二进制 JSON
>
> www.bsonspec.org 查看更多 BSON 的规范
>
> MongoDB驱动程序以 BSON 的形式发送和接收数据
>
> 当数据写入 MongoDB时，它存储为 BSON
>
> 在应用程序方面，MongoDB 驱动程序将 BSON 映射到任何本机数据类型

BSON 的设计很轻巧

- Lightweight
- Traversable
- Efficient

BSON 扩展了 JSON 值类型，包括整数、双精度日期和二进制数据，例如，支持图像和许多其他类型的数据

```
// JSON
   { "hello" : "world" }

// BSON
"\x16\x00\x00\x00\x02hello\x00 
\x06\x00\x00\x00world\x00\x00"
```



#### Intro to Creating and Reading Documents

**Mongo shell 介绍**：

- Mongo shell 是一个功能齐全的 MongoDB 客户端应用，支持所有 CRUD 操作

- 在 mongo shell中输入 help 显示所有帮助命令
- shell 是一个功能齐全的 JavaScript 解释器

```powershell
	db.help()                    help on db methods
	db.mycoll.help()             help on collection methods
	sh.help()                    sharding helpers
	rs.help()                    replica set helpers
	help admin                   administrative help
	help connect                 connecting to a db help
	help keys                    key shortcuts
	help misc                    misc things to know
	help mr                      mapreduce

	show dbs                     show database names
	show collections             show collections in current database
	show users                   show users in current database
	show profile                 show most recent system.profile entries with time >= 1ms
	show logs                    show the accessible logger names
	show log [name]              prints out the last segment of log in memory, 'global' is default
	use <db_name>                set current database
	db.foo.find()                list objects in collection foo
	db.foo.find( { a : 1 } )     list objects in foo where a == 1
	it                           result of the last line evaluated; use to further iterate
	DBQuery.shellBatchSize = x   set default number of items to display on shell
	exit                         quit the mongo shell
```



命令介绍：

```powershell
# 使用 user <db_name> 选择数据库
use video ## 【注意事项】如果选择的数据库不存在，MongoDB 将为你创建数据库

# 使用 insertOne 插入一条记录
db.movies.insertOne({ "title": "Jaws", "year": 1975, "imdb": "tt0073195" });

# 插入成功后会返回
{
	"acknowledged" : true, 
	"insertedId" : ObjectId("5bc58af578fe53787aaf6836")
}
## acknowledged 意味着它成功了
## insertedId 是唯一的标识符
## [注意事项]，在 MongoDB中，所有文档都必须具有下划线 ID 字段

# 使用insertOne 再多加两条记录
db.movies.insertOne({ "title": "Mad Max 2: The Road Warrior", "year": 1981, "imdb": "tt0082694" })
db.movies.insertOne({ "title": "Raiders of the Lost Ark", "year": 1981, "imdb": "tt0082971" })

# 使用 db.movies.find() 查看所有记录
# 使用 db.movies.find().pretty() 查看所有格式化记录

# 使用 find 过滤标题参数查看匹配的记录
db.movies.find({"title":"Jaws"}).pretty();

# 使用 find 过滤年份参数查看匹配的记录
db.movies.find({"year": 1981}).pretty();

# shell 是一个功能齐全的 JavaScript 解释器，可以像在 JavaScript 程序中那样做
var c = db.movies.find()
c.hasNext() // 返回 true，意味着还有一个文档
c.next // 将获取该文档

## CRUD是 Create，Read，Update和 Delete 的首字母缩写
```



#### System requirements

```powershell
# 查看jdk 版本
java -version
javac -version

Lecture Notes
Updated Requirements:

Since this video was recorded in 2013, we've made some updates. The most important one is that you should use the JDK 1.7, not 1.6; the course will no longer work with 1.6.

Furthermore, as of the writing of this note (1/5/2016), people have been using newer versions of OS X and Windows without problems.
```



#### Installing and Using Maven

1. Apache Maven 下载

2. 安装 Maven

   ```powershell
   # 进入目录
   cd 下载目录
   
   # 查看文件
   ls -lrt | grep apache
   
   # 解压文件
   unzip apache-maven-3.2.5.zip
   
   # 配置 maven 环境变量到.bash_profile 文件中
   export MAVEN_HOME=/Users/yulei/software/apache-maven-3.5.2
   export PATH=$PATH:$MAVEN_HOME/bin
   
   # 检查 maven 版本
   mvn -version
   
   # 运行 maven archetype 插件创建工程，或者使用 IntelliJ IDEA 创建项目
   mvn archetype:generate
   # 选择原型
   # 选择哪个版本
   # 输入 groupId：com.mongodb
   # 输入 项目名称：M101J
   # 选择默认版本和包
   ```



#### Intro to the Spark Web Application Framework

#### Hello world spark style

Spark - a tiny web framework for Java 8：

http://sparkjava.com/download 

https://github.com/perwendel/spark

1. Add dependency to your POM:

```xml
<dependency>
    <groupId>com.sparkjava</groupId>
    <artifactId>spark-core</artifactId>
    <version>2.8.0</version>
</dependency>
```

2. Add main method

```java
import static spark.Spark.*;

public class HelloWorld {
    public static void main(String[] args) {
        get("/hello", (request, response) -> "Hello World!");
    }
}
```

3. View at: <http://localhost:4567/hello>



#### Intro to the Freemarker Templating Engine

#### Hello world freemarker style

Apache FreeMarker™ is a *template engine*: https://freemarker.apache.org/

1. Add dependency to your POM:

   ```xml
   <dependency>
       <groupId>org.freemarker</groupId>
       <artifactId>freemarker</artifactId>
       <version>2.3.28</version>
   </dependency>
   ```

2. Add main method

   ```java
   public class HelloWorldFreemarkerStyle {
   
     public static void main(String[] args) throws IOException, TemplateException {
       Configuration configuration = new Configuration();
       configuration.setClassForTemplateLoading(HelloWorldFreemarkerStyle.class, "/");
       Template helloTemplate = configuration.getTemplate("hello.ftl");
       StringWriter writer = new StringWriter();
       Map<String, Object> helloMap = new HashMap<String, Object>();
       helloMap.put("name", "Freemarker");
       helloTemplate.process(helloMap, writer);
       System.out.println(writer);
     }
   }
   ```

3. 定义 hello.ftl

   ```html
   <html>
     <head>
         <title>Welcome!</title>
     </head>
     <body>
       <h1>Hello ${name}</h1>
     </body>
   </html>
   ```

4. run and output

   ```html
   <html>
     <head>
         <title>Welcome!</title>
     </head>
     <body>
       <h1>Hello Freemarker</h1>
     </body>
   </html>
   ```


#### Spark and Freemarker Together

```java
public class HelloWorldSparkFreemarkerStyle {

  public static void main(String[] args) {
    final Configuration configuration = new Configuration();
    configuration.setClassForTemplateLoading(HelloWorldSparkFreemarkerStyle.class, "/");

    Spark.get("/", (Route) (request, response) -> {
      Template helloTemplate = null;
      StringWriter writer = new StringWriter();
      try {
        helloTemplate = configuration.getTemplate("hello.ftl");
        Map<String, Object> helloMap = new HashMap<>();
        helloMap.put("name", "Freemarker");
        helloTemplate.process(helloMap, writer);
      } catch (Exception e) {
        halt(500); // 返回 500 错误
        e.printStackTrace();
      }
      return writer;
    });
  }
}

```



#### Spark: Handling GET requests

![mongoDB](/img/mongoDB/Java_Developers/process.jpg)

```java
public class SparkRoutes {

  public static void main(String[] args) {

    //  port(5678); <- Uncomment this if you want spark to listen to port 5678 instead of the default 4567

    get("/hello", (request, response) -> "Hello World!");

    post("/hello", (request, response) ->
        "Hello World: " + request.body()
    );

    get("/private", (request, response) -> {
      response.status(401);
      return "Go Away!!!";
    });

    get("/users/:name", (request, response) -> "Selected user: " + request.params(":name"));

    get("/news/:section", (request, response) -> {
      response.type("text/xml");
      return "<?xml version=\"1.0\" encoding=\"UTF-8\"?><news>" + request.params("section") + "</news>";
    });

    get("/protected", (request, response) -> {
      halt(403, "I don't think so!!!");
      return null;
    });

    get("/redirect", (request, response) -> {
      response.redirect("/news/world");
      return null;
    });

    get("/", (request, response) -> "root");
  }
}
```



#### Spark: Handling POST requests

1. Add main method

   ```java
   public class SparkFormHandling {
   
     public static void main(String[] args) {
       final Configuration configuration = new Configuration();
       configuration.setClassForTemplateLoading(SparkFormHandling.class, "/");
   
       Spark.get("/", (request, response) -> {
         Map<String, Object> fruitsMap = new HashMap<>();
         fruitsMap.put("fruits", Arrays.asList("apple", "orange", "banana", "peach"));
         try {
           Template fruitPickerTemplate = configuration.getTemplate("fruitPicker.ftl");
           StringWriter writer = new StringWriter();
           fruitPickerTemplate.process(fruitsMap, writer);
           return writer;
         } catch (Exception e) {
           halt(500);
           return null;
         }
       });
   
       Spark.post("/favorite_fruit", (request, response) -> {
         String fruit = request.queryParams("fruit");
         if (null == fruit) {
           return "Why don't you pick one?";
         } else {
           return "Your favorite fruit is " + fruit;
         }
       });
       
     }
   }
   ```

2. Add fruitPicke.ftl

   ```html
   <html>
     <head><title>Fruit Picker</title></head>
     <body>
       <form action="/favorite_fruit" method="post">
         <p>What is your favorite fruit?</p>
         <#list fruits as fruit>
           <p>
             <input type="radio" name="fruit" value="${fruit}">${fruit}</input>
           </p>
         </#list>
         <input type="submit" value="Submit"/>
       </form>
     </body>
   </html>
   ```


#### Introduction to Our Class Project, The Blog

![mongoDB](/img/mongoDB/Java_Developers/blog.jpg)



#### Blog in Relational Tables

![mongoDB](/img/mongoDB/Java_Developers/relational_tables.jpg)

**关系型数据库表结构如下：**

1.POSTS 文章表

| 字段      | 类型   | 备注 |
| --------- | ------ | ---- |
| post_id   | id     |      |
| auther_id | id     |      |
| title     | string |      |
| post      |        |      |
| date      |        |      |

2.COMMNETS 评论表

| 字段       | 类型   | 备注 |
| ---------- | ------ | ---- |
| comment_id | id     |      |
| name       | string |      |
| comment    | string |      |
| email      | string |      |

3.TATG 标签表

| 字段   | 类型   | 备注 |
| ------ | ------ | ---- |
| tag_id | id     |      |
| name   | string |      |

4.POST_TAGS 文章与标签的关系表

| 字段    | 类型 | 备注 |
| ------- | ---- | ---- |
| post_id | id   |      |
| tag_id  | id   |      |

5.POST_COMMENTS 文章与评论的关系表

| 字段       | 类型 | 备注 |
| ---------- | ---- | ---- |
| post_id    | id   |      |
| comment_id | id   |      |

6.AUTHORS 作者表

| 字段      | 类型   | 备注 |
| --------- | ------ | ---- |
| author_id | id     |      |
| username  | string |      |
| passowork | string |      |

**Problem:**

let's assume that our blog can be modeled with the following relational tables:

```
authors:
    author_id,
    name,
    email,
    password

posts:
    post_id,
    author_id
    title,
    body,
    publication_date

comments:
    comment_id,
    name,
    email,
    comment_text

post_comments:
    post_id,
    comment_id


tags
    tag_id
    name

post_tags
    post_id
    tag_id
```

In order to display a blog post with its comments and tags, how many tables will need to be accessed?

answer: 6 个表



#### Blog in Documents

![mongoDB](/img/mongoDB/Java_Developers/blog_document.jpg)

Given the document schema that we proposed for the blog, how many collections would we need to access to display the blog home page?

answer: 1



#### Introduction to Schema Design

![mongoDB](/img/mongoDB/Java_Developers/schema_design.jpg)

In which scenario is it impossible to embed data within a document (you must put the data in a separate collection)?

answer: The embedded data could exceed the 16MB document limit within MongoDB.



#### Homework 1.1

```shell
$ mongorestore dump
2018-10-16T18:37:44.824+0800	preparing collections to restore from
2018-10-16T18:37:44.825+0800	reading metadata for m101.funnynumbers from dump/m101/funnynumbers.metadata.json
2018-10-16T18:37:44.825+0800	reading metadata for m101.hw1 from dump/m101/hw1.metadata.json
2018-10-16T18:37:44.857+0800	restoring m101.hw1 from dump/m101/hw1.bson
2018-10-16T18:37:44.891+0800	restoring m101.funnynumbers from dump/m101/funnynumbers.bson
2018-10-16T18:37:44.892+0800	no indexes to restore
2018-10-16T18:37:44.892+0800	finished restoring m101.hw1 (1 document)
2018-10-16T18:37:44.893+0800	no indexes to restore
2018-10-16T18:37:44.893+0800	finished restoring m101.funnynumbers (100 documents)
2018-10-16T18:37:44.893+0800	done

$ mongo
$ show dbs
$ use m101
$ show collections
$ db.hw1.find()
{ "_id" : ObjectId("50773061bf44c220307d8514"), "answer" : 42, "question" : "The Ultimate Question of Life, The Universe and Everything" }
answer: 42
```



#### Homework 1.2

Which of the following are valid JSON documents? Please choose all that apply.

```json
{ "title" : "Star Wars", "quotes" : [ "Use the Force", "These are not the droids you are looking for" ], "director" : "George Lucas" }

{}

{ "a" : 1, "b" : { "b" : 1, "c" : "foo", "d" : "bar", "e" : [1, 2, 4] } }
```



Answer
There are no equal signs, nor are there semicolons in JSON*. It uses colons and commas, respectively.

Everything else here is valid.

* Unless, of course, they are contained within a string.



#### Homework 1.3

```shell
mvn compile exec:java -Dexec.mainClass=com.mongodb.SparkHomework

The answer is: 523258
```





### Week 2: CRUD

> Mongo Shell, Query Operators, Update Operators and a Few Commands

