



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

   











