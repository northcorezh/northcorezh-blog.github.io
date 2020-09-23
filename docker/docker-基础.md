

https://www.cnblogs.com/ryxiong-blog/articles/11142585.html


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

** docker容器的增删改查 **

```

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

```



















