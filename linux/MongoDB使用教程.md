



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
show databases

# 使用某个数据库，不存在则创建
use database

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

- **admin:** 

- **local:**
- **config:**





























