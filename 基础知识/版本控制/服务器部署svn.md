#  服务器部署svn

- subversion [官网网址](http://subversion.apache.org/)

# 安装svn

[安装环境选择](http://subversion.apache.org/packages.html#ubuntu)

```shell
apt-get install subversion
apt-get install libapache2-svn
```

# 配置

```shell
# 创建目录
mkdir svn
cd svn
# 创建仓库 test
svnadmin create repo1
cd repo1
cd conf
# 修改服务设置
vim svnserve.conf
```

> 注意⚠️: 
>
> 1. 创建目录是存放所有 svn 管理的版本库
> 2. `svnadmin create xxx` 命令是创建仓库的名称，其中包含配置信息

修改配置如下：

```properties
[general]
# 不允许匿名登陆
anon-access = none 
# 权限为 write/read
auth-access = write 
# 读取密码数据库文件
password-db = passwd
# 读取权限数据库文件
authz-db = authz
# 项目的名称
realm = repo1
```

其中当前目录有两个文件`authz`和`passwd` 一个存放用户权限，一个存放用户的密码

添加用户`vim passwd `:

```properties
[users]
didiaoyuan = didiaoyuan20200530
```

添加项目权限`vim authz`:

```properties
[groups]
# 组名 = 用户1.用户2
admin = didiaoyuan


# [<版本库>:/项目/目录]
# @<用户组名> = <权限>
# <用户名> = <权限>
[test:/]
@admin = rw
```

> 注意⚠️:
>
> 1. 添加组，是为了给仓库添加用户权限的同时，可以根据用户组来划分权限
> 2. 版本库可以权限可以划分到子目录

# 启动

输入`svnserve -d -r /home/user/document `命令,`-d` 表示后台运行

> 注意⚠️: 这里是使所有的仓库都有效，而不是启动单独一个库. 默认端口3690



# 开启防火墙端口



# 测试

```shell
svn checkout svn://localhost/repo1
Authentication realm: <svn://localhost:3690> repo1
Password for 'root': *************

Authentication realm: <svn://localhost:3690> repo1
Username: didiaoyuan
Password for 'didiaoyuan': ******************


-----------------------------------------------------------------------
ATTENTION!  Your password for authentication realm:

   <svn://localhost:3690> repo1

can only be stored to disk unencrypted!  You are advised to configure
your system so that Subversion can store passwords encrypted, if
possible.  See the documentation for details.

You can avoid future appearances of this warning by setting the value
of the 'store-plaintext-passwords' option to either 'yes' or 'no' in
'/root/.subversion/servers'.
-----------------------------------------------------------------------
Store password unencrypted (yes/no)? yes
```

# 参考链接

- [Linux环境(CentOS6.7 64位)下安装subversion1.9.5的方法](https://www.bnskd.com/5457.html)
- [linux 下的subversion客户端安装](