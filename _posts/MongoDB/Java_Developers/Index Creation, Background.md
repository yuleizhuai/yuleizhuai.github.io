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



