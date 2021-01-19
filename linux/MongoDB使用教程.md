



# MongoDB 使用教程



> 作者 changhao
>
> 邮箱 <wu_chang_hao@qq.com>
>
> 版本 `1.0`



## 知识点

- Mongodb安装
- mongo命令使用
- mongo基础操作
- pymongo 模块使用



## Mongodb安装



### windows 安装方式

1). 下载安装包

MongoDB 提供了可用于 32 位和 64 位系统的预编译二进制包，你可以从MongoDB官网下载安装，MongoDB 预编译二进制包下载地址：https://www.mongodb.com/download-center#community



2). 安装

指定安装路径，我这里安装在D:\software\mongodb，添加D:\software\mongodb\bin到环境变量中。



3). 新建目录与文件夹

```
D:\software\mongodb\data\db
D:\software\mongodb\log\mongod.log
```



4). 新建配置文件D:\software\mongodb\mongod.cfg

```
systemLog:
  destination: file
  path: "D:/software/mongodb/log/mongod.log"
  logAppend: true
storage:
  journal:
    enabled: true
  dbPath: "D:/software/mongodb/data/db"
net:
  bindIp: 0.0.0.0
  port: 27017
setParameter:
  enableLocalhostAuthBypass: false
```



5). 制作系统服务

```shell
mongod --config "D:\software\mongodb\mongod.cfg" --bind_ip 0.0.0.0 --install
```

或者直接在命令行指定配置

```shell
mongod --bind_ip 0.0.0.0 --port 27017 --logpath D:\software\mongodb\log\mongod.log --logappend --dbpath    D:\software\mongodb\data\db --serviceName "mongodb" --serviceDisplayName "mongodb" --install
```



6). 启动MongoDB服务

```shell
net start MongoDB
net stop MongoDB
```



7). 登录MongoDB

```
mongo
链接：http://www.runoob.com/mongodb/mongodb-window-install.html
  当没有账号密码登录的时候，默认就是管理员登录。，因为刚刚做系统服务install的时候没有指定
  --auth(没有指定则没有权限认证这一说),(相当于mysql跳过授权表启动一样)
```



8). 创建有权限的用户

```shell
use admin
db.createUser(
    {
        user: "root",  # 这个root可以随便写
        pwd: "123",
        roles: [{role: "root", db: "admin"}]  # 权限，role是root说明是管理员，
    }
)

use test
db.createUser(
    {
        user: "test",
        pwd: "123",
        roles: [
        {role: "readWrite", db: "test"},  # 针对test库有读写权限，操作自己的库有读写权限
        {role: "read", db: "db1"}
        ]  # 针对db1库读权限,操作其他库有读权限
    }
)
```



9). 重启数据库

```shell
mongod --remove
mongod --config "D:\software\mongodb\mongod.cfg" --bind_ip 0.0.0.0 --install --auth

# 或者
mongod --bind_ip 0.0.0.0 --port 27017 --logpath D:\software\mongodb\log\mongod.log --logappend --dbpath
D:\software\mongodb\data\db --serviceName "MongoDB" --serviceDisplayName "MongoDB" --install --auth
```



10). 重新登录

```shell
# 方式一
mongo --port 27017 -u "root" -p "123" --authenticationDatabase "admin"
# 方式二：在登录之后用db.auth("账号","密码")登录
mongo
use admin
db.auth("root","123")
```



### Linux 安装方式

1). 安装依赖

```shell
yum install -y libcurl openssl
```



2). 下载MongoDB源码安装或yum管理器安装

```shell
# mongodb源码下载地址
https://www.mongodb.com/download-center#community

# 或yum 安装
yum install -y mongodb
```



3). 创建数据库目录

```shell
# 创建目录，并设置用户权限
mkdir -p /var/lib/mongo
mkdir -p /var/log/mongodb
chown `whoami` /var/lib/mongo
chown `whoami` /var/log/mongodb
```



4). 配置mongo服务参数, 允许所有ip访问

```shell
mongod --dbpath /var/lib/mongo --logpath /var/log/mongodb/mongod.log --bind_ip_all --fork
```



5). 或者修改配置文件 /etc/mongod.conf

```shell
# mongod.conf
  
# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: /var/lib/mongo
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# network interfaces
net:
  port: 27017
  bindIp: true


# how the process runs
processManagement:
  timeZoneInfo: /usr/share/zoneinfo
```



6). 后台启动

```shell
mongod -f /etc/mongod.conf &
```



7). 登录mongodb

```shell
# 登录
mongo

# 创建数据库 test
use test

# 查看当前的数据库
db
```



## mongo命令使用

```shell
# 连接到任何数据库config
mongo 127.0.0.1:27017/config

# 不连接任何数据库
mongo --nodb

# 启动之后，在需要时运行new Mongo(hostname)命令就可以连接到想要的mongod了
conn = new Mongo('127.0.0.1:27017')
db = conn.getDB('admin')

# 显示当前数据库所有的数据库
show databases   或者  show dbs

# 使用某个数据库，不存在则创建
use database

# 删除数据库
db.dropDatabase()
```



## mongo基础操作

在MongoDB中相关术语的解释和sql术语对应关系

| SQL术语     | MongoDB术语 | 解释/说明                          |
| ----------- | ----------- | ---------------------------------- |
| database    | database    | 数据库                             |
| table       | collection  | 数据库表/集合                      |
| row         | document    | 数据库记录行/文档                  |
| column      | field       | 数据字段/域                        |
| index       | index       | 索引                               |
| table joins |             | 表连接，MongoDbB不支持             |
| primary key | primary key | 主键，MongoDB自动将_id字段设置主键 |



有一些数据库名是保留的，可以直接访问这些特殊作用的数据库

- **admin:**  从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。

- **local:** 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合
- **config:** 当Mongo用于分片设置时，config数据库在内部使用，用于保存分片的相关信息。



### 集合(表)的增删改

```shell
use test
# 增加表内容
db.table1.insert({"a":1})
db.table2.insert({"b":2})

# 查看表
show collections  或者 show tables

# 删除
db.user.info.help()  # 查看帮助
db.user.info.drop()

# 帮助， 不明白就help()
db.table.help()     # 举例
```



#### 增加操作

```shell
# 插入单条数据
user0 = {
	"name": "changhao",
	"age": "23",
	"hobbies": ['music', 'read', 'dancing'],
	"addr": {
		"country": 'China',
		"city": 'BJ',
	}
}
# 方式一
db.table1.insert(user0)
# 方式二
db.table1.insertOne(user0)

# 插入多条数据
user1 = {
	"_id": 1,
	"name": "changhao1",
	"age": "23",
	"hobbies": ['music', 'read', 'dancing'],
	"addr": {
		"country": 'China',
		"city": 'BJ',
	}
}

user2 = {
	"_id": 2,
	"name": "changhao2",
	"age": "23",
	"hobbies": ['music', 'read', 'dancing'],
	"addr": {
		"country": 'China',
		"city": 'BJ',
	}
}
# 方式一
db.table1.insertMany(user1, user2)
# 方式二
db.table1.insertMany([user1, user2])
```



#### 查找操作

```shell
# 查看所有记录
db.table1.find()

# 美化输出
db.table1.find().pretty()

# 查找符合条件的所有
db.table1.find({"name": "changhao"})

# 查找符合条件的第一条
db.table1.findOne({"name": "changhao"})
```



#### 条件查找-比较运算

```shell
# id = 1
db.table1.find({"_id":1})

# id != 1
db.table1.find({"_id": {"$ne": 1}})

# id < 2
db.table1.find({"_id": {"$lt": 1}})

# id > 1
db.table1.find({"_id": {"$gt": 1}})

# id >= 1
db.table1.find({"_id": {"$gte": 1}})

# id <= 2
db.table1.find({"_id": {"$lte": 1}})
```



#### 条件查找-逻辑运算

```shell
#逻辑运算:$and,$or,$not
# id >=3 and id <= 4
db.table1.find({"_id": {"$gte":3, "$lte":4}})

# id >=3 and id <=4 and age >=40
db.table1.find({
    "_id":{"$gte":3,"$lte":4},
    "age":{"$gte":40}
})
或者
db.table1.find({"$and":[
    {"_id":{"$gte":3,"$lte":4}},  # 一个字典就是一个条件
    {"age":{"$gte":40}}
]})

# id >=0 and id <=1 or id >=4 or name = "changhao"
db.table1.find({"$or": [
{"_id": {"$lte": 1, "$gte": 0}},
{"_id": {"$gte": 4}},
{"name": "changhao"}
]})

# id % 2 = 1  奇数
db.table1.find({"_id": {"$mod": [2, 1]}})

# 偶数, 取反
db.table1.find({"_id": {"$mod": [2, 1]}})
```



#### 正则匹配

```shell
db.table1.find({
    "name":/^jin.*?(g|n)$/i
})
```



#### 排序、跳过、截取

```shell
# 排序， 根据年龄排序， 1位正序， 1位倒序
db.table1.find().sort("age": 1)
# 选取两条数据
db.table1.find().limit(2)
# 跳过查询的数据的前2条
db.table1.find().skip(2)
db.table1.sort({"age":-1}).skip(0).limit(2)
```



#### 记录修改

语法格式如下

```shell
db.collection.update(
	<query>,
	<update>,
	{
		upsert: <boolean>,
		multi: <boolean>,
		writeConcern: <document>
	}
)
```

参数说明：

- query: 相当于where条件
- update: update的对象和一些更新的操作符(如inc...等, 相当于set后面的)
- upsert: 可选, 默认为false, 代表如果不存在update的记录不更新也不插入，设置为true代表插入。
- multi: 可选，默认false，代表只更新找到的第一条记录，设置true, 代表更新找到的全部记录。
- writeConcern：可选, 抛出异常的级别。

实战语句

```shell
# 1.覆盖式
db.table1.update({'age': 20}, {"name": "changhao"})

# 2. 这一种简单的更新，完全替换匹配
var obj = db.table1.findOne({"_id": 2})
obj.username = obj.name + 'test'
obj.age = 23
db.table1.update({"_id": 1}, obj)
```



设置

```shell
# 设置 $set
通常文档只会有一部分需要更新。可以使用原子性的更新修改器，指定对文档中的某些字段进行更新。
更新修改器是种特殊的键，用来指定复杂的更新操作，比如修改、增加后者删除

# set  name="changhao" where id = 2
db.table1.update({'_id': 2}, {"$set": {"name": "changhao", "age": 23}, {"upsert":true})

# 没有匹配成功则新增一条{"upsert":true}
db.table1.update({'_id':2},{"$set":{"name":"changhao", "age":23}},{"upsert":true})

# 默认只改匹配成功的第一条,{"multi":改多条}
db.table1.update({'_id':{"$gt":4}},{"$set":{"age":28}})
db.table1.update({'_id':{"$gt":4}},{"$set":{"age":38}},{"multi":true})

# 修改内嵌文档，把名字为test的人所在的地址国家改成Japan
db.table1.update({'name':"test"},{"$set":{"addr.country":"Japan"}})

# 把名字为test的人的地2个爱好改成piao
db.table1.update({'name':"test"},{"$set":{"hobbies.1":"piao"}})

# 删除test的爱好,$unset
db.table1.update({'name':"test"},{"$unset":{"hobbies":""}})
```



#### 增加和减少

```shell
# 增加和减少 $inc
# 1.所有人年龄增加一岁
db.table1.update({},
    {
        "$inc":{"age":1}
    },
    {
        "multi":true
    }
)
# 2.所有人年龄减少5岁
db.user.update({},
    {
        "$inc":{"age":-5}
    },
    {
        "multi":true
    }
)
```



#### 删除

```shell
# 删除多个中的第一个
db.table1.deleteOne({"age":8})

# 删除国家为China的全部
db.table1.deleteMany({"addr.country": "China"})
```



## pymongo 模块使用



### 连接数据库

```python
import pymongo
client = pymongo.MongoClient("192.168.158.137", 27017)
# 显示服务器上的所有数据库
dblist = client.database_names()

print(dblist)
# 打印信息：['admin', 'config', 'local', 'test']

# 连接数据库，获取库，如果库名存在，则使用，不存在创建
db = client['test']
```



### 增加数据库内容

```python
# 增加单条
mydict = {"name": "changhao001", "age": 23}
res = db.table1.insert_one(mydict)
print(res.inserted_id)				# 返回ObjectId对象

# 增加多条
mylist = [{"name": "changhao002", "age": 23}, {"name": "changhao003", "age": 23}]
res = db.table1.insert_many(mylist)
print(res.inserted_ids)				# 返回ObjectId对象列表
```



### 查询语句

查询一条语句

```python
# 查询一条数据
res1 = db.table1.find_one({'name': 'changhao1'})
print(res1.items())             # 查看匹配的数据内容
```



查询集合中的所有数据或多条

```python
# 查询集合中的所有数据或多条
res2 = db.stu.find({})          # 查询所有记录
res3 = db.stu.find({"age": 23})
```



高级查询

```python
# 年龄大于20岁的
myquery = {"age": {"$gt": 20}}
res = db.table1.find(myquery)
```



正则表达式查询

```python
# 读取 name 字段中第一个字母为 "c" 的数据
myquery = {"name": {"$regex": "^c"}}
res = db.table1.find(myquery)
# 遍历出匹配的数据
for x in res: print(x)
```



查询结果排序，跳过， 截取条数

```python
# 查询结果的排序，跳过，和截取
sort_res = list(db.table1.find().sort("age", pymongo.DESCENDING))   # 查询所有结果，并根据年龄的降序排序
skip_res = list(db.table1.find().skip(2))                           # 查询所有结果，并过滤掉前两条
limit_res = list(db.table1.find().limit(2))                         # 查询所有结果，并截取前两条

# 分页效果
pagination_res = list(db.table1.find().sort("age", pymongo.DESCENDING).limit(2).skip(2))
```



### 修改语句

```python
# 查询到直接更新设置值
res = db.table1.update_one({'_id': 1}, {"$set": {"name": "changhao_update"}})
print(res.modified_count)

# 查询到记录, 在字典中增加值
res = db.table1.find_one({"name": "changhao_update"})
res['age'] = 20
# 将值更新到记录中
res = db.table1.update_one({"name": "changhao_update"}, {"$set": res})
print(res.modified_count)
```



### 删除语句

删除单条

```python
# 删除结果中的一条
res = db.table1.delete_one({"name": "changhao_update"})
print(res.deleted_count)
```



删除多条

```python
# 删除多条, 正则匹配的数据
query = {"name": {"$regex": "^c"}}
res = db.table1.delete_many(query)
print(res.deleted_count)

# 删除所有
del_res = db.table1.delete_many({})
```

