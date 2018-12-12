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

# 去掉 shell 中用户名前缀
# export PS1='$ '

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

![mongoDB](/img/mongoDB//Java_Developers/blog_document.jpg)

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

1. 增加依赖

   ```xml
   <dependency>
       <groupId>org.mongodb</groupId>
       <artifactId>mongo-java-driver</artifactId>
       <version>3.8.2</version>
   </dependency>
   ```

2. 查看 API

   ```java
   public class App {
   
     public static void main(String[] args) {
       // 默认构造函数连接的是本机 localhost 27107 端口
       // MongoClient client = new MongoClient();
   
       // 使用 MongoClient(String host, int port) 有参构造函数
       // MongoClient client = new MongoClient("localhost", 27107);
   
       // 使用 MongoClient(ServerAddress addr) 有参构造函数
       // MongoClient client = new MongoClient(new ServerAddress("localhost", 27107));
   
       // 使用 MongoClient(List<ServerAddress> seeds) 有参构造函数连接到 MongoDB 集群
       // MongoClient client = new MongoClient(Arrays.asList(new ServerAddress("localhost", 27107)));
   
       // 使用 MongoClient(MongoClientURI uri) 连接字符串来连接
       // MongoClient client = new MongoClient(new MongoClientURI("mongodb://localhost:27017"));
   
       // 使用 MongoClient(final ServerAddress addr, final MongoClientOptions options) 构造连接，设置每个主机的最大连接数。
       MongoClientOptions options = MongoClientOptions.builder().connectionsPerHost(500).build();
       MongoClient client = new MongoClient(new ServerAddress(), options);
   
       // 获得数据库
       MongoDatabase database = client.getDatabase("test");
   
       // 获得 collection
       MongoCollection<Document> collection = database.getCollection("test");
       MongoCollection<BsonDocument> collection1 = database.getCollection("test", BsonDocument.class);
     }
   }
   ```


#### Java driver, document representation

练习

```java
public class DocumentTest {

  public static void main(String[] args) {

    // 不是类型安全的
    Document document = new Document()
        .append("str", "Hello MongoDB")
        .append("int", 42)
        .append("l", 1L)
        .append("double", 1.1)
        .append("b", false)
        .append("date", new Date())
        .append("ObjectId", new ObjectId())
        .append("null", null)
        .append("embeddedDoc", new Document("x", 0))
        .append("list", Arrays.asList(1, 2, 3));

    String str = document.getString("str");
    Integer integer = document.getInteger("int");
    Helpers.printJson(document);

    // 推荐使用 BsonDocument
    BsonDocument bsonDocument = new BsonDocument("str", new BsonString("MongoDB, HELLO"));
  }
}

public class Helpers {

  public static void printJson(Document document) {
//    JsonWriterSettings.builder().outputMode(JsonMode.SHELL).build()
//    new JsonWriterSettings(JsonMode.SHELL, false)
    JsonWriter jsonWriter = new JsonWriter(new StringWriter(),
        JsonWriterSettings.builder().outputMode(JsonMode.SHELL).indent(true).build());
    new DocumentCodec().encode(jsonWriter, document,
        EncoderContext.builder().isEncodingCollectibleDocument(true).build());
    System.out.println(jsonWriter.getWriter());
    System.out.flush();
  }
}
```

Quiz:

How would you create a document using the Java driver with this JSON structure:

```json
{
   "_id" : "user1",
   "interests" : [ "basketball", "drumming"]
}
```

Choose the best answer:

```java
new Document("_id", "user1").append("interests", Arrays.asList("basketball", "drumming"));
```



#### Java Driver: Insert

练习

```java
public class InsertTest {

  public static void main(String[] args) {
    // 连接本机 mongodb
    MongoClient client = new MongoClient();
    // 获取 course 数据库
    MongoDatabase clientDatabase = client.getDatabase("course");
    // 获取 insertTest 实例
    MongoCollection<Document> mongoCollection = clientDatabase.getCollection("insertTest");
    // 删除 insertTest 集合
    mongoCollection.drop();
    // 创建一份文档
    Document smith = new Document("name", "Smith")
        .append("age", 30)
        .append("profession", "programer");
    // 使用 insertOne 插入集合中
    // mongoCollection.insertOne(smith);

    // 创建另外一份文档
    Document jones = new Document("name", "Jones")
        .append("age", 25)
        .append("profession", "hacker");

    // 使用 insertMany 完成批量插入
    mongoCollection.insertMany(Arrays.asList(smith, jones));

    Helpers.printJson(smith);
    Helpers.printJson(jones);
  }

}
```

Quiz:

Do you expect the second insert below to succeed?

```java
MongoClient client = new MongoClient();
MongoDatabase database = client.getDatabase("school");
MongoCollection<Document> people = database.getCollection("people");

Document doc = new Document("name", "Andrew Erlichson").append("company", "10gen");

 people.insertOne(doc);      // first insert
 doc.remove("_id");             // remove the _id key
 people.insertOne(doc);      // second insert
```



Yes, because the remove call will remove the `_id` field added by the driver in the first insert. 是的，因为remove调用将删除驱动程序在第一个插入中添加的_id字段。



#### Java Driver: Find, FindOne,Count

练习

```java
public class FindTest {

  public static void main(String[] args) {
    MongoClient client = new MongoClient();
    MongoDatabase mongoDatabase = client.getDatabase("course");
    MongoCollection<Document> mongoCollection = mongoDatabase.getCollection("findTest");
    mongoCollection.drop();

    // insert 10 Documents
    for (int i = 0; i < 10; i++) {
      mongoCollection.insertOne(new Document("x", i));
    }

    System.out.println("Find One:");
    Document first = mongoCollection.find().first();
    Helpers.printJson(first);

    System.out.println("Find All with into:");
    ArrayList<Document> documents = mongoCollection.find().into(new ArrayList<Document>());
    for (Document cur : documents) {
      Helpers.printJson(cur);
    }

    System.out.println("Find All with iteration:");
    MongoCursor<Document> cursor = mongoCollection.find().iterator();
    try {
      while (cursor.hasNext()) {
        Document document = cursor.next();
        Helpers.printJson(document);
      }
    } finally {
      cursor.close();
    }

    System.out.println("count:");
    long countDocuments = mongoCollection.countDocuments();
    System.out.println(countDocuments);
  }
}
```

Quiz:

In the following code snippet:

```java
MongoClient client = new MongoClient();
MongoDatabase database = client.getDatabase("school");
MongoCollection<Document> people = database.getCollection("people");
Document doc;
// xxxx
System.out.println(doc);
```

Please enter the simplest one line of Java code that would be needed in place of // xxxx to make it print one document from the people collection.

Enter answer here:

```java
doc = people.find().first();
```



#### Java Driver: Querying with a filter

> 修改日志等级，显示更详细的信息 
>
> https://docs.mongodb.com/manual/reference/parameters/#param.logLevel

```shell
db.adminCommand( { setParameter: 1, logLevel: 2 } )

#查看状态：级别和时间
PRIMARY> db.getProfilingStatus()
{ "was" : 1, "slowms" : 200 }
#查看级别
PRIMARY> db.getProfilingLevel()
1
#设置级别
PRIMARY> db.setProfilingLevel(2)
{ "was" : 1, "slowms" : 100, "ok" : 1 }
#设置级别和时间
PRIMARY> db.setProfilingLevel(1,200)
{ "was" : 2, "slowms" : 100, "ok" : 1 }

mongod -f mongodb.conf
```

练习使用 Java 驱动程序应用查询过滤器的方式

```java
public class FindWithFilterTest {

  public static void main(String[] args) {
    MongoClient client = new MongoClient();
    MongoDatabase course = client.getDatabase("course");
    MongoCollection<Document> collection = course.getCollection("findWithFilterTest");
    collection.drop();

    // insert 10 documents with two random integers
    for (int i = 0; i < 10; i++) {
      collection.insertOne(new Document().append("x", new Random().nextInt(2))
          .append("y", new Random().nextInt(100)));
    }

//    Bson fileter = new Document().append("x", 0).append("y", new Document("$gt", 10));
    Bson filter = and(eq("x", 0), gt("y", 10), lt("y", 90));

    ArrayList<Document> documents = collection.find(filter).into(new ArrayList<Document>());
    for (Document cur : documents) {
      Helpers.printJson(cur);
    }

    long count = collection.countDocuments(filter);
  }
}
```

Quiz:

Given a collection named "scores" of documents with two fields -- type and score -- what is the correct line of code to find all documents where type is "quiz" and score is greater than 20 and less than 90. Select all that apply.

```java
scores.find(new Document("type", "quiz").append("score", new Document("$gt", 20).append("$lt", 90)));

scores.find(Filters.and(Filters.eq("type", "quiz"), Filters.gt("score", 20), Filters.lt("score", 90)))
```



#### Java Driver: Querying with a Projection

如何使用字段来投影，如何使用投影构建器中静态方法

```java
public class FindWithFilterTest {

  public static void main(String[] args) {
    MongoClient client = new MongoClient();
    MongoDatabase course = client.getDatabase("course");
    MongoCollection<Document> collection = course.getCollection("findWithFilterTest");
    collection.drop();

    // insert 10 documents with two random integers
    for (int i = 0; i < 10; i++) {
      collection.insertOne(new Document().append("x", new Random().nextInt(2))
          .append("y", new Random().nextInt(100)));
    }

//    Bson fileter = new Document().append("x", 0).append("y", new Document("$gt", 10));
    Bson filter = and(eq("x", 0), gt("y", 10), lt("y", 90));

    // 排除字段 x 的输出
//    Bson projection = new Document("x", 0);

    // 排序字段 x 和 _id 的输出
//    Bson projection = new Document("x", 0).append("_id", 0);

    // 允许字段 x 和 y 的输出，但是默认为输出 _id，如若不要请声明去除
//    Bson projection = new Document("x", 1).append("y", 1);

    // 使用 Projections.exclude 排除字段
//    Bson projection = Projections.exclude("x", "_id");

    // 使用 Projections.include 包含字段
//    Bson projection = Projections.include("y");

    // 使用 Projections.fields组合来同时包含字段和排除字段操作
//    Bson projection = Projections.fields(Projections.exclude("_id"), Projections.include("y"));

    // 使用 Projections.fields组合来同时包含字段和使用 Projections.excludeId() 排除 _id 字段
//    Bson projection = Projections.fields(Projections.include("x"), Projections.excludeId());

    // 通过导入静态方法，简化代码
    Bson projection = fields(include("x"), excludeId());


    ArrayList<Document> documents = collection.find(filter).projection(projection).into(new ArrayList<Document>());
    for (Document cur : documents) {
      Helpers.printJson(cur);
    }

    long count = collection.countDocuments(filter);
  }
}
```



Quiz:

Given a variable named "students" of type MongoCollection<Document>, which of the following lines of code could be used to find all documents in the collection, retrieving only the "phoneNumber" field.

```java
students.find().projection(Projections.fields(Projections.include("phoneNumber"), Projections.excludeId());
                           
students.find().projection(new Document("phoneNumber", 1).append("_id", 0));
```



#### Java Driver: Querying with Sort, Skip and Limit

排序、跳过、限制的练习

```java
public class FindWithSortSkipLimitTest {

  public static void main(String[] args) {
    MongoClient client = new MongoClient();
    MongoDatabase database = client.getDatabase("course");
    MongoCollection<Document> collection = database
        .getCollection("findWithSortSkipLimitTest");
    collection.drop();

    // insert 100 documents with two random integers
    for (int i = 0; i < 10; i++) {
      for (int j = 0; j < 10; j++) {
        collection.insertOne(new Document().append("i", i).append("j", j));
      }
    }

    Bson projection = Projections.fields(Projections.include("i", "j"), Projections.excludeId());
    // 【排序】
    // 按照 i 字段升序排列
    // Bson sort = new Document("i", 1);

    // 按照 i 字段、j 字段升序排列
    // Bson sort = new Document("i", 1).append("j", 1);
    // 按照 i 字段升序、j 字段降序排列
    // Bson sort = new Document("i", 1).append("j", -1);

    // 通过静态方法 Sorts.ascending 指定 i 升序
    // Bson sort = Sorts.ascending("i");

    // 通过静态方法 Sorts.ascending 指定 i 升序，并且通过 Sorts.descending 指定 j 降序
    // Bson sort = Sorts.orderBy(Sorts.ascending("i"), Sorts.descending("j"));

    // 通过 Sorts.descending 指定 i，j 都降序排列，一种复合排序
    Bson sort = Sorts.descending("i", "j");

    List<Document> all = collection.find()
        .projection(projection)
        .sort(sort)
        .skip(20) // 【跳过】跳过前20条
        .limit(50) // 【限制】通过 limit 限制前50条
        .into(new ArrayList<>());

    for (Document cur : all) {
      Helpers.printJson(cur, false);
    }
  }

}
```

Quiz:

Supposed you had the following documents in a collection named things.

```json
{ "_id" : 0, "value" : 10 }
{ "_id" : 1, "value" : 5 }
{ "_id" : 2, "value" : 7 }
{ "_id" : 3, "value" : 20 }
```

If you performed the following query in the Java driver:

```java
collection.find().sort(new Document("value", -1)).skip(2).limit(1)
```

which document would be returned?

The document with _id=2



#### Java Driver:Update and Replace

如何进行替换、更新、批量更新，当更新不存在时变为增加

```java
public class UpdateTest {

  public static void main(String[] args) {
    MongoClient client = new MongoClient();
    MongoDatabase course = client.getDatabase("course");
    MongoCollection<Document> collection = course.getCollection("updateTest");
    collection.drop();

    // insert 8 documents, with both _id and x set to the value of the loop variable
    // and y set to true
    for (int i = 0; i < 8; i++) {
      collection.insertOne(new Document().append("_id", i).append("x", i).append("y", true));
    }

    // 使用 replaceOne 方式进行【替换】更新操作：更新 x 为5的记录，并把 x 改为20，且增加 updated 为 true 的字段，删除 y 的字段
    // collection.replaceOne(Filters.eq("x", 5), new Document("x", 20).append("updated", true));

    // 使用 updateOne 方式进行【更新】操作：更新 x为5的记录，并把 x 改为20，且增加 updated 为 true的字段，忽略其他字段
    // collection.updateOne(Filters.eq("x", 5), new Document("$set", new Document("x", 20).append("updated", true)));

    // 使用 Updates.set 构造器进行更新一个字段
    // collection.updateOne(Filters.eq("x", 5),Updates.set("x", 20));

    // 使用 Updates.set 构造器进行更新多个字段
    // collection.updateOne(Filters.eq("x", 5), Updates.combine(Updates.set("x", 20), Updates.set("updated", true)));

    // 使用 Updates.set 和 upsert(true) 使更新具有新增功能（当记录不存在时）
    // collection.updateOne(Filters.eq("_id", 9), Updates.combine(Updates.set("x", 20), Updates.set("updated", true)), new UpdateOptions().upsert(true));

    // 使用 updateMany 方式进行【批量更新】操作，更新大于等于5的记录，所有的 x 值增量加1
    collection.updateMany(Filters.gte("x", 5), Updates.inc("x", 1));

    for (Document cur: collection.find().into(new ArrayList<>())) {
      Helpers.printJson(cur);
    }
  }
}
```

Quiz

In the following code fragment, what is the Java expression in place of xxxx that will set the field "examiner" to the value "Jones" for the document with _id of 1. Please use the $set operator.

```java
# update using $set
scores.updateOne(new Document("_id", 1), xxxx);
```

Enter answer here:

```java
new Document("$set", new Document("examiner", "Jones"))
```



#### Java Driver:Delete

使用 deleteOne 删除 或 deleteMany 批量删除

```java
public class DeleteTest {

  public static void main(String[] args) {
    MongoClient client = new MongoClient();
    MongoDatabase database = client.getDatabase("course");
    MongoCollection<Document> collection = database.getCollection("deleteTest");
    collection.drop();

    // insert 8 documents, with _id set to the value of the loop variable
    for (int i = 0; i < 8; i++) {
      collection.insertOne(new Document("_id", i));
    }

    // 使用 deleteMany 方法完成批量删除 _id的值大于4的记录
    // collection.deleteMany(Filters.gt("_id", 4));

    // 使用 deleteOne 方法完成删除 _id 的值为4的记录
    collection.deleteOne(Filters.eq("_id", 4));

    for (Document cur : collection.find().into(new ArrayList<>())) {
      Helpers.printJson(cur);
    }
  }
}
```

Quiz

Given a collection with the following documents, how many will be affected by the following deletion statement?

```json
{ _id: 0, x: 1 }
{ _id: 1, x: 1 }
{ _id: 2, x: 1 }
{ _id: 3, x: 2 }
{ _id: 4, x: 2 }
```

```java
collection.deleteOne(Filters.eq("x", 1))
```

Choose the best answer: 1

> 注意，虽然字段x为1的记录有3条，但是 delteOne 只会删除1条



#### All Together Now: MongoDB, Spark and Freemarker

使用 Spark Web 请求框架，结合 MongoDB数据查询填充到 Freemarker 模板中

```java
public class HelloWorldMongoDBSparkFreemarkerStyle {

  public static void main(String[] args) {
    final Configuration configuration = new Configuration();
    configuration.setClassForTemplateLoading(HelloWorldMongoDBSparkFreemarkerStyle.class, "/freemarker");
    MongoClient mongoClient = new MongoClient();
    MongoDatabase database = mongoClient.getDatabase("course");
    MongoCollection<Document> collection = database.getCollection("hello");
    collection.drop();

    collection.insertOne(new Document("name", "MongoDB"));

    Spark.get("/", new Route() {
      @Override
      public Object handle(Request request, Response response) throws Exception {
        StringWriter writer = new StringWriter();
        try {
          Template helloTemplate = configuration.getTemplate("hello.ftl");
          helloTemplate.process(collection.find().first(), writer);
        } catch (Exception e) {
          Spark.halt(500);
          e.printStackTrace();
        }
        return writer;
      }
    });
  }
}
```



#### Blog, Internals

博客的内幕：

- View -> ftl - 由freemarker处理
- Controller/Model - java/spark



#### Blog, Session Management

会话管理及用户的方式

![mongoDB](/img/mongoDB/Java_Developers/session_management.png)



#### Blog, User Interface

包含三个页面

- Author Signup
- Author Login
- Author Logout

![mongoDB](/img/mongoDB/Java_Developers/blog_ui.png)



#### Homework 2.1

```
THE ANSWER IS:2805
```



#### Homework 2.2

```shell
# 导入数据
mongoimport --drop -d students -c grades grades.json

Now it's your turn to analyze the data set. Find all exam scores greater than or equal to 65, and sort those scores from lowest to highest.

What is the student_id of the lowest exam score above 65?

db.grades.find({"score":{$gte:65}}).sort({"score": 1})
22
```



#### Homework 2.3

先写一段 java 程序删除每个学生最低成绩的家庭作业记录

```java
// 这是我自己写的
public class StudentScore {

  public static void main(String[] args) {
    MongoClient mongoClient = new MongoClient();
    MongoDatabase database = mongoClient.getDatabase("students");
    MongoCollection<Document> collection = database.getCollection("grades");

    ArrayList<Document> documents = collection
        .aggregate(asList(new Document("$match", new Document("type", "homework"))
        , new Document("$group", new Document("_id", "$student_id").append("score", new Document("$min", "$score")))
            , new Document("$sort", new Document().append("_id", 1))
        ))
        .into(new ArrayList<>());

    for (Document cur : documents) {
      cur.put("student_id",cur.get("_id"));
      cur.remove("_id");
      collection.deleteOne(cur);
    }

    FindIterable<Document> documents1 = collection.find(Filters.eq("type", "homework"))
        .sort(Sorts.ascending("student_id"));
    for (Document cur : documents1) {
      Helpers.printJson(cur);
    }
  }

}

/*
 * Copyright 2015 MongoDB, Inc. 这是 mongoDB 官方提供的
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *  http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package course.homework.week2;

import com.mongodb.MongoClient;
import com.mongodb.MongoClientURI;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;

import java.io.IOException;

import static com.mongodb.client.model.Filters.eq;
import static com.mongodb.client.model.Sorts.ascending;

public class RemoveLowest {
    public static void main(final String[] args) {
        MongoClient client = new MongoClient();
        MongoDatabase numbersDB = client.getDatabase("students");
        MongoCollection<Document> grades = numbersDB.getCollection("grades");

        MongoCursor<Document> cursor = grades.find(eq("type", "homework"))
                                .sort(ascending("student_id", "score")).iterator();

        Object studentId = -1;
        try {
            while (cursor.hasNext()) {
                Document entry = cursor.next();
                if (!entry.get("student_id").equals(studentId)) {
                    System.out.println("Removing: " + entry);
                    Object id = entry.get("_id");
                    grades.deleteOne(eq("_id", id));

                }
                studentId = entry.get("student_id");
           }
        } finally {
            cursor.close();
        }
    }
}
```

再运行 shell 脚本来计算出答案

```shell
# 导入数据
mongoimport --drop -d students -c grades grades.json

# 进入 mongo shell
mongo

# 选择 students 数据库
use students

# 查看 grades 集合的数量
db.grades.count()

# The result should be 600. Now let us find the student who holds the 101st best grade across all grades:
db.grades.find().sort( { 'score' : -1 } ).skip( 100 ).limit(1)

# The correct result will be:
# { "_id" : ObjectId("50906d7fa3c412bb040eb709"), "student_id" : 100, "type" : "homework", "score" : 88.50425479139126 }

# Now let us sort the students by student_id , and score, while also displaying the type to then see what the top five docs are:
# db.grades.find( { }, { 'student_id' : 1, 'type' : 1, 'score' : 1, '_id' : 0 } ).sort( { 'student_id' : 1, 'score' : 1 } ).limit( 5 )
{ "student_id" : 0, "type" : "quiz", "score" : 31.95004496742112 }
{ "student_id" : 0, "type" : "exam", "score" : 54.6535436362647 }
{ "student_id" : 0, "type" : "homework", "score" : 63.98402553675503 }
{ "student_id" : 1, "type" : "homework", "score" : 44.31667452616328 }
{ "student_id" : 1, "type" : "exam", "score" : 74.20010837299897 }

# To verify that you have completed this task correctly, provide the identity of the student with the highest average in the class with following query that uses the aggregation framework. The answer will appear in the _id field of the resulting document.
db.grades.aggregate( { '$group' : { '_id' : '$student_id', 'average' : { $avg : '$score' } } }, { '$sort' : { 'average' : -1 } }, { '$limit' : 1 } )
{ "_id" : 54, "average" : 96.19488173037341 }
```



#### Homework 2.4(MongoProc)

涉及到修改 `UserDAO.java`

```java
// validates that username is unique and insert into db
public boolean addUser(String username, String password, String email) {

    String passwordHash = makePasswordHash(password, Integer.toString(random.nextInt()));

    Document user = new Document();

    user.append("_id", username).append("password", passwordHash);

    if (email != null && !email.equals("")) {
        // the provided email address
        user.append("email", email);
    }

    try {
        usersCollection.insertOne(user);
        return true;
    } catch (MongoWriteException e) {
        if (e.getError().getCategory().equals(ErrorCategory.DUPLICATE_KEY)) {
            System.out.println("Username already in use: " + username);
            return false;
        }
        throw e;
    }
}


public Document validateLogin(String username, String password) {
    Document user;

    user = usersCollection.find(eq("_id", username)).first();

    if (user == null) {
        System.out.println("User not in database");
        return null;
    }

    String hashedAndSalted = user.get("password").toString();

    String salt = hashedAndSalted.split(",")[1];

    if (!hashedAndSalted.equals(makePasswordHash(password, salt))) {
        System.out.println("Submitted password is not a match");
        return null;
    }

    return user;
}
```



#### Homework 2.5

先导入“Creating Documents”中的video 库

```shell
$ mongorestore dump
```



Which of the choices below is the title of a movie from the year 2013 that is rated PG-13 and won no awards? Please query the `video.movieDetails` collection to find the answer.

NOTE: There is a dump of the video database included in the handouts for the "Creating Documents" lesson. Use that data set to answer this question.

```shell
show dbs
use video
show collections
db.movieDetails.find({"rated": {"$eq": "PG-13"}, "year": 2013, "awards":{"$exists": true}, "awards.wins": {"$eq": 0}})

# Answer
# You can find the answer using the following query: 
db.movieDetails.find({ "rated": "PG-13", "year": 2013, "awards.wins": 0})

A Decade of Decadence, Pt. 2: Legacy of Dreams
```



#### Homework 2.6

Using the video.movieDetails collection, how many movies list "Sweden" second in the the list of countries.

NOTE: There is a dump of the video database included in the handouts for the "Creating Documents" lesson. Use that data set to answer this question.

```shell
db.movieDetails.find({"countries.1":"Sweden"}).count()
# answer
6
```



#### Challenge Problem: Arrays with Nested Documents

Quiz: Quiz: Challenge Problem: Arrays with Nested Documents

*This problem is provided as a supplementary learning opportunity. It is more challenging that the ordinary homework. It is ungraded. We do not ask you submit an answer.*

Suppose our movie details documents are structured so that rather than contain an awards field that looks like this:

```
"awards" : {
    "wins" : 56,
    "nominations" : 86,
    "text" : "Won 2 Oscars. Another 56 wins and 86 nominations."
}
```

they are structured with an awards field as follows:

```
"awards" : {
    "oscars" : [
        {"award": "bestAnimatedFeature", "result": "won"},
        {"award": "bestMusic", "result": "won"},
        {"award": "bestPicture", "result": "nominated"},
        {"award": "bestSoundEditing", "result": "nominated"},
        {"award": "bestScreenplay", "result": "nominated"}
    ],
    "wins" : 56,
    "nominations" : 86,
    "text" : "Won 2 Oscars. Another 56 wins and 86 nominations."
}
```

What query would we use in the Mongo shell to return all movies in the video.movieDetails collection that either won or were nominated for a best picture Oscar? You may assume that an award will appear in the `oscars` array only if the movie won or was nominated. You will probably want to create a little sample data for yourself in order to work this problem.

HINT: For this question we are looking for the simplest query that will work. This problem has a very straightforward solution, but you will need to extrapolate a little from some of the information presented in the "Reading Documents" lesson.

Answer

```
db.movieDetails.find({"awards.oscars.award": "bestPicture"})
```



#### Challenge Problem: Updating Based on Multiple Criteria

This problem is provided as a supplementary learning opportunity. It is more challenging that the ordinary homework. It is ungraded. We do not ask you submit an answer.

Write an update command that will remove the "tomato.consensus" field for all documents matching the following criteria:

The number of imdb votes is less than 10,000
The year for the movie is between 2010 and 2013 inclusive
The tomato.consensus field is null
How many documents required an update to eliminate a "tomato.consensus" field?

NOTE: There is a dump of the video database included in the handouts for the "Creating Documents" lesson. Use that data set to answer this question.

Answer

You can arrive at the answer here in a couple of different ways, either of which provide some good learning opportunities. The key is realizing that you need to report on the number of documents that actually required an update to remove the `tomato.consensus` field. You can do this either by ensuring that you filter for only those documents that do not contain a `tomato.consensus` field or by recognizing that only 13 documents were actually modified by your update.

Using the first approach, you can issue the following command.

```
db.movieDetails.updateMany({ year: {$gte: 2010, $lte: 2013},
                             "imdb.votes": {$lt: 10000},
                             $and: [{"tomato.consensus": {$exists: true} },
                                    {"tomato.consensus": null} ] },
                           { $unset: { "tomato.consensus": "" } });
```

In response, you will receive the following:

```
{ "acknowledged" : true, "matchedCount" : 13, "modifiedCount" : 13 }
```

Using the second approach, you can issue a simpler command, but one that is not precise about what needs to be updated.

```
db.movieDetails.updateMany({ year: {$gte: 2010, $lte: 2013},
                             "imdb.votes": {$lt: 10000},
                             "tomato.consensus": null },
                           { $unset: { "tomato.consensus": "" } });
```

In response, you will receive the following:

```
{ "acknowledged" : true, "matchedCount" : 204, "modifiedCount" : 13 }
```

Note that while the query portion of the update matches 204 documents, only 13 documents actually required an update.



### Week 3: Schema Design

#### Introduction to Week 3

模式设计，MongoDB 有动态模式，每个文档都可以有自己的架构。这使得决定什么架构变得有点困难。在大多数应用程序中，每个文档都具有相同的模式，但他们仍然有选择。你可以将数据嵌入文档中，也可以将其放入文档中进入自己的收藏，而这些决定将对绩效产生影响，易于编程。因此，通过一些列用例，我们将带给你通过 MongoDB 架构设计。



#### MongoDB Schema Design

在 Mongo中证明它更重要，以有利于应用程序的方式保存数据、使用数据。

因此，你需要考虑应用程序数据模式。

考虑一起使用哪些数据，哪些数据块主要用于只读，哪些数据是一直写的。

然后我们将在其中组织我们的数据，MongoDB特别适合应用数据访问模式。

这与关系型数据库有点不同，相反，你试图保持数据的方式与应用程序无关。

首先，MongoDB 支持丰富的文档类型。

在 MongoDB你可以存储项目数组或值，某个键可以是整个其他文档。这将允许我们预先加入用于快速访问的数据。

你需要考虑想要与其他数据一起使用的数据，如果有可能，你可能想要嵌入它直接在文档中。

在 MongoDB 没有外键约束，但它会变得不像那么重要，因为嵌入而思考，嵌入将使重要性降低一点，我们将考虑原子操作，我们不支持交易。所有我们将考虑如何组织我们的数据。如果需要原子操作，请支持原子操作。

没有声明的架构在 MongoDB 中，但是你的申请将有可能有一个架构，在一个集合中大多数文档具有类似的架构，有可能文档之间有些细微的差别。

Quiz

What's the single most important factor in designing your application schema within MongoDB?（在MongoDB中设计应用程序模式最重要的因素是什么?）

Choose the best answer:

Matching the data access patterns of your application.（匹配应用程序的数据访问模式。与这些数据访问模式一致，因为这是你将获得最佳表现的方式，这就是我们从中获得最大便利的方式在你的应用程序中编程）



#### Relational Normalization

MongoDB中的一个想法就是使用你的想法，数据库到你尝试编写的应用程序以及你想要解决的问题。所以我们不会担心避免这种趋势对某些特定的访问模式。

我不想重新设计系统，每当我们改变某事时都会完成。

MongoDB 非常灵活，因为我们可以添加文档的键和属性，无需更改所有现有文件。

最后一件事是释放数据库这些修改异常，虽然你可能认为嵌入数据，它会导致这种情况，但事实并非如此。主要是，我们将避免在文档中嵌入数据。在 MongoDB 中会议某种方式导致创造了这些异常。所有我们要小心，不要吸收在很多地方。

偶尔，出于性能原因，我们会做出决定复制文档中的一些数据，但它不会是默认值。默认是我们将避免它，所以我们没有数据可以存在的这类异常改变不一致。

在某些应用程序中，你可能希望允许它没关系，或者你可能希望和在应用程序中更新它们。很多时候我们会避免它



#### Mongo Design for Blog

Quiz

Which data access pattern is not well supported by the blog schema?

Choose the best answer:

Providing a table of contents by tag（要做到这一点，你可能不得不适用聚合框架做某种类型的分组）



#### Alternative Schema for Blog

注意嵌套文档只是在16MB 字节以内

![mongoDB](/img/mongoDB/Java_Developers/schema_for_blog.png)



#### Living Without Constraints

What does Living Without Constraints refer to?
Keeping your data consistent even though MongoDB lacks foreign key constraints



#### Living Without Transacitons

MongoDB缺乏事务支持。从关系型数据库来看，很多人都知道。

事务提供原子性、一致性和隔离性、耐用性（ACID）

MongoDB存在原子操作

Which of the following operations operate atomically within a single document? Check all that apply.

Check all that apply:

- Update
- findAndModify
- $addToSet (within an update)

- $push within an update



#### One to One Relations

What's a good reason you might want to keep two documents that are related to each other one-to-one in separate collections? Check all that apply.

Check all that apply:

To reduce the working set size of your application.
Because the combined size of the documents would be larger than 16MB

分开以减少工作集的大小，因为你只是想访问部分的数据，不想带来这两个部分数据到内存

或者你可以担心合并后的文档大于16MB 字节

一对一的情况下，建议一个集合 collection



#### One to Many Relations

一对多的情况下，建议多个集合 collection

When is it recommended to represent a one to many relationship in multiple collections?
Choose the best answer:

Whenever the many is large



#### Many to Many Relations

Books: Authors（一本书可能有多个作者，每个作者都可以不止有一本书）

Students: Teachers（一个学生可能有多个老师，每个老师都可以由多个学生）



#### Multikeys

引用和嵌入的原因之一在 MongoDB 中工作得很好就是存在一个功能称为多重索引（Multikey Indexes）。

```
设计方案：
Students
{"_id": 0, "name": "Andreo", "teachers": [0,1,5,10]}

Teachers
{"_id": 0, "name": "Tony"}

思考两个问题：
1.我怎样才能找到所有老师，特定学生是什么？
2.我怎样才能知道所有学生，谁有一位特定的老师？
db.students.find()

# 给 students collection 加入老师字段的索引
db.students.ensureIndex({"teachers":1})

# 找到所有的学生，它们中有 id为0的老师和 id 为1的老师
db.students.find({"teachers": {"$all":[0,1]}})

# 在查询的末尾添加 explain,它会告诉我们它在查询时所作的事情(是否用在索引等)
db.students.find({"teachers":{"$all":[0,1]}}).explain()
```



#### Benefits of Embedding

嵌入来自两个不同集合的数据的主要好处，为了表现，将它们进行组合，提高阅读表现。并在同一个数据库中。



#### Trees

representing trees

设计方案中一个经典问题是如何表示数据库中的树

```
Products
{"category":7, "product_name": "xxxx"}

Category
{"_id": 7, "category_name": "doors", "children":[3,4,5,6]}
```

Given the following typical document for a e-commerce category hierarchy collection called *categories*

```json
{
  _id: 34,
  name: "Snorkeling",
  parent_id: 12,
  ancestors: [12, 35, 90]
}
```

Which query will find all descendants of the snorkeling category?

db.categories.find( { ancestors: 34 } )



####  When to denormalize

正常和非规范化

将数据标准化的原因之一是，在关系型数据库中个，要避免重复带来的修改异常数据。



#### What is an ODM?

ODM 带给你的是它位于应用程序与代码驱动程序之间，它有助于保护你免受驱动程序更改，API 发生变化。提供一个和好的 Java API 编写你的应用程序



#### Field mappings, indexes and constraints in Morphia

使用 Morphia 映射你的实体

```java
// @Entity 默认情况下，使用类名，可通过 value 指定 collection 名称
// noClassnameStored 默认值为 false，设置为 true 时关闭将类名称作为一个字段
@Entity(value = "users", noClassnameStored = true)
@Indexes({
    // 定义一个名为popular的索引，且是复合索引，在用户名上创建一个升序索引、追随者的降序索引，所以它会更受欢迎的用户按字母顺序排列用户
    @Index(value = "username, -followers", name = "popular"),
    // TTL索引，只能针对日期类型，意味着在这个数字之后，一个运行的线程将检查何时查看最后更新，最后一次活跃，如果超过这个秒数，MongoDB会从集合中删除这条记录，这是一种自动删除过期记录的好方法，无需管理它，使用它可能不是最好的主意，在用户集合中，因为你可能希望将它们标记为非活动状态，例如，禁用他的账户，而不是删除他的账户。如果你存储日志或任何类型的历史记录，一段时候后你就放弃了关心，你可以将它设置为三年后过期，它们就会神奇地从集合中自动消失
    @Index(value = "lastActive", name = "idle", expireAfterSeconds = 1000000000)
})
public class GithubUser {
    // @Id 定义 _id 字段，默认情况下 MongoDB和 Java 驱动程序使用对象 ID
    @Id
    public String userName;
    // 没有注释，默认情况下保留该字段 full_name
    public String fullName;
    // @Property，将重命名 document 属性
    @Property("since")
    public Date memberSince;
    public Date lastActive;
    // @Reference(lazy = true) 懒加载引用对象数组
    @Reference(lazy = true)
    public List<Repositiory> repositories = new ArrayList<>();
    public int followers = 0;
    public int following = 0;
}

@Entity("orgs")
public classs Organization {
    @Id
    public String name;
    // @Indexed 在属性字段上定义索引，当没有传递名字时自动使用属性名 
    // unique  唯一索引
    // dropDups 删除重复项索引
	// background 后台运行
    // sparse 是创建时的设置，字段不再每个文档上，你可以将其定义为稀疏，并且它具有一定的帮助功能，并在多少秒后过期，默认值为-1，这意味着不创建 TTL，这是可以直接索引属性的另一种方式
    @Indexed(value = IndexDirection.ASC, name = "", unique = false, dropDups = false, expireAfterSeconds = -1, background = false, sparse = false)
    public Date created;
    // @Version 版本的注释，必须指定在 long类型
    @Version("v")
    public long version;
}

@Entity("repos")
public class Repository {
    @Id
    public String name;
    @Reference // 非懒惰对象的引用，每次自动获取
    public Organization organization;
    @Reference
    public GithubUser owner;
    public Settings settings = new Settings();
}
```



#### CRUD Operations in Morphia

```java
public class Demos extends BaseTest {
    private SimpleDateFormat sdf = new SimpleDateFormat("MM-dd-yyyy");
    private GithubUser evanchooly;
    private Date date;
    
    public Demos() {
        date = sdf.parse("6-15-1987");
    }
    
    @Test
    public void basicUser() {
        evanchooly = new GithubUser("evanchooly");
        evanchooly.fullName = "Justin Lee";
        evanchooly.memberSince = date;
        evanchooly.following = 1000;
        ds.save(evanchooly);
    }
    
    @Test(dependsonMethods = {"basicUser"})
    public void repositories() throws ParseException {
        Organization org = new Organization("mongodb");
        ds.save(org);
        
        Repository morphia1 = new Repository(org, "morphia");
        Repository morphia2 = new Repository(evanchooly, "morphia");
        
        ds.save(morphia1);
        ds.save(morphia2);
        
        evanchooly.repositories.add(morphia1);
        evanchooly.repositories.add(morphia2);
        ds.save(evanchooly);
    }
    
    @Test(dependsOnMethods = {"repositories"}) 
    public void query() {
        Query<Repository> query = ds.createQuery(Repository.class);
        Repository repository = query.get();
        List<Repository> repositories = query.asList();
        Iterable<Repository> fetch = query.fetch();
        ((MorphiaIterator) fetch).close();
        
        Iterator<Repository> iterator = fetch.iterator();
        while(iterator.hasNext()) {
            iterator.next();
        }
        
        iterator = fetch.iterator();
        while(iterator.hasNext()){
            iterator.next();
        }
        
        query.field("owner").equal(evanchooly).get();
        GithubUser memberSince = ds.createQuery(GithubUser.class)
            .field("memberSince").equal(date).get();
        System.out.println("memberSince = " + memberSince);
        GithubUser since = ds.createQuery(GithubUser.class)
            .field("since").equal(date).get();
        System.out.prinln("since = " + since);
    }
    
    @Test(dependsOnMethods = {"repositories"})
    pubic void updates() {
        evanchooly.followers = 12;
        evanchooly.following = 678;
        ds.save(evanchooly);
    }
    
    @Test(dependsOnMethods = {"repositories"})
    public void massUpdates() {
        UpdateOperations<GithubUser> update =
            ds.createUpdateOperations(GithubUser.class)
            .inc("followers")
            .set("following", 42);
        Query<GithubUser> query = ds.createQuery(GithubUser.class)
            .field("followers").equal(0);
        ds.update(query, update);
    }
    
    @Test(dependsOnMethods = {"repositories"},
         expectedExceptions = {ConcurrentModificationException.class})
    public void versioned() {
        Organization organization = ds.createQuery(Organization.class).get();
        Organization organization2 = ds.createQuery(Organization.class).get();
        Assert.assertEquals(organization.version, 1L);
        ds.save(organization);
        
        Assert.assertEquals(organization.version, 2L);
        ds.save(organization);
        
        Assert.assertEquals(organization.version, 3L);
        ds.save(organization2);
    }
}
```



#### Homework 3.1

Download the students.json file from the Download Handout link and import it into your local Mongo instance with this command:

```shell
mongoimport --drop -d school -c students students.json
```

用代码实现删除数组中低分数的家庭作业

```java
// 我的实现，使用 $pull 操作符从现有数组中删除与指定条件匹配的值或值的所有实例
public class StudentScore {

  public static void main(String[] args) {
    MongoClient mongoClient = new MongoClient();
    MongoDatabase database = mongoClient.getDatabase("school");
    MongoCollection<Document> collection = database.getCollection("students");

    ArrayList<Document> documents = collection.find().into(new ArrayList<>());

    for (Document cur : documents) {
      List<Document> scores = (List<Document>) cur.get("scores");
      Document minHomework = null;
      for (Document score: scores) {
        if (score.getString("type").equals("homework")) {
          if (minHomework == null) {
            minHomework = score;
            continue;
          }
          if (score.getDouble("score") < minHomework.getDouble("score")) {
            collection.updateMany(new Document("_id", cur.get("_id")),new Document("$pull",new Document("scores", score)));
          }
        }
      }
      Helpers.printJson(cur);
    }
  }
}

// 官方的实现
/**
 * Drop the lowest score for students in a data set.
 */
public class RemoveLowest {
  public static void main(final String[] args) {
    MongoClient client = new MongoClient();
    MongoDatabase schoolDB = client.getDatabase("school");
    MongoCollection<Document> students = schoolDB.getCollection("students");

    // Iterate through the students and repair them.
    MongoCursor<Document> cursor = students.find().iterator();
    try {
      while (cursor.hasNext()) {
        Document student = cursor.next();
        List<Document> scores = (List<Document>) student.get("scores");
        // Now find the lowest homework score.
        Document minScoreObj = null;
        double minScore = Double.MAX_VALUE;  // Minimum score value.

        for (Document scoreDocument : scores) {
          // The array contains documents with "type" and "score".
          double score = scoreDocument.getDouble("score");
          String type = scoreDocument.getString("type");

          if (type.equals("homework") && score < minScore) {
            minScore = score;
            minScoreObj = scoreDocument; // this is the lowest score obj
          }
        }

        // Remove the lowest score.
        if (minScoreObj != null) {
          scores.remove(minScoreObj);   // remove the lowest
        }

        // replace the scores array for the student
        students.updateOne(eq("_id", student.get("_id")),
                   new Document("$set", new Document("scores", scores)));
      }
    } finally {
      cursor.close();
    }

    client.close();
  }
}
```



To verify that you have completed this task correctly, provide the identity (in the form of their _id) of the student with the highest average in the class with following query that uses the aggregation framework. The answer will appear in the _id field of the resulting document.

```shell
db.students.aggregate( [
  { '$unwind': '$scores' },
  {
    '$group':
    {
      '_id': '$_id',
      'average': { $avg: '$scores.score' }
    }
  },
  { '$sort': { 'average' : -1 } },
  { '$limit': 1 } ] )
```

Enter answer here: 13



#### Homework 3.2(MongoProc)

*Making your blog accept posts*

In this homework you will be enhancing the blog project to insert entries into the posts collection. After this, the blog will work. It will allow you to add blog posts with a title, body and tags and have it be added to the posts collection properly.

We have provided the code that creates users and allows you to login (the assignment from last week). To get started, please download blog.zip from the Download Handout link and unpack. You will be using these file for this homework and for homework 3.3.

The areas where you need to add code are marked with "XXX". You need only touch the BlogPostDAO class. There are three locations for you to add code for this problem. Scan that file for XXX to see where to work.

Here is an example of valid blog post:

```
> db.posts.find().pretty()
{
    "_id" : ObjectId("513d396da0ee6e58987bae74"),
    "title" : "Martians to use MongoDB",
    "author" : "andrew",
    "body" : "Representatives from the planet Mars announced today that the planet would adopt MongoDB as a planetary standard. Head Martian Flipblip said that MongoDB was the perfect tool to store the diversity of life that exists on Mars.",
    "permalink" : "martians_to_use_mongodb",
    "tags" : [
        "martians",
        "seti",
        "nosql",
        "worlddomination"
    ],
    "comments" : [ ],
    "date" : ISODate("2013-03-11T01:54:53.692Z")
}
```

**Note:** You must add at least one post like the one above to the posts collection before running the blog.

As a reminder, to run your blog you type:

```
mvn compile exec:java -Dexec.mainClass=course.BlogController
```

Or, use an IDE to run it.

To play with the blog you can navigate to the following URLs:

```
http://localhost:8082/
http://localhost:8082/signup
http://localhost:8082/login
http://localhost:8082/newpost
```

Ok, now it's time to validate that you got it all working using MongoProc. To do that, locate this problem in the homework browser pane in MongoProc. When you believe you have solved the problem correctly, test your solution using the "Test" button. When you see confirmation that you have completed the assignment successfully in the feedback window, you can *Turn in* your assignment.

You will see a message below about your number of submissions at the bottom of this page, but you must submit this assignment using MongoProc.

Tip: Be sure to go to settings in mongoProc, and point mongod1 to your mongod (probably localhost:27017), and web1 to your web url (probably localhost:8082)

You have used 0 of 3 submissions.

官方答案 Answer *BlogPostDAO.java:*

**BlogPostDAO.java:**



```
import static com.mongodb.client.model.Filters.eq;
import static com.mongodb.client.model.Sorts.descending;

import java.util.ArrayList;
```



```java
// Return a single post corresponding to a permalink
public Document findByPermalink(String permalink) {

    // XXX HW 3.2,  Work Here

    Document post = postsCollection.find(eq("permalink", permalink)).first();

    return post;
}
```



```java
// Return a list of posts in descending order. Limit determines
// how many posts are returned.
// Requires importing com.mongodb.client.model.Sorts.descending
public List<Document> findByDateDescending(int limit) {

    // XXX HW 3.2,  Work Here
    // Return a list of DBObjects, each one a post from the posts collection

    List<Document> posts = postsCollection.find().sort(descending("date")).limit(limit).into(new ArrayList<Document>());

    return posts;
}
```



```java
public String addPost(String title, String body, List tags, String username) {

    System.out.println("inserting blog entry " + title + " " + body);

    String permalink = title.replaceAll("\\s", "_"); // whitespace becomes _
    permalink = permalink.replaceAll("\\W", ""); // get rid of non alphanumeric
    permalink = permalink.toLowerCase();


    // XXX HW 3.2, Work Here
    // Remember that a valid post has the following keys:
    // author, body, permalink, tags, comments, date
    //
    // A few hints:
    // - Don't forget to create an empty list of comments
    // - for the value of the date key, today's datetime is fine.
    // - tags are already in list form that implements suitable interface.
    // - we created the permalink for you above.

    // Build the post object and insert it


    // Build the post object.
    Document post = new Document()
                    .append("title", title)
                    .append("author", username)
                    .append("body", body)
                    .append("permalink", permalink)
                    .append("tags", tags)
                    .append("comments", new ArrayList())
                    .append("date", new java.util.Date());

    postsCollection.insertOne(post);
    System.out.println("Inserting blog post with permalink " + permalink);

    return permalink;

}
```



#### Homework 3.3(MongoProc)

*Making your blog accept posts*

In this homework you will add code to your blog so that it accepts comments. You will be using the same code as you downloaded for HW 3.2.

Once again, the area where you need to work is marked with an XXX in the BlogPostsDAO class. There is only a single location you need to work to insert comments. You don't need to figure out how to retrieve comments for this homework because the code you did in 3.2 already pulls the entire blog post (unless you specifically projected to eliminate the comments) and we gave you the code in the template that pulls them out of the JSON document.

This assignment has fairly little code, but it's a little more subtle than the previous assignment because you are going to be manipulating an array within the Mongo document. For the sake of clarity, here is a document out of the posts collection from a working project that also has comments.

```
{
    "_id" : ObjectId("513d396da0ee6e58987bae74"),
    "author" : "andrew",
    "body" : "Representatives from the planet Mars announced today that the planet would adopt MongoDB as a planetary standard. Head Martian Flipblip said that MongoDB was the perfect tool to store the diversity of life that exists on Mars.",
    "comments" : [
        {
            "author" : "Larry Ellison",
            "body" : "While I am deeply disappointed that Mars won't be standardizing on a relational database, I understand their desire to adopt a more modern technology for the red planet.",
            "email" : "larry@oracle.com"
        },
        {
            "author" : "Salvatore Sanfilippo",
            "body" : "This make no sense to me. Redis would have worked fine."
        }
    ],
    "date" : ISODate("2013-03-11T01:54:53.692Z"),
    "permalink" : "martians_to_use_mongodb",
    "tags" : [
        "martians",
        "seti",
        "nosql",
        "worlddomination"
    ],
    "title" : "Martians to use MongoDB"
}
```



Note that you add comments in this blog from the blog post detail page, which appears at

```
http://localhost:8082/post/post_slug
```

where post_slug is the permalink. For the sake of eliminating doubt, the permalink for the example blog post above is <http://localhost:8082/post/martians_to_use_mongodb>

As a reminder, to run your blog you type:

```
mvn compile exec:java -Dexec.mainClass=course.BlogController
```



Or, use an IDE to run it.

To play with the blog you can navigate to the following URLs

```
http://localhost:8082/
http://localhost:8082/signup
http://localhost:8082/login
http://localhost:8082/newpost
```



When you believe you have solved the problem correctly, test your solution in MongoProc. When you see confirmation that your solution is correct, turn it in.

You will see a message below about the number of times you have submitted a solution through MongoProc. You should not submit until testing in MongoProc confirms that your solution is correct.



You have used 0 of 3 submissions.

Answer

**BlogPostDAO.java**

```java
// Append a comment to a blog post
public void addPostComment(final String name, final String email, final String body,
                           final String permalink) {

    // XXX HW 3.3, Work Here
    // Hints:
    // - email is optional and may come in NULL. Check for that.
    // - best solution uses an update command to the database and a suitable
    //   operator to append the comment on to any existing list of comments
    Document comment = new Document("author", name).append("body", body);
    if (email != null && !email.equals("")) {
        comment.append("email", email);
    }

    postsCollection.updateOne(eq("permalink", permalink),
                              new Document("$push", new Document("comments", comment)));
}
```



### Week4: Performance

#### Introduction to Week 4

计算机的性能由多种因素驱动，包括底层硬件的性能-CPU、硬盘和内存，一旦选择了配置，它将是你的算法决定性能。对于数据库支持的应用程序，它将成为算法用于满足你的查询。

有两种方法可以影响数据库查询的延迟和吞吐量。一种是向集合添加索引、一种是在多个服务器之间分配负载（使用分片）。

在 MongoDB 3.0中引入了可插拔存储引擎更有趣。存储引擎是控制的软件，数据如何存储在磁盘上，如何选择存储将影响机器性能。

性能通常是 DBA 数据库管理员负责，但优秀的开发人员了解性能，并在考虑性能的情况下编写应用程序可以解决性能问题。



#### Storage Engines: Introduction

Indexing: 索引

Sharding: 分片，分发数据库查询，跨多个服务器

数据库中存储引擎的含义：

存储引擎是接口和持久存储之间，我们将调用磁盘，这可能是一个固态硬盘。

MongoDB服务器与磁盘通信进行持久存储，通过存储引擎。

用 Python编写程序，将用 PyMongo驱动程序使用有线协议与数据库服务器通信，当想要创建数据，读取数据，更新数据时，并删除数据，它将与存储引擎进行通信，然后将与磁盘通信。和所有不同的结构持有保存索引的文件和涉及文件的元数据都被写入持久存储，通过这个存储引擎。现在，可能是存储引擎本身决定使用内存来优化流程。换句话说，磁盘真的很慢。

因为数据的想法是持久存储东西，会发生什么是存储引擎控制计算机上的内存，它可以决定放入内存中的内容，以及如何消耗内存以及持续存储到磁盘的内容。

因此数据库服务器本身推迟了处理，服务器上的内存以及磁盘本身到存储引擎。

现在，我们提供可插拔存储引擎架构，你可以在那里使用多个。

例如，一辆车，你希望它有不同的引擎，你可以把引擎放在车里，然后它可能会改变汽车的性能特点。它可能加速更快或者它可能获得更好或更差的汽油里程，因为你放入汽车的引擎类型。同样，取决于发动机的类型。

你加入 MongoDB，将获得不同的性能特征。

有两个主要的存储引擎，一个是 MMAP，另一个是 WiredTiger(2014年收购 WiredTiger公司)

如果你有一堆 MongoDB 服务器都在集群中运行，即存储引擎。不会影响那些不同的 MongoDB 之间的通信，存储引擎不会影响 API。例如，数据库呈现给程序员无论是什么接口，但底层都是相同的，你使用 MMAP 或 WiredTiger，性能会有一些差异

Quiz

The storage engine directly determines which of the following? Check all that apply.

- The data file format
- Format of indexes

存储引擎控制数据文件格式，如何将数据写入磁盘到持久存储。它肯定会影响存储格式索引。它不会影响整体架构在复制方面的集群或者如何从服务器传输数据，服务器提供容错，它们不受存储引擎的影响。驱动程序的写协议完全不受影响。驱动程序与服务器通信，服务器通信到存储引擎。



#### Storage Engines: MMAPv1

MMAP存储引擎v1，它是 MongoDB 的原始存储引擎，它使用 MMAP 系统调用为了实现存储管理。

```shell
# 查看分配内存或映射
man mmap
```

Quiz

Which of the following statements about the MMAPv1 storage engine are true? Check all that apply.

- MMAPv1 automatically allocates power-of-two-sized documents when new documents are inserted

- MMAPv1 is built on top of the mmap system call that maps files into memory

回答
正确答案是:

当插入新文档时，MMAPv1会自动分配两个大小的文档
这是由存储引擎处理的。
MMAPv1构建在将文件映射到内存的mmap系统调用之上
这就是为什么我们称它为MMAPv1的基本思想。
错误的是:

MMAPv1提供文档级锁定
它具有收集级锁定。
MongoDB管理每个映射文件使用的内存，决定将哪些部分交换到磁盘。
操作系统处理这个。



#### Storage Engines: WiredTiger

课堂讲稿
在这节课中，Andrew用一个“-”来引用mongod过程的论证。虽然这样做有效，但是首选语法使用了两个破折号，因此，例如，在超过3.0的版本中调用WiredTiger存储引擎，您将使用以下命令。使用MongoDB 3.2或更高版本，您不需要提供使用WiredTiger的——storageEngine选项，因为这是默认选项。

1. 不是文档级锁定，实际上是一个无锁实现，有一个乐观的并发模型存储。引擎假定两次写入都没有进行，如果是相同文件的话，然后是其中给一个写入到同一个文档，必须再试一次，而且是解开的对实际应用程序不可见，但我们确实得到了这个文档级别的并发性与 MMAP 存储中的集合级别并发引擎，这是一个巨大的胜利。
2. 这个存储引擎提供压缩，两个文件本身、数据和索引。
3. 也是一个仅附加存储引擎没有到位的更新。这意味着，如果你更新文件，标记不再使用此文档，将在某个地方分配新的空间，他们会在磁盘上写它。

```shell
# 杀死现有的 mongod进程
killall mongod

# 创建目录
mkdir WT

# 告诉 mongo 使用该目录
mongod -dbpath WT -storageEngine wiredTiger

# 如果杀掉 mongoDB 数据库后，使用原来的/data/db 数据目录，再尝试使用 wiredTiger 引擎启动会导致失败，不起作用，因为它无法读取这些文件。

# 使用 db.xxxx.stats() 查看统计数据的详情，可以查看到关于 wiredTiger 的信息，包括它的size
```

WiredTiger 存储引擎无法打开 MMAP V1创建的文件

Quiz

下面哪个是WiredTiger存储引擎的特性?

1. 文档级并发(因为它是一个无锁实现)
2. 压缩



#### Indexes

谈谈索引对数据库性能的影响，如果没有索引，你想要找到某份文件，你需要扫描集合中的每个文档，并且可能有数百万，而这个集合扫描或者表扫描。你的查询是否会表现良好，这可能是最重要的因素。比 CPU 的速度更重要，比有多少内存更重要，是否可以使用某种索引，不得不看整个集合。

那么索引如何运作？什么是指数？

索引是一组有序的东西，比如说，你有一个名字索引，你可以把它想象成一个有序的清单，Andrew 排在最左边，Zoe排在最右边，中间肯能是 Bav,Charle。并且每个索引点都有指向物理记录的指针，所以会有某种指针到光盘上的位置。

拥有有序的东西是件好事，搜索它的速度非常快。因为如果它实际上是一个线性列表，像这样，它不再典型的数据库中。但是如果它是这样的线性，那么你可以使用二进制进行搜索，它会记录两个项目的数量在此列表中。

在真实数据库和 MongoDB 中，实际的方式该索引的结构成为 BTree，会使查询更快。

但有时我们不只是想查询名字，我们还想查询名称和头发颜色。那怎么会有用呢？

假如：(a,b,c)是一个索引二进制，搜索以下时索引会起作用：

- a
- a,b
- a,b,c

搜索以下时索引不会有作用：

- c
- c,b

索引不是免费的，无论何时你在文件中改变某些东西都将影响索引，你将不得不更新该索引。你将不得不把它写在内存上，最终在磁盘上。

索引实际上会减慢你的写入速度，但是你的阅读速度会快得多。

Quiz

哪种优化通常对数据库的性能影响最大?

- 在大型集合上添加适当的索引，以便只有一小部分查询需要扫描集合。



#### Creating Indexes

```shell
# 进入 mongo shell
mongo

# 选择 school db
use school

# 查找
db.students.findOne()

# 测试运行在 MMAPv1存储引擎上有1000万条记录的数据查询的性能
db.students.find({student_id: 5})

# 添加 explain() 命令，它运行在一个集合之上，它会告诉你它会有哪些索引用来满足这个查询
db.students.explain().find({student_id: 5})
# 返回以下内容
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "school.students",
		"indexFilterSet" : false,
		"parsedQuery" : {

		},
		"winningPlan" : {
			"stage" : "COLLSCAN", // 表明它正在进行收集扫描，它正在查看所有文件，这将是非常缓慢的
			"direction" : "forward"
		},
		"rejectedPlans" : [ ]
	},
	"serverInfo" : {
		"host" : "yl.local",
		"port" : 27017,
		"version" : "4.0.3",
		"gitVersion" : "7ea530946fa7880364d88c8d8b6026bbc9ffa48c"
	},
	"ok" : 1
}

# 使用 findOne更快的原因是，一旦找到单个文档，它就可以退出查找
db.students.findOne({student_id: 5})

# 让我们添加索引 db.students.createIndex
# 为 student_id 升序上编入索引
db.students.createIndex({student_id: 1});

# 当加入索引后按照 student_id搜索会非常快
# 看看这次搜索在用什么索引
db.students.explain().find({student_id: 5})
# 返回一下内容
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "school.students",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"student_id" : {
				"$eq" : 1
			}
		},
		"winningPlan" : {
			"stage" : "FETCH",
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"student_id" : 1
				},
				"indexName" : "student_id_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"student_id" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"student_id" : [
						"[1.0, 1.0]"
					]
				}
			}
		},
		"rejectedPlans" : [ ]
	},
	"serverInfo" : {
		"host" : "yl.local",
		"port" : 27017,
		"version" : "4.0.3",
		"gitVersion" : "7ea530946fa7880364d88c8d8b6026bbc9ffa48c"
	},
	"ok" : 1
}

# 运行 explain(true) 可以告诉我们它实际上有多少文件
db.students.explain().find({"student_id": 1})
# 返回以下文件
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "school.students",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"student_id" : {
				"$eq" : 1
			}
		},
		"winningPlan" : {
			"stage" : "FETCH",
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"student_id" : 1
				},
				"indexName" : "student_id_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"student_id" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"student_id" : [
						"[1.0, 1.0]"
					]
				}
			}
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 0, 
		"executionTimeMillis" : 0,
		"totalKeysExamined" : 0,
		"totalDocsExamined" : 0,
		"executionStages" : {
			"stage" : "FETCH",
			"nReturned" : 0,
			"executionTimeMillisEstimate" : 0,
			"works" : 1,
			"advanced" : 0,
			"needTime" : 0,
			"needYield" : 0,
			"saveState" : 0,
			"restoreState" : 0,
			"isEOF" : 1,
			"invalidates" : 0,
			"docsExamined" : 0, // 检查的文件数量
			"alreadyHasObj" : 0,
			"inputStage" : {
				"stage" : "IXSCAN",
				"nReturned" : 0,
				"executionTimeMillisEstimate" : 0,
				"works" : 1,
				"advanced" : 0,
				"needTime" : 0,
				"needYield" : 0,
				"saveState" : 0,
				"restoreState" : 0,
				"isEOF" : 1,
				"invalidates" : 0,
				"keyPattern" : {
					"student_id" : 1
				},
				"indexName" : "student_id_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"student_id" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "forward",
				"indexBounds" : {
					"student_id" : [
						"[1.0, 1.0]"
					]
				},
				"keysExamined" : 0,
				"seeks" : 1,
				"dupsTested" : 0,
				"dupsDropped" : 0,
				"seenInvalidated" : 0
			}
		},
		"allPlansExecution" : [ ]
	},
	"serverInfo" : {
		"host" : "yl.local",
		"port" : 27017,
		"version" : "4.0.3",
		"gitVersion" : "7ea530946fa7880364d88c8d8b6026bbc9ffa48c"
	},
	"ok" : 1
}

# 还可以添加复合索引到集合中
# student_id 升序，class_id 降序
db.students.createIndex({student_id: 1, class_id: -1})
```

Quiz

请提供mongo shell命令，向名为students的集合添加索引，索引键为class, student_name。

两者都不会朝“-1”方向发展。

```shell
db.students.createIndex({"class": 1, "student_name": 1})
```



#### Discovering(and Deleting)Indexes

```shell
# 获得集合中有哪些索引
db.students.getIndexes();

# 删除库
use school
db.runCommand({dropDatabase: 1})

# 根据签名，删除索引
db.students.dropIndex({"student_id": 1})
```

Quiz

下列哪一种方法是在mongoDB中发现集合索引的有效方法?

db.collection.getIndexes()



#### Multikey Indexes

在数组上创建索引，称为多键索引。注意不能有两个复合索引项

```json
{"name": "Andrew", tags: ["photography", "hiking", "golf"], "color": "red", "location": ["NY", "CA"]}
```

(tags) 

(tags, color)

(tags, location)

```shell
db.foo.insert({a:1, b:2})

# 创建索引，字段 a 升序，字段 b 升序
db.foo.createIndex({a:1, b:1})

# 查看索引信息，winningPlan中使用索引扫描，但是 isMultikey: false 不是一个多键索引
db.foo.explain().find({a:1, b:1})

db.foo.insert({a:3, b:[3,5,7]})

# 这时查看索引信息就显示 多键索引 isMultikey: true 已升级为多键
db.foo.explain().find({a:1, b:1})

# 这时查看索引信息也显示 多键索引 isMultikey: ture 已升级为多键
db.foo.explain().find({a:3, b:5})

# 查看集合的索引调用，就可看到两个索引
db.foo.getIndexes()

# 会提示错误，不能插入
db.foo.insert({a:[3,4,6], b:[7,8,9]})

# 查看记录发现，b是列表或数组，而 a 则不是
db.foo.find()

# 这样插入是可以的，总是一个多键索引的方式，b 必须是一个数组，它是一个多键索引，允许任何组合，例如：a 可以是数组，b 可以使标量；或者 b 可以是数组，a 可以是标量。这都是合法的，只是不能 a 和 b 都是数组
db.foo.insert({a:[3,4,6], b:8})
```

Quiz

Suppose we have a collection *foo* that has an index created as follows:

```
db.foo.createIndex( { a:1, b:1 } )
```

Which of the following inserts are valid to this collection?

- db.foo.insert( { a : [ "apples", "oranges" ], b : "grapes" } )
- db.foo.insert( { a : "grapes", b : "oranges" } )
- db.foo.insert( { a : "grapes", b : [ 8, 9, 10 ] } )

【注意：在多键索引中不合法】`db.foo.insert({a:[1,2,3], b:[5,6,7]})`



#### Dot Notation and Multikey

使用点表示法深入到文档中，并为某些内容添加索引在主文档的子文档中，而且用数组的东西来做这件事，所以结合起来带点符号的多键

```shell
# 为 scores 集合中的 score 字段加上索引
# 因为有1000万份文件，创建索引大约需要15或20分钟来创建这个索引
db.students.createIndex({"scores.score": 1})

# 可以看到集合上现在有两个索引，一个是 _id, 另一个是 scores.score
db.students.getIndexes()

# 使用查询优化器，查找scores数组中score字段大于99的记录
# 看到它会进行索引扫描
db.students.explain().find({"scores.score": {"$gt": 99}})

# 使用 elemMatch 匹配运算符（包含至少一个元素的数组字段的文档），检查分数数组匹配所有制定的查询条件
db.students.explain().find({"scores": {"$elemMatch": {"type": "exam", "score": {"$gt": 99.8}}}})

db.students.find({"scores": {"$elemMatch": {"type": "exam", "score": {"$gt": 99.8}}}}).pretty()

# 统计共有多少条记录 20278条
db.students.find({"scores": {"$elemMatch": {"type": "exam", "score": {"$gt": 99.8}}}}).count()

# 通过 explain(true) 得知检查了多少文件
db.students.explain(true).find({"scores": {"$elemMatch": {"type": "exam", "score": {"$gt": 99.8}}}})

# 使用 and 运算符不能保证什么时候在没有 elemMatch 的情况下找到满足此条件的文档
db.students.find({"$and":[{"scores.type": "exam"},{"scores.score": {"$gt": 99.8}}]}).pretty()
```

Quiz

假设您在数据库earth中有一个名为people的集合，其中包含以下形式的文档:

```json
{
    "_id" : ObjectId("551458821b87e1799edbebc4"),
    "name" : "Eliot Horowitz",
    "work_history" : [
        {
            "company" : "DoubleClick",
            "position" : "Software Engineer"
        },
        {
            "company" : "ShopWiki",
            "position" : "Founder & CTO"
        },
        {
            "company" : "MongoDB",
            "position" : "Founder & CTO"
        }
    ]
}
```

键入您将在Mongo shell中发出的命令，在company上按降序创建索引。

```shell
db.people.createIndex({"work_history.company": -1})
```



#### Index Creation Option, Unique

创建唯一约束

```shell
# 删除 stuff 集合
db.stuff.drop()

# 插入数据
db.stuff.insert({"thing": "apple"})
db.stuff.insert({"thing": "pear"})
db.stuff.insert({"thing": "apple"})

# 查看数据
db.stuff.find()

# 为 thing 字段创建升序排列索引
db.stuff.createIndex({"thing": 1})

# 删除刚刚创建的索引
db.stuff.dropIndex({"thing": 1})

# 为 thing 字段创建升序排列索引、且是唯一的索引
# 结果报错，无法创建索引，因为这个系列里面有两个相同的值
db.stuff.createIndex({"thing": 1, {"unique": true}})

# 那就先删除其中给一个 apple吧
db.stuff.remove({"thing": "apple", {"justOne": true}})

# 删除后再去添加唯一索引
db.stuff.createIndex({"thing": 1, {"unique": true}})

# 当插入一个已经存在的值时为报错，duplicate key
db.stuff.insert({"thing": "pear"})

# 查看 stuff 集合的索引，会看到关于 thing 的索引是唯一的 unique
# 但是不会显示 _id 是 unique，虽然 _id 本来就是唯一的
db.stuff.getIndexes()

# 通过插入，验证 _id 是否唯一
db.stuff.insert({"_id": "andrew"})
# 再插入时会出现错误，_id上的重复键错误，这是完全相同的错误，所以即使数据库没有告诉你_id的索引是唯一的，但它确实是独一无二的
db.stuff.insert({"_id": "andrew"})

```

Quiz

请提供mongo shell命令，在student_id上创建唯一的索引class_id，为集合中的学生进行升序。

```shell
db.students.createIndex( {"student_id" : 1, "class_id" : 1 }, { "unique" : true } );
```



#### Index Creation, Sparse

稀疏索引是索引键丢失时可以使用的索引

```json
{a:1, b:2, c:5}
{a:10, b:5, c:10}
{a:13, b:17}
{a:7, b:23}
```

如果在 a 上创建索引不会成为问题，在 b 上创建唯一索引不会成为问题，但是在 c 上的唯一索引确实存在问题。原因是如果在 c 上创建一个唯一索引，这些文档，两者的 c 值都为 null，因此，它会违反索引的唯一约束，解决方案是用 sparse option

sparse 告诉数据库服务器是不应该的，在索引中包含缺少密钥的文档

```shell
# 创建一个“员工”小集合
db.employees.find().pretty()

# 查看集合中的索引
db.employees.getIndexes()

# 给手机 cell 上添加唯一索引，会出现错误，有多个员工没有手机号码
# errmsg: exception: E11000 duplicate key error index
db.employees.createIndex({cell:1}, {unique:true})
# 如果指定一个额外的选项就可以愉快地创建索引
db.employees.createIndex({cell:1}, {unique:true, sparse:true}})

# 再次查看集合中的索引，可以看到 cell 的索引属性 unique:true, sparse:true
db.employees.getIndexes()

# 按照员工 id 排升序
db.employess.find().sort({"employee_id": 1})

# 按照员工 id 排降序
db.employess.find().sort({"employee_id": -1})

# 使用解释器查看员工id查询
# 可以看到 winningPlan 中 IXSCAN 对文档进行排序，这将是一个非常快速的排序
db.employees.explain().find().sort({"employee_id": -1})

# 使用解释器查看员工手机查询
# 可以看到 winningPlan 中 SORT 对文档进行排序，看到它进行了完整的收集扫描，它无法在手机上使用该索引，因为这是一个稀缺索引，数据库知道有缺少的条目，即有些文件没有编入索引，如果它使用该索引进行排序，它知道它会省略文件，而且它不想省略文件，因此它会恢复到集合扫描。
# 因此，请记住，稀疏索引不能用于如果有任何文件丢失的排序情况
db.employees.explain().find().sort({"cell": -1})
```

关于使用稀疏索引，它可以使用更少的空间。这就是你为什么使用稀疏索引的原因之一。

Quiz

What are the advantages of a sparse index? Check all that apply.(稀疏指数的优点是什么?检查所有适用的。)

- The index will be smaller than it would if it were not sparse.(该指数将会比如果它不是稀疏的会更小。)
- You can gain greater flexibility with creating Unique indexes.(通过创建惟一索引，您可以获得更大的灵活性。)



#### Index Creation, Background

forground 前台索引创建时 MongoDB 中的默认设置，相对有以下好处：

- 相对较快
- 阻止用户操作数据库（正式环境下，要小心）

background 后台索引，有以下好处：

- 相对于前台索引比较慢
- 不会阻止用户操作数据库
- 一次只能有一个索引在建，下一个将排队并等待

```shell
# 以一个1000万条的学生库为例
# 当这样操作时，将会消耗大约20分钟左右的时间创建索引
db.studens.createIndex({"scores.score": 1})

# 当我继续打开一个标签页执行 Mongo shell 操作时
mongo
use school
db.students.findOne() # 会完全被阻止，卡在那里
# 没办法只能杀掉它 command + c
do you want to kill the current op(s) on the server?(y/n): y
# 再进入
mongo
# 查询位于别的库的集合就不会影响
db.employees.find()
```