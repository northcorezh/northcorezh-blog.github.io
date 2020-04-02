
# deb包详解及打包教程

---

Version 0.1  
create time 2020.04.01 15:38  
update time 2020.04.02 09:02  
author changhao  

---

```
外网源
vim /etc/apt/sources.list
http://archive.kylinos.cn/kylin/KYLIN-ALL/
```

## 一、deb包介绍

deb包本身有三部分组成

组成 | 详细 |
:-: | :-: |
数据包 | 包含实际安装的程序数据 |
安装信息及控制包 | 包含deb的安装说明，标识，脚本等，文件名为“control.tar.gz” |
二进制数据	| 包含文件头等信息，需要特殊软件才能查看 |

### 1.1 deb包文件结构
```
|----DEBIAN
       |-------control
       |-------postinst(postinstallation)
       |-------postrm(postremove)
       |-------preinst(preinstallation)
       |-------prerm(preremove)
       |-------copyright(版权)
       |-------changlog(修订记录)
       |-------conffiles
|----etc
|----usr
|----opt
|----tmp
|----boot
       |-----initrd-vstools.img
```

### 1.2 control文件
  这个文件主要描述软件包的名称（Package），版本（Version），Installed-Size（大小），
Maintainer（打包人和联系方式）以及描述（Description）等，是deb包必须具备的描述性文件，
以便于软件的安装管理和索引。  

字段 | 用途 | 例子/其他 |
:-: | :-: | :-: |
Package | 程序名称 | 	中间不能有空格 |
Version | 软件版本 | 	 |
Description | 程序说明 | 	 |
Section | 软件类别 | utils, net, mail, text, x11 |
Priority | 软件对于系统的重要程度 | required, standard, optional, extra等 |
Essential | 是否是系统最基本的软件包 | yes/no，若为yes,则不允许卸载（除非强制性卸载） |
Architecture | 	软件所支持的平台架构 | i386, amd64, m68k, sparc, alpha, powerpc等 |
Source | 	软件包的源代码名称 | |
Depends | 软件所依赖的其他软件包和库文件 | 若依赖多个软件包和库文件，采用逗号隔开 |
Pre-Depends | 软件安装前必须安装、配置依赖性的软件包和库文件 | 常用于必须的预运行脚本需求 |
Recommends | 	推荐安装的其他软件包和库文件 | |
Suggests | 	建议安装的其他软件包和库文件 | |

### 1.3 preinst文件
在Deb包文件解包之前（即软件安装前），将会运行该脚本。可以停止作用于待升级软件包的服务，直到软件包安装或升级完成  

### 1.4 postinst文件
负责完成安装包时的配置工作。如新安装或升级的软件重启服务。软件安装完后，执行该Shell脚本，一般用来配置软件执行环境，必须以“#!/bin/sh”为首行  
```SHELL
#!/bin/sh
echo "my deb" > /root/mydeb.log
```

```SHELL
#!/bin/sh
if [ "$1" = "configure" ]; then
/Applications/MobileLog.app/MobileLog -install
/bin/launchctl load -wF /System/Library/LaunchDaemons/com.iXtension.MobileLogDaemon.plist 
fi
```

### 1.5 prerm 文件
该脚本负责停止与软件包相关联的daemon服务。它在删除软件包关联文件之前执行。

```SHELL
#!/bin/sh
if [[ $1 == remove ]]; then
/Applications/MobileLog.app/MobileLog -uninstall
/bin/launchctl unload -wF /System/Library/LaunchDaemons/com.iXtension.MobileLogDaemon.plist 
fi
```

### 1.6 postrm 文件
负责修改软件包链接或文件关联，或删除由它创建的文件。软件卸载后，执行该Shell脚本，
一般作为清理收尾工作，必须以“#!/bin/sh”为首行  
```SHELL
#!/bin/sh
rm -rf /root/mydeb.log
```


## 二、dpkg 命令使用

### 2.1 打包 dpkg -b
第一个参数为将要打包的目录名（.表示当前目录），第二个参数为生成包的名称<.deb file name>  
```SHELL
dpkg -b . mydeb-1.deb
```

### 2.2 安装(解包并配置) dpkg -i | --install <.deb file name>
```SHELL
dpkg -i mydeb-1.deb

# 强制安装
dpkg --force-depends -i mydeb-1.deb

```

### 2.3 解包
```SHELL
dpkg --unpack mydeb-1.deb
```

### 2.4 卸载
```SHELL
# 删除包，但保留配置文件
dpkg -r my-deb

# 删除包，且删除配置文件
dpkg -P my-deb

```

### 2.5 其他参数
```SHELL

# 查看deb包是否安装/deb包的信息 dpkg -s|--status <package>
dpkg -s my-deb

# 查看deb包文件内容
dpkg -c mydeb-1.deb

# 查看当前目录某个deb包的信息
dpkg --info mydeb-1.deb

# 解压deb中所要安装的文件
dpkg -x  mydeb-1.deb mydeb-1

# 解压deb包中DEBIAN目录下的文件（至少包含control文件）
dpkg -e mydeb-1.deb mydeb-1/DEBIAN

# 列出与该包关联的文件 dpkg -L|--listfiles <package>
dpkg -L my-deb

# 配置软件包 dpkg --configure <package>
dpkg --configure my-deb

# 查看某个文件属于哪个deb包
dpkg -S filepath

```

## 三、制作deb流程
准备好可执行的二进制文件，这个二进制文件要可执行，提前要考虑兼容性，如果程序有目录要完整的一个程序目录  

**参考**
```
1) Package 包名
2) Version 版本
3) Architecture 目标机架构（i386, arm等）
4) Maintainer 维护者
5) Depends 依赖软件包
6) Description 描述
```

### 3.1 示例一
我们测试名称为JFeng-deb  
新建一个名为DEBIAN文件夹   
此文件夹内存放控制信息  


在DEBIAN里新建一个文本文档, 名为control, 编码为utf-8, 内容如下所示：
```
Package: JFeng
Version: 1.1.0
Architecture: amd64
Section: utils
Priority: optional
Maintainer: MC
Homepage: http://montecarlo.org.cn
Description: Gale debug
```

然后我们创建对应的二进制包安装完成后的路径信息放置在DEBIAN的同级目录下，
也就是把当前的目录当成根(“/”)目录,制作完成后安装时，
当前目录下除了DEBIAN目录的其他目录都会被默认安装到系统的“/”目录下。

完整实验例子目录结构
```
JFeng-deb
├── DEBIAN
│  └── control
├── opt
│  └── JFeng
│      ├── heart
│      └── heart.desktop
└── usr
    ├── bin
    │  └── heart -> /home/wxyz/桌面/JFeng-deb/opt/MyDeb/heart
    └── share
        ├── applications
        │  └── heart.desktop
        └── icons
            └── heart_98.png
```

**打包**  
```SHELL
sudo dpkg -b JFeng-deb/ JFeng-linux-amd64.deb
```


### 3.2 使用checkinstall方法创建deb包
如果你已经从它的源码运行”make install”安装了linux程序. 想完整移除它将变得真的很麻烦, 除非程序的开发者在Makefile里提供了uninstall的目标设置. 否则你必须在安装前后比较你系统里文件的完整列表，然后手工移除所有在安装过程中加入的文件. 这时候Checkinstall就可以派上使用。Checkinstall会跟踪install命令行所创建或修改的所有文件的路径(例如：”make install”、”make install_modules”等)并建立一个标准的二进制包，让你能用你发行版的标准包管理系统安装或卸载它，(例如Red Hat的yum或者Debian的apt-get命令)

```SHELL
apt-get install checkinstall
```

可以使用checkinstall --help来查看帮助信息  

checkinstall不仅可以生成deb包, 还可以生成rpm包，使用简单，但是不灵活，功能粗糙，只做介绍，不推荐使用  

但是他适合从源代码直接构建我们的deb包, 我们下载到待打包的源代码以后, 先使用make和make install编译安装, 然后运行checkinstall即可完成deb的打包  

```SHELL

git clone git@github.com:chinaran/color-compile.git   # 下载源代码
cd color-compile   
make && make install # 构建

```

```
# 制作deb
checkinstall -D --pkgname=color-compile --pkgversion=2014-12-20 --install=no  --pkgsource=../color-compile
```

### 3.3 使用dh_make方法创建deb包
deb包所需的默认信息  

```SHELL
# 生成制作
dh_make -s -e gatieme@163.com -p color-compile_2014-12-20  -f ./color-compile_2014-12-20.tar.gz
```

此时当前目录下生成了debian目录, 此时通常修改两个文件:  
- 修改debian/control文件，配置你的信息，具体字段见参考部分
- 修改debian/rules脚本，它决定编译参数(也可以不改)

dpkg是最基本的制作deb包的方法, 推荐使用  

```SHELL
dpkg-buildpackage -rfakeroot
```

此时可以看到，上层目录中已建立了deb包  


### 3.4 修改已有deb包
自己创建deb所需目录结构(控制信息和安装内容)，然后打包，一般使用这种方法来修改已有的deb包，而不是新建deb包，命令如下：  
```SHELL

dpkg -X xxx.deb test #  解包安装内容
cd test
dpkg -e ../xxx.deb #  解包控制信息

```

**修改其中内容, 重新打包**
```SHELL

cd ../
dpkg -b dirname xxx_new.deb #  重新打包

```



**参考  https://blog.csdn.net/gatieme/article/details/52829907**  
**参考  https://blog.csdn.net/baidu_38172402/article/details/81022667**  
**参考  https://blog.csdn.net/qq_16149777/article/details/86672286**  
**参考  https://zh.opensuse.org/Packaging_for_Debian_and_Ubuntu**

---

**官方**  
https://www.debian.org/doc/manuals/maint-guide/index.zh-cn.html  
