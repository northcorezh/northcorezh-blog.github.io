




## docker 三大基本概念
- 镜像: image
- 容器: container
- 仓库: repository


### docker 镜像增删改查
```
docker search 镜像名：去docker hub 搜索有关镜像文件
docker pull 镜像名：下载docker镜像
docker images：查看本地有哪些docker镜像，同docker image ls
docker rmi 镜像id或者镜像名：删除本地docker镜像
docker rmi -f 镜像id：强制删除镜像文件
```


### docker 容器
- docker容器中必须有进程在后台运行,否则容器挂掉
- docker镜像每次运行 都会生成新的容器id记录

### docker 容器的增删改查

```shell
docker run 镜像名/镜像id：运行处容器进程实例

docker ps：查看正在运行的容器进程

docker ps -a：显示所有运行过的容器进程(正在运行的,以及挂掉的容器进程)，同docker container ls -a，旧的命令

docker run -it 镜像id/镜像名 /bin/bash：运行一个交互的容器

参数：-i：interactive，交互式的命令操作；-t：terminate，开启一个终端界面 ；/bin/bash：指定linux的解释器

docker rm 容器id：删除容器id记录,只能删除挂掉的容器

docker rm `docker ps -aq` ：批量删除停止运行的容器记录

docker exec -it 运行着的容器id /bin/bash：进入一个正在运行着的容器

docker logs 容器id：查看容器的日志

docker logs -f 容器id：检查容器内的日志

# 启动容器 (后台模式)
docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"

# -P参数：将容器内部使用的网络端口随机映射到主机上
docker run -P mongo
docker ps    # 查看随机映射的端口

# -p参数：设置固定映射端口
docker -run -d -p 27017:27017 xxx_id /bin/bash

# --net host参数： 可以使用通过web界面访问到容器了
　docker run --net host xxx_id /bin/bash

```



### docker 其他命令使用

```shell
# docker 查看配置信息
docker info

# 返回docker对象信息,底层信息, 返回一个 JSON 文件记录着 Docker 容器的配置和状态信息
docker inspect xxx_id

```



### docker 导入/导出

```shell
# 导出某个镜像
docker export 1e560fca3906 > center.tar

# 导入镜像
docker import cneter.tar

# 导出本地镜像
docker save -o docker-mongo.tar 213c2b6cee9f

# 导入本地镜像
docker load -i docker-mongo.tar
```





### docker 镜像市场

```
# DaoCloud 厂商
https://hub.daocloud.io/
```



### docker 阿里云镜像加速

```shell
# 阿里云镜像加速地址
https://dev.aliyun.com/
```



## 安装 docker

```SHELL

wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum install -y docker

# 配置镜像下载加速器
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io

cat /etc/docker/daemon.json
{"registry-mirrors": ["http://f1361db2.m.daocloud.io"]}

```





## docker 实验    

### 测试

```shell

docker run -d centos /bin/sh -c "while true;do echo  '熊大真棒'; sleep 1;done"
    -d：daemonize，后台运行
    centos：指定哪个镜像
    /bin/sh：指定linux的解释器
    -c：指定一段shell代码
    "while true;do echo '熊大真棒'; sleep 1;done"，每秒打印一个

```

### 自定义docker镜像

#### 案例一
```shell
# 获取一个centos镜像，运行出容器实例
docker run -it  centos /bin/bash

# 进入容器空间, 安装vim
yum install vim -y

# 退出容器，提交这个容器为一个新的镜像
exit
docker ps -a  # 查看刚才安装了vim的容器id，这里
4595c3de8418 9f3 # 这里我的查到时459

# 提交镜像语法：docker commit 容器id 新的镜像名字
docker commit 459 centos-vim

# 查看提交的镜像文件
docker images

# 导出镜像文件到指定的文件，注意为压缩文件
docker save centos-vim > /opt/centos-vim.tar.gz

# 在本地测试导入这个镜像，可以先删除我们生成的镜像
docker rmi centos-vim

# 导入镜像
docker load < /opt/centos-vim.tar.gz

# 导入的镜像，修改名字
# docker tag 旧的镜像名 以docker仓库id开头/新的镜像名
docker tag 84d  mydocker/centos-vim

# 执行docker镜像，运行出容器，查看是否携带了vim
docker run -it mydocker/centos-vim
vim  # 成功的话，vim可以执行

```


#### 案例二 docker运行一个flask框架的脚本
```shell
# 下载training/webapp的镜像，用来运行python的flask项目。
# docker run的特点，如果镜像不存在，则自动去docker pull下载

# 通过training/webapp实例一个容器对象
docker run -d -P training/webapp python app.py

# 查看运行容器的端口
docker port 容器id

# 容器的启停命令
docker start 容器id
docker stop 容器id

# 根据端口映射，查看flask框架
访问：192.168.16.122:32768

```

## Dockerfile 制作

### Dockerfile 基本结构
- 基础镜像信息
- 维护者信息
- 镜像操作指令
- 容器启动时执行指令



### Dockerfile 文件说明

**FROM: 指定基础镜像, 必须为第一个命令**

```SHELL
格式：
　　FROM <image>
　　FROM <image>:<tag>
　　FROM <image>@<digest>
　　
示例：　　FROM mysql:5.6
注：　　  tag或digest是可选的，如果不使用这两个值时，会使用latest版本的基础镜像

```



**MAINTAINER:  维护者信息**

```SHELL
格式：
    MAINTAINER <name>
示例：
    MAINTAINER Jasper Xu
    MAINTAINER sorex@163.com
    MAINTAINER Jasper Xu <sorex@163.com>
```



**RUN:  构建镜像时执行的命令**

```SHELL
RUN用于在镜像容器中执行命令，其有以下两种命令执行方式：
shell执行
格式：
    RUN <command>
    
exec执行
格式：
    RUN ["executable", "param1", "param2"]
示例：
    RUN ["executable", "param1", "param2"]
    RUN apk update
    RUN ["/etc/execfile", "arg1", "arg1"]
    
注：
　　RUN指令创建的中间镜像会被缓存，并会在下次构建中使用。如果不想使用这些缓存镜像，可以在构建时指定--no-cache参数，如：docker build --no-cache
```



**ADD: 将本地文件添加到容器中, tar类型文件会自动解压, 可以访问网络资源, 类似wget**

```SHELL
格式：
    ADD <src>... <dest>
    ADD ["<src>",... "<dest>"] 用于支持包含空格的路径
    
示例：
    ADD hom* /mydir/          # 添加所有以"hom"开头的文件
    ADD hom?.txt /mydir/      # ? 替代一个单字符,例如："home.txt"
    ADD test relativeDir/     # 添加 "test" 到 `WORKDIR`/relativeDir/
    ADD test /absoluteDir/    # 添加 "test" 到 /absoluteDir/
```



**COPY: 功能类似ADD，但是不会自动压缩文件，也不能访问网络资源**



**CMD: 构建容器后调用，也就是在容器启动时才进行调用**

```SHELL
格式：
    CMD ["executable","param1","param2"] (执行可执行文件，优先)
    CMD ["param1","param2"] (设置了ENTRYPOINT，则直接调用ENTRYPOINT添加参数)
    CMD command param1 param2 (执行shell内部命令)
    
示例：
    CMD echo "This is a test." | wc -
    CMD ["/usr/bin/wc","--help"]

注：
 　　CMD不同于RUN，CMD用于指定在容器启动时所要执行的命令，而RUN用于指定镜像构建时所要执行的命令。
```



**ENTRYPOINT: 配置容器, 使其可执行化。配合CMD可省去"application", 只使用参数**

```SHELL
格式：
    ENTRYPOINT ["executable", "param1", "param2"] (可执行文件, 优先)
    ENTRYPOINT command param1 param2 (shell内部命令)
    
示例：
    FROM ubuntu
    ENTRYPOINT ["top", "-b"]
    CMD ["-c"]
    
注：
　　　ENTRYPOINT与CMD非常类似，不同的是通过docker run执行的命令不会覆盖ENTRYPOINT，而docker run命令中指定的任何参数，都会被当做参数再次传递给ENTRYPOINT。Dockerfile中只允许有一个ENTRYPOINT命令，多指定时会覆盖前面的设置，而只执行最后的ENTRYPOINT指令。
```



**LABEL: 用于为镜像添加元数据**

```SHELL
格式：
    LABEL <key>=<value> <key>=<value> <key>=<value> ...

示例：
　　LABEL version="1.0" description="这是一个Web服务器" by="IT笔录"
　　
注：
　　使用LABEL指定元数据时，一条LABEL指定可以指定一或多条元数据，指定多条元数据时不同元数据之间通过空格分隔。推荐将所有的元数据通过一条LABEL指令指定，以免生成过多的中间镜像。

```



**ENV: 设置环境变量**

```shell
格式：
    ENV <key> <value>      #<key>之后的所有内容均会被视为其<value>的组成部分，因此，一次只能设置一个变量
    ENV <key>=<value> ...  #可以设置多个变量，每个变量为一个"<key>=<value>"的键值对，如果<key>中包含空格，可以使用\来进行转义，也可以通过""来进行标示；另外，反斜线也可以用于续行
    
示例：
    ENV myName John Doe
    ENV myDog Rex The Dog
    ENV myCat=fluffy
```



**EXPOSE: 指定于外界交互的端口**

```shell
格式：
    EXPOSE <port> [<port>...]
    
示例：
    EXPOSE 80 443
    EXPOSE 8080
    EXPOSE 11211/tcp 11211/udp
    
注：
　　EXPOSE并不会让容器的端口访问到主机。要使其可访问，需要在docker run运行容器时通过-p来发布这些端口，或通过-P参数来发布EXPOSE导出的所有端口
```



**VOLUME: 用于指定持久化目录**

```shell
格式：
    VOLUME ["/path/to/dir"]
    
示例：
    VOLUME ["/data"]
    VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"
    
注：
　　一个卷可以存在于一个或多个容器的指定目录，该目录可以绕过联合文件系统，并具有以下功能
1 卷可以容器间共享和重用
2 容器并不一定要和其它容器共享卷
3 修改卷后会立即生效
4 对卷的修改不会对镜像产生影响
5 卷会一直存在，直到没有任何容器在使用它
```



**WORKDIR：工作目录，类似于cd命令**

```shell
格式：
    WORKDIR /path/to/workdir

示例：
    WORKDIR /a  (这时工作目录为/a)
    WORKDIR b  (这时工作目录为/a/b)
    WORKDIR c  (这时工作目录为/a/b/c)

注：
　　通过WORKDIR设置工作目录后，Dockerfile中其后的命令RUN、CMD、ENTRYPOINT、ADD、COPY等命令都会在该目录下执行。在使用docker run运行容器时，可以通过-w参数覆盖构建时所设置的工作目录。
```



**USER：指定运行容器时的用户名或 UID，后续的 RUN 也会使用指定用户。使用USER指定用户时，可以使用用户名、UID或GID，或是两者的组合。当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户**

```shell
格式:
　　USER user
　　USER user:group
　　USER uid
　　USER uid:gid
　　USER user:gid
　　USER uid:group

 示例：
　　USER www

 注：

　　使用USER指定用户后，Dockerfile中其后的命令RUN、CMD、ENTRYPOINT都将使用该用户。镜像构建完成后，通过docker run运行容器时，可以通过-u参数来覆盖所指定的用户。
```



**ARG: 用于指定传递给构建运行时的变量**

```shell
格式：
    ARG <name>[=<default value>]
    
示例：
    ARG site
    ARG build_user=www
```



**ONBUILD: 用于设置镜像触发器**

```shell
格式：
　　ONBUILD [INSTRUCTION]
　　
示例：
　　ONBUILD ADD . /app/src
　　ONBUILD RUN /usr/local/bin/python-build --dir /app/src
　　
注：
　　当所构建的镜像被用做其它镜像的基础镜像，该镜像中的触发器将会被钥触发
```



**一张图解释常用指令的意义**

![img](https://images2017.cnblogs.com/blog/911490/201712/911490-20171208222222062-849020400.png)



### Dockerfile 案例一

```shell
# This my first nginx Dockerfile
# Version 1.0

# Base images 基础镜像
FROM centos

#MAINTAINER 维护者信息
MAINTAINER tianfeiyu 

#ENV 设置环境变量
ENV PATH /usr/local/nginx/sbin:$PATH

#ADD  文件放在当前目录下，拷过去会自动解压
ADD nginx-1.8.0.tar.gz /usr/local/  
ADD epel-release-latest-7.noarch.rpm /usr/local/  

#RUN 执行以下命令 
RUN rpm -ivh /usr/local/epel-release-latest-7.noarch.rpm
RUN yum install -y wget lftp gcc gcc-c++ make openssl-devel pcre-devel pcre && yum clean all
RUN useradd -s /sbin/nologin -M www

#WORKDIR 相当于cd
WORKDIR /usr/local/nginx-1.8.0 

RUN ./configure --prefix=/usr/local/nginx --user=www --group=www --with-http_ssl_module --with-pcre && make && make install

RUN echo "daemon off;" >> /etc/nginx.conf

#EXPOSE 映射端口
EXPOSE 80

#CMD 运行以下命令
CMD ["nginx"]
```



### Dockerfile build使用

**准备文件，构件docker镜像**

```shell
#!/bin/bash
# 在/opt/DjangoProject下新建了一个dockerfile-test文件夹
# ls
CentOS-Base.repo  Dockerfile  epel.repo  flask-test.py

# 构件docker镜像命令，相对路径在Dockerfile同级下执行
docker build .

# 构建镜像完成后，查看镜像文件
docker images

# 将生成的iamges 命名
docker tag 6f0 flask-docker

# 运行这个镜像并访问测试
docker run  -d  -p 7777:8080  6f0  # -p 可以指定端口来映射

```









