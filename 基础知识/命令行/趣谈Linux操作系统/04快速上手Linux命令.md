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



