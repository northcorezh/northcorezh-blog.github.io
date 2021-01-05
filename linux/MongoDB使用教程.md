



# MongoDB 使用教程



## 安装



### windows 安装方式

1. 下载安装包

   MongoDB 提供了可用于 32 位和 64 位系统的预编译二进制包，你可以从MongoDB官网下载安装，MongoDB 预编译二进制包下载地址：https://www.mongodb.com/download-center#community



2. 安装

   指定安装路径，我这里安装在D:\software\mongodb，添加D:\software\mongodb\bin到环境变量中。



3. 新建目录与文件夹

   ```
   D:\software\mongodb\data\db
   D:\software\mongodb\log\mongod.log
   ```

   

4. 新建配置文件D:\software\mongodb\mongod.cfg

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



5. 制作系统服务

   ```shell
   mongod --config "D:\software\mongodb\mongod.cfg" --bind_ip 0.0.0.0 --install
   ```

   或者直接在命令行指定配置

   ```shell
   mongod --bind_ip 0.0.0.0 --port 27017 --logpath D:\software\mongodb\log\mongod.log --logappend --dbpath    D:\software\mongodb\data\db --serviceName "mongodb" --serviceDisplayName "mongodb" --install
   ```

   

6. 启动MongoDB服务

   ```shell
   net start MongoDB
   net stop MongoDB
   ```

   

7. 登录MongoDB

   ```
   mongo
   链接：http://www.runoob.com/mongodb/mongodb-window-install.html
     当没有账号密码登录的时候，默认就是管理员登录。，因为刚刚做系统服务install的时候没有指定
     --auth(没有指定则没有权限认证这一说),(相当于mysql跳过授权表启动一样)
   ```




 8. 创建有权限的用户

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



9. 重启数据库

   ```shell
   mongod --remove
   mongod --config "D:\software\mongodb\mongod.cfg" --bind_ip 0.0.0.0 --install --auth
   
   # 或者
   mongod --bind_ip 0.0.0.0 --port 27017 --logpath D:\software\mongodb\log\mongod.log --logappend --dbpath
   D:\software\mongodb\data\db --serviceName "MongoDB" --serviceDisplayName "MongoDB" --install --auth
   ```



10. 重新登录

    ```shell
    # 方式一
    mongo --port 27017 -u "root" -p "123" --authenticationDatabase "admin"
    # 方式二：在登录之后用db.auth("账号","密码")登录
    mongo
    use admin
    db.auth("root","123")
    ```

    

### Linux 安装方式

1. 安装依赖

   ```shell
   yum install -y libcurl openssl
   ```



2. 下载MongoDB源码安装或yum管理器安装

   ```shell
   # mongodb源码下载地址
   https://www.mongodb.com/download-center#community
   
   # 或yum 安装
   yum install -y mongodb
   ```



 3. 创建数据库目录

    ```shell
    # 创建目录，并设置用户权限
    mkdir -p /var/lib/mongo
    mkdir -p /var/log/mongodb
    chown `whoami` /var/lib/mongo
    chown `whoami` /var/log/mongodb
    ```



 4.  配置mongo服务参数

    ```shell
    mongod --dbpath /var/lib/mongo --logpath /var/log/mongodb/mongod.log --fork
    ```



 5. 后台启动

    ```shell
    
    ```

    

 6. 登录mongodb

    ```
    mongo
    ```

    

	7. 创建数据库

    ```shell
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



## mongo基础

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
db.user.find({
    "_id":{"$gte":3,"$lte":4},
    "age":{"$gte":40}
})
或者
db.user.find({"$and":[
    {"_id":{"$gte":3,"$lte":4}},  #一个字典就是一个条件
    {"age":{"$gte":40}}
]})

# id >=0 and id <=1 or id >=4 or name = "changhao"
db.table1.find({"$or":[
{"_id": {"$let": 1, 
]
```



























