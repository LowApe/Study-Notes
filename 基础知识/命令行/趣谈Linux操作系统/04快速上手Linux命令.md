# 快速上手 Linux 命令

## 用户与密码

修改密码：

```shell
passwd
Changing password for root.
Old Password:
```

这个命令放在 `/etc/passwd`文件，组的信息放在`/etc/group`



添加用户：

```shell
useradd userName
# 创建用户的使用指定组
useradd -h
```

## 游览文件

Linux 文件需要通过命令来进行查看

```shell
cd directory # 切换目录
cd . #表示切换当前目录
cd .. #表示切换到上一级目录
ls -l#列出当前目录下的文件 -l表示用列表显示
```



```shell
ls -l
total 45
drwxrwxr-x+ 103 root  admin  3502 12  3 08:05 Applications
-rw-r--r--  66 root  wheel  2244 10 10 15:07 Library
```

其中第一个字段的第一个字符是**文件类型**。

- 如果是 `-`,表示普通文件；
- 如果是 `d`,就表示目录

第一个字段剩下的 9个字符是**模式**,其实就是权限位(access permission bits).3 个一组，每组 `rwx` 表示读(read)、写(write)、执行(execute).**如果是字母，就说明有这个权限**

`-rw-r-r--`表示这是一个普通文件，对所属用户，可读可写不能执行，对于所属的组，仅仅可读；对于其他用户，也是仅仅可读。如果想改变权限，可以使用命令 `chmod711 hosts`

第二个字段是**硬链接数目**

第三个字段是**所属用户**

第四个字段是**所属组**

第五个字段是**文件的大小**

第六个字段是**文件被修改的日期**

第七个字段是**文件名**

可以通过命令 `chown`改变所属用户，`chgrp`改变所属组

> ⚠️注意：上传`scp /本地文件路径/本地文件名 服务器用户名@服务器IP:路径`
> 下载：`scp 服务器用户名@服务器IP:路径/文件名 /本地文件路径`

## 安装软件

对于 windows 系统，最方便的方式就是下载 exe。对于 Linux 来讲，也是类似的方法，你可以下载 rpm 或者 deb

因为 Linux 常用的有两大体系

- CentOS-rpm
- Ubuntu-deb

CentOS 使用 `rpm -i jdk-XXX_linux-x64_bin.rpm`

Ubuntu 下面使用 `dpkg -i jdk-XXX_linux-x64_bin.deb`

 在 Linux 下面，拼接 `rpm -qa` 和 `dpkg -l`就可以查看安装的软件列表，`-q`就是 query，a 就是 all，-l 就是 list

使用`grep`来进行筛选。如果要删除，可以用 `rpm -e` 和 `dpkg -r` 。-e 就是 earse， -r 就是remove



Linux 也有自己的软件管家，CentOS是` yum`，Ubuntu 下面是 `apt-get`

> 💡提示：可以通过 search 参数搜索软件，如果数量太多，你可以通过管道 `grep` 、`more`、`less` 来进行过滤
>
> 

如何卸载

- yum erase software
- apt-get purge software



Linux 下载的软件都会有一个统一的服务端，来保存这些软件

- CentOS 配置文件 `/etc/yum.repos.d/CentOs-Base.repo`
- Ubuntu来讲，配置文件在 `/etc/apt/sources.list`



> ​	如果不同国家，可以修改不同国家的数据源，相对速度快一些
>
> ⚠️注意：不管是先下载再安装，还是通过软件管家进行安装，都是下载一些文件，然后将这些文件放在某个路径下，然后在相应的配置文件中配置一下



Linux 默认有 tar 程序，如果是解压缩 zip 包，就需要另行安装

```shell
apt-get install zip unzip
```

在 Linux 通过 tar 解压缩之后，也需要配置环境变量，可以通过 export 命令来配置

```shell
export JAVA_HOME = /root/xxxx
export PATH = $JAVA_HOME/bin:$PATH
```

> ⚠️注意：export 命令仅在当前命令行的会话中管用，一旦退出重新登录进来，就不管用，需要编辑`/root获取/home/cliu8` 目录下的`.bashrc`

编辑文本使用 vim ，需要掌握 vim 基本的操作



## 运行程序

Linux 不是根据后缀名来执行的。它的执行条件是这样的：只要文件有 `x` 执行权限，都能到文件所在的目录下，通过 `./filename` 运行这个程序。当然，如果放在 PATH 里设置的路径下面，就不用 `./` 了，直接输入文件名就可以运行了，Linux 会帮你找

**Linux 运行程序的第二种方式，后台运行**，使用 `nohup`命令，这个命令意思是 no hang up 不挂起。表示当前交换命令行退出的时候，程序还在。



```shell
# 1 表示文件描述1表示标准输出，2表示文件描述符2标准错误输出，
# "2>&1" 表示标准输出和错误输出合并输出到 out.file 里
nohup command >out.file 2>&1 &
```

如何结束进程

```shell
# ps -ef 列出所有正在运行的程序
# grep 查找关键字
# awk 工具可以对文本进行处理,‘{print &2}’ 表示第二列的内容，运行的程序ID
# xargs 传递给 kill -9 关闭运行的程序
ps -ef | grep 关键字 | awk '{print &2}'|xargs kill -9

```



**程序运行的第三种方式，以服务的方式运行**，例如常用的数据库 MySQL，就是使用这种方式

```shell
apt-get install mysql-server #安装mysql
systemctl start mysql  #启动 mysql
systemctl enable mysql #设置开机启动
```

之所以成为服务并且能够开机启动，是因为 `/lib/systemd/system`目录下会创建一个 XXX.service 的配置文件，里面定义了如何启动、如何关闭

> ⚠️注意：CentOS 里，因为担心 mysql 授权问题，改为使用 MariaDB，通过
> 命令 `yum install mariadb-server mariadb`进行安装
> 命令`systemctl start mariadb`启动
> 命令`systemctl enable mariadb` 设置开机启动
> 在`/usr/lib/systemd/system`目录下

## 关机重启

```shell
shudown -h now # 现在就关机
reboot # 重启
```

