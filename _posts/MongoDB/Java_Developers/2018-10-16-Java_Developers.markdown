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

> MongoDB Documentation: https://docs.mongodb.com



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

#### [Creating Documents](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/index.html)

`db.collection.insertOne()` : Inserts a document into a collection.

The [`insertOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#db.collection.insertOne) method has the following syntax:

```
db.collection.insertOne(
   <document>,
   {
      writeConcern: <document>
   }
)

```

insertOne 练习：

```shell
# 使用 db.collection.insertOne() 插入一条文档记录，如果集合尚未存在，则创建集合
db.moviesScratch.insertOne({"title":"Rockey","year":"1976","imdb":"tt0075148"})

# MongoDB中的所有文档都会自动生成包含 _id (下划线的 id 唯一标识符)
# 也可以手动插入 _id （作为唯一标识符）

MongoDB Enterprise > db.moviesScratch.insertOne({"title": "Rocky","year": "1976"})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5bc6d2c0be888622623df40f")
}
MongoDB Enterprise > db.moviesScratch.insertOne({"title": "Rocky","year": "1976","_id": "tt0075148"})
{ "acknowledged" : true, "insertedId" : "tt0075148" }
MongoDB Enterprise > db.moviesScratch.find().pretty()
{
	"_id" : ObjectId("5bc6d2c0be888622623df40f"),
	"title" : "Rocky",
	"year" : "1976"
}
{ "_id" : "tt0075148", "title" : "Rocky", "year" : "1976" }
MongoDB Enterprise >
```



`db.collection.insertMany()` : Inserts multiple documents into a collection.

The [`insertMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany) method has the following syntax:

```
db.collection.insertMany(
   [ <document 1> , <document 2>, ... ],
   {
      writeConcern: <document>,
      ordered: <boolean>
   }
)
```

insertMany 练习：

```shell
# 批量插入
# ordered: true 代表默认执行有序插入，一旦遇到错误，它将停止插入文件
# ordered: false 代表默认执行无序插入，一旦遇到错误，它将忽略继续插入文件
db.moviesScratch.insertMany(
    [
        {
	    "_id" : "tt0084726",
	    "title" : "Star Trek II: The Wrath of Khan",
	    "year" : 1982,
	    "type" : "movie"
        },
        {
	    "_id" : "tt0796366",
	    "title" : "Star Trek",
	    "year" : 2009,
	    "type" : "movie"
        },
        {
	    "_id" : "tt0084726",
	    "title" : "Star Trek II: The Wrath of Khan",
	    "year" : 1982,
	    "type" : "movie"
        },
        {
	    "_id" : "tt1408101",
	    "title" : "Star Trek Into Darkness",
	    "year" : 2013,
	    "type" : "movie"
        },
        {
	    "_id" : "tt0117731",
	    "title" : "Star Trek: First Contact",
	    "year" : 1996,
	    "type" : "movie"
        }
    ],
    {
        "ordered": false 
    }
);
```

> 当 db.collection.updateOne() 或 db.collection.updateMany() 设置了 **upsert: true**时也可能触发新增文档



#### The ` _id` Field

MongoDB，如果我们不提供，则创建 `_id` 字段

默认情况下，所有集合都有唯一的主索引在 `_id`  字段上

默认情况下，MongoDB 为 `_id` 字段创建值，类型为 `ObjectId` ，这是 BSON 规范定义的值类型

`ObjectId` 值都是12字节的十六进制字符串，前4个字节表示秒的值（自 Unix 时代的时间戳），以下的3个字符串是 MAC ADDR 机器标识符（它们是 MongoDB 所在机器的 Mac 地址），然后2个字节包含 ProcessID，最后3个字节是一个计数器。 确保所有 ObjectID都是唯一的，这就是 ObjectID 的构造方式。

Returns a new [ObjectId](https://docs.mongodb.com/manual/reference/bson-types/#objectid) value. The 12-byte [ObjectId](https://docs.mongodb.com/manual/reference/bson-types/#objectid) value consists of:

- a 4-byte value representing the seconds since the Unix epoch,
- a 5-byte random value, and
- a 3-byte counter, starting with a random value.



#### Reading Documents

`db.collection.find`(*query*, *projection*)

Selects documents in a collection or view and returns a [cursor](https://docs.mongodb.com/manual/reference/glossary/#term-cursor) to the selected documents.

查找文档练习：

```shell
db.movieDetails.find({"rated": "PG-13"}).pretty()

# 识别嵌套文档的字段查询(通过使用点表示法将字段名称串联在一起，不要忘记引号)
db.movieDetails.find({"tomato.meter": 100}).pretty()
```



使用 count 命令统计查询返回的文档数量结果

Counts the number of documents referenced by a cursor. Append the [`count()`](https://docs.mongodb.com/manual/reference/method/cursor.count/index.html#cursor.count) method to a [`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find) query to return the number of matching documents. The operation does not perform the query but instead counts the results that would be returned by the query.

```shell
db.movieDetails.find({"rated": "PG-13"}).count()
```



Equality Matches On Arrays(数组上的相等匹配)

- On The Entire Array(匹配整个数组)
- Based On Any Element(匹配基于数据中的任何元素)
- Based On A Specific Element(匹配特定元素，比如只匹配第一个元素匹配的数组)
- More Complex Matches Osing Operators(更复杂的匹配操作)

数组匹配练习：

```shell
# 匹配数组中有两个元素，且按照顺序
db.movieDetails.find({"writers": ["Ethan Coen", "Joel Coen"]}).count()

# 匹配数组中的一个元素，无序
db.movieDetails.find({"actors": "Jeff Bridges"}).pretty()

# 匹配数组中的第一个元素
db.movieDetails.find({"actors.0": "Jeff Bridges"}).pretty()
```



[Cursor 游标](https://docs.mongodb.com/manual/tutorial/iterate-a-cursor/index.html)

```shell
db.movieDetails.find({"rated": "PG"})
# cursor.next 将执行更多遍历操作
Type "it" for more 
> it
> it

# 关注游标在 shell 中的工作方式
# 分配游标，使用“var”关键字变量
var c = db.movieDetails.find();
# 创建一个函数使用此光标，首先检查是否存在
var doc = function() { return c.hasNext() ? c.next: null;}
# cursor.objsLeftInBatch() returns the number of documents remaining in the current batch. 返回当前批次中剩余的文档数量
c.objsLeftInBatch(); # 101
doc();
doc();
doc();
doc();
# 返回当前批次中剩余的文档数量为97
c.objsLeftInBatch();
```



Projection 投影是一种减少方便的方法为任何一个查询返回的数据大小。预测可以减少网络开销和处理，限制返回字段的要求。

```shell
# 现在不是返回所有匹配的文档的所有字段，只会返回 _id,title 两个字段。默认情况下始终返回 _id 字段
db.movieDetails.find({"rated": "PG"}, {title: 1}).pretty()

# 通过简单地为 id 指定0来实现排除
db.movieDetails.find({"rated": "PG"},{title: 1, _id: 0}).pretty()

# 通过明确排除的字段来达到其他字段返回的写法
db.movieDetails.find({"rated": "PG"},{writers: 0, actors:0, _id: 0}).pretty()
```



#### [Comparison Operators](https://docs.mongodb.com/manual/reference/operator/aggregation/)

For comparison of different BSON type values, see the [specified BSON comparison order](https://docs.mongodb.com/manual/reference/bson-type-comparison-order/#bson-types-comparison-order).

| Name                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`$eq`](https://docs.mongodb.com/manual/reference/operator/query/eq/#op._S_eq) | Matches values that are equal to a specified value.          |
| [`$gt`](https://docs.mongodb.com/manual/reference/operator/query/gt/#op._S_gt) | Matches values that are greater than a specified value.      |
| [`$gte`](https://docs.mongodb.com/manual/reference/operator/query/gte/#op._S_gte) | Matches values that are greater than or equal to a specified value. |
| [`$in`](https://docs.mongodb.com/manual/reference/operator/query/in/#op._S_in) | Matches any of the values specified in an array.             |
| [`$lt`](https://docs.mongodb.com/manual/reference/operator/query/lt/#op._S_lt) | Matches values that are less than a specified value.         |
| [`$lte`](https://docs.mongodb.com/manual/reference/operator/query/lte/#op._S_lte) | Matches values that are less than or equal to a specified value. |
| [`$ne`](https://docs.mongodb.com/manual/reference/operator/query/ne/#op._S_ne) | Matches all values that are not equal to a specified value.  |
| [`$nin`](https://docs.mongodb.com/manual/reference/operator/query/nin/#op._S_nin) | Matches none of the values specified in an array.            |



比较运算符查询练习

```shell
# 运行时间大于90分钟
db.movieDetails.find({"runtime": {$gt: 90}}).pretty()
    
# 只显示电影标题和运行时间字段
db.movieDetails.find({"runtime": {$gt: 90}}, {title: 1, runtime: 1, _id: 0}).pretty();

# 运行时间大于等于90分钟，小于等于120分钟
db.movieDetails.find({"runtime": {$gte: 90, $lte: 120}}).pretty();

# 混合使用组合查询
db.movieDetails.find({"tomato.meter": {$gte: 95}, runtime: {$gt: 180}}, {title: 1, runtime: 1, _id: 0}).pretty();

# 不等于"未评级"的记录
db.movieDetails.find({"rated": {$ne: "UNRATED"}}).count()

# $in的值必须是数组，即将返回这三个值中的任何一个
db.movieDetails.find({"rated": {$in: ["G", "PG", "PG-13"]}}).pretty()

# $nin 是 $in 的反面，不存在
```



#### Element Operators

| Name                                                         | Description                                            |
| ------------------------------------------------------------ | ------------------------------------------------------ |
| [`$exists`](https://docs.mongodb.com/manual/reference/operator/query/exists/#op._S_exists) | Matches documents that have the specified field.       |
| [`$type`](https://docs.mongodb.com/manual/reference/operator/query/type/#op._S_type) | Selects documents if a field is of the specified type. |

练习

```shell
# $exists 匹配字段存在或不存在的记录
db.movieDetails.find({"tomato.meter": {$exists: true}})

# $type 匹配字段的类型
# 匹配字符串类型的字段
db.movieDetails.find({"_id": {$type: "string"}}).pretty()
```



#### Logical Operators

| Name                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`$and`](https://docs.mongodb.com/manual/reference/operator/query/and/#op._S_and) | Joins query clauses with a logical `AND` returns all documents that match the conditions of both clauses. |
| [`$not`](https://docs.mongodb.com/manual/reference/operator/query/not/#op._S_not) | Inverts the effect of a query expression and returns documents that do *not* match the query expression. |
| [`$nor`](https://docs.mongodb.com/manual/reference/operator/query/nor/#op._S_nor) | Joins query clauses with a logical `NOR` returns all documents that fail to match both clauses. |
| [`$or`](https://docs.mongodb.com/manual/reference/operator/query/or/#op._S_or) | Joins query clauses with a logical `OR` returns all documents that match the conditions of either clause. |

练习

```shell
# 或的限制
# $or 将数组作为参数，我们在其中指定标准
db.movieDetails.find({
	$or: [{
		"tomato.meter": {
			$gt: 95
		}
	}, {
		"metacritic": {
			$gt: 88
		}
	}]
}).pretty()

# 和（并且）的限制
# $and 将数组作为参数，我们在其中指定标准
db.movieDetails.find({
	$and: [{
		"tomato.meter": {
			$gt: 95
		}
	}, {
		"metacritic": {
			$gt: 88
		}
	}]
}).pretty()
# 相当于隐式的查询
db.movieDetails.find({
		"tomato.meter": {
			$gt: 95
		}
	}, {
		"metacritic": {
			$gt: 88
		}
}).prety()
## $and 的特别用处，同一个字段设置多个条件
# 有效值查询：metacritic 不等于 null，并且要存在metacritic字段
db.movieDetails.find({
	$and: [{
		"metacritic": {
			$ne: null
		}
	}, {
		"metacritic": {
			$exists: true
		}
	}]
}).pretty()
```



#### Regex Operator

| Name                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`$expr`](https://docs.mongodb.com/manual/reference/operator/query/expr/#op._S_expr) | Allows use of aggregation expressions within the query language. |
| [`$jsonSchema`](https://docs.mongodb.com/manual/reference/operator/query/jsonSchema/#op._S_jsonSchema) | Validate documents against the given JSON Schema.            |
| [`$mod`](https://docs.mongodb.com/manual/reference/operator/query/mod/#op._S_mod) | Performs a modulo operation on the value of a field and selects documents with a specified result. |
| [`$regex`](https://docs.mongodb.com/manual/reference/operator/query/regex/#op._S_regex) | Selects documents where values match a specified regular expression. |
| [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#op._S_text) | Performs text search.                                        |
| [`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#op._S_where) | Matches documents that satisfy a JavaScript expression.      |

允许我们使用正则表达式来匹配字段

```shell
# 使用正则表达匹配字符串
# 以赢得一词开头
db.movieDetails.find({"awards.text": {$regex: /^Won\s.*/}})
```



#### Array Operators

| Name                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`$all`](https://docs.mongodb.com/manual/reference/operator/query/all/#op._S_all) | Matches arrays that contain all elements specified in the query. |
| [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#op._S_elemMatch) | Selects documents if element in the array field matches all the specified [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#op._S_elemMatch)conditions. |
| [`$size`](https://docs.mongodb.com/manual/reference/operator/query/size/#op._S_size) | Selects documents if the array field is a specified size.    |

练习

```shell
# $all 用于匹配数组字段，必须全部包含在数组元素中
db.movieDetails.find({
	"genres": {
		$all: ["Comedy", "Crime", "Drana"]
	}
}).pretty();

# $size 匹配文档数组的长度
db.movieDetails.find({"countries": {$size: 1}}).pretty()
 
# 要对子文档或数组项执行查询，必须使用点符号。
# 匹配数组中的每个元素选择文档，如果数组字段中的元素匹配所有指定的$elemMatchconditions。
db.movieDetails.find({
	boxOffice: {
		$elemMatch: {
			country: "UK",
			revenue: {
				$gt: 15
			}
		}
	}
}).pretty()

```



#### Updating Documents

有一些情况，更新实际上可以创建文档

MongoDB提供了三种不同的更新命令

- Update One
- Update Many
- Replace One



`db.collection.updateOne`(*filter*, *update*, *options*)

Updates a single document within the collection based on the filter.

The [`updateOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/index.html#db.collection.updateOne) method has the following form:

```
db.collection.updateOne(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)
```

updateOne 练习

```shell
# 第一个参数，首先制定过滤器或选择器，这将标识要更新的文档文档
# 第二个参数，然后使用更新运算符指定要更新的内容

# 更新匹配我们选择题的第一个
db.movieDetails.updateOne({"title": "The Martian"}, {$set, {"poster": "http://www.baidu.com"}});

# $set 更新字段，若不存在则增加此字段
# $unset 将彻底消除这个字段
# $min 取两个值的最小值
# $max 取最大值
# $inc 增量运算符

db.movieDetails.updateOne({
	"title": "The Martian"
}, {
	$inc: {
		"tomato.reviews": 3,
		"tomato.userReviews": 25
	}
})

# $addToSet 仅当元素不存在于集合中时，才向数组中添加元素。
# $push 增加一个元素到数组中
db.movieDetails.updateOne({
	"title": "The Martian"
}, {
	$push: {
		reviews: {
			ratting: 4.5,
			date: ISODate("2018-10-17T17:44:34Z"),
			reviewer: "Spencer H.",
			text: "some text"
		}
	}
});
```

Update Operators

| Name                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`$currentDate`](https://docs.mongodb.com/manual/reference/operator/update/currentDate/#up._S_currentDate) | Sets the value of a field to current date, either as a Date or a Timestamp. |
| [`$inc`](https://docs.mongodb.com/manual/reference/operator/update/inc/#up._S_inc) | Increments the value of the field by the specified amount.   |
| [`$min`](https://docs.mongodb.com/manual/reference/operator/update/min/#up._S_min) | Only updates the field if the specified value is less than the existing field value. |
| [`$max`](https://docs.mongodb.com/manual/reference/operator/update/max/#up._S_max) | Only updates the field if the specified value is greater than the existing field value. |
| [`$mul`](https://docs.mongodb.com/manual/reference/operator/update/mul/#up._S_mul) | Multiplies the value of the field by the specified amount.   |
| [`$rename`](https://docs.mongodb.com/manual/reference/operator/update/rename/#up._S_rename) | Renames a field.                                             |
| [`$set`](https://docs.mongodb.com/manual/reference/operator/update/set/#up._S_set) | Sets the value of a field in a document.                     |
| [`$setOnInsert`](https://docs.mongodb.com/manual/reference/operator/update/setOnInsert/#up._S_setOnInsert) | Sets the value of a field if an update results in an insert of a document. Has no effect on update operations that modify existing documents. |
| [`$unset`](https://docs.mongodb.com/manual/reference/operator/update/unset/#up._S_unset) | Removes the specified field from a document.                 |

Array Operators

| Name                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`$addToSet`](https://docs.mongodb.com/manual/reference/operator/update/addToSet/#up._S_addToSet) | Adds elements to an array only if they do not already exist in the set. |
| [`$pop`](https://docs.mongodb.com/manual/reference/operator/update/pop/#up._S_pop) | Removes the first or last item of an array.                  |
| [`$pull`](https://docs.mongodb.com/manual/reference/operator/update/pull/#up._S_pull) | Removes all array elements that match a specified query.     |
| [`$push`](https://docs.mongodb.com/manual/reference/operator/update/push/#up._S_push) | Adds an item to an array.                                    |
| [`$pullAll`](https://docs.mongodb.com/manual/reference/operator/update/pullAll/#up._S_pullAll) | Removes all matching values from an array.                   |

Modifiers

| Name                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`$each`](https://docs.mongodb.com/manual/reference/operator/update/each/#up._S_each) | Modifies the [`$push`](https://docs.mongodb.com/manual/reference/operator/update/push/#up._S_push) and [`$addToSet`](https://docs.mongodb.com/manual/reference/operator/update/addToSet/#up._S_addToSet) operators to append multiple items for array updates. |
| [`$position`](https://docs.mongodb.com/manual/reference/operator/update/position/#up._S_position) | Modifies the [`$push`](https://docs.mongodb.com/manual/reference/operator/update/push/#up._S_push) operator to specify the position in the array to add elements. |
| [`$slice`](https://docs.mongodb.com/manual/reference/operator/update/slice/#up._S_slice) | Modifies the [`$push`](https://docs.mongodb.com/manual/reference/operator/update/push/#up._S_push) operator to limit the size of updated arrays. |
| [`$sort`](https://docs.mongodb.com/manual/reference/operator/update/sort/#up._S_sort) | Modifies the [`$push`](https://docs.mongodb.com/manual/reference/operator/update/push/#up._S_push) operator to reorder documents stored in an array. |



`db.collection.updateMany`(*filter*, *update*, *options*)

Updates multiple documents within the collection based on the filter.

The [`updateMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/index.html#db.collection.updateMany) method has the following form:

```
db.collection.updateMany(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)
```

匹配过滤器完成批量更新

```shell
# 查看字段值为 null的记录
db.movieDetails.find({rated: null}).count()
# 查看字段值为 UNRATED 的记录
db.movieDetails.find({rated:"UNRATED"}).count()

# 查询 rated为 null的记录，更新为 UNRATED
db.movieDetails.updateMany({"rated": null}, {$set: {"rated": "UNRATED"}});

# 查询 rated 为 null 的记录，并将删除它们
db.movieDetails.updateMany({"rated": null}, {$unset: {"rated"}})
```

Upserts: 如果找不到与我们的过滤器匹配的文档，插入一个新文件

```shell
# detail 为 定义的一个 var 对象
# {upsert: true} 如果这个过滤器不匹配，将继续插入它
# 你想要对文档进行某种类型的更新，但是如果没有该文件，你想继续使用创建一个新的
db.movieDetails.updateOne({"imdb.id": detail.imdb.id}, {$set: detail}, {upsert: true});
```



`db.collection.replaceOne`(*filter*, *replacement*, *options*)

Replaces a single document within the collection based on the filter.

The [`replaceOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.replaceOne/index.html#db.collection.replaceOne) method has the following form:

```
db.collection.replaceOne(
   <filter>,
   <replacement>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>
   }
)
```

与更新命令一样，Replace One 采用过滤器

```shell
db.movies.replaceOne({"imdb":detail.imdb.id}, detail)
```



#### The MongoDB Java Driver

