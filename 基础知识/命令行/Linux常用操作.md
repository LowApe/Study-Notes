# Linux 常用操作





# Linux 常用命令



## Linux 用户管理

### 1 useradd 添加用户

```shell
# 创建用户 默认uid 和 用户组
useradd didiaoyuan
# 在 /etc/passwd 末尾添加一条记录
didiaoyuan:x:1001:1001::/home/didiaoyuan:/bin/bash
```

**创建用户操作**

- 在 `/etc/passwd `文件末尾插入用户信息
- 为该用户创建家目录(`/home/userName`)
- 复制 `/etc/skel`文件(隐藏文件到家目录)
- 新建一个与用户名相同的用户组

```shell
# 创建一个指定 uid 10005 指定用户组 root 的用户 test
useradd -u 10005 -g root test

# 在 /etc/passwd 末尾添加一条记录
test:x:10005:0::/home/test:/bin/bash
```

- `-u`指定 uid 
- `-g`指定用户组 

```shell
# 指定用户的家目录
useradd -d xxxx/path userName
```

### 2 passwd 修改密码

创建用户后没用登陆，设置密码，所以 `/etc/shadow`显示两个 `!!`

```properties
didiaoyuan:!!:18542:0:99999:7:::
test:!!:18542:0:99999:7:::
```

```shell
[root@centos-7 didiaoyuan]# passwd didiaoyuan
Changing password for user didiaoyuan.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
```

> 上面 # 是因为 root 用户，使用命令可以跟用户名，普通用户 passwd 后面不能跟用户名

### 3 usermod 修改用户



```shell
# 修改用户的家目录
[root@centos-7 home]# cat /etc/passwd | grep didiaoyuan
didiaoyuan:x:1002:1002::/home/didiaoyuan:/bin/bash

[root@centos-7 home]# usermod -d /home/modHomePath  didiaoyuan
# 查看 /etc/passwd 修改家路径
[root@centos-7 home]# cat /etc/passwd | grep didiaoyuan
didiaoyuan:x:1002:1002::/home/modHomePath:/bin/bash

```

- `-d`指定家目录路径

```shell
# 查看密码信息
[root@centos-7 home]# cat /etc/shadow | grep didiaoyuan
didiaoyuan:$6$le4/DwzY$vUNqGXtv.5DKtm8lE6C7SBAMw9eHMhWtI794lMWSFfxYMuzAZF9KuIxdlK7YrwdJgEDJmYfcBihwd11MvJMfq/:18542:0:99999:7:::
# 冻结用户
[root@centos-7 home]# usermod -L didiaoyuan
# ！ 表示冻结
[root@centos-7 home]# cat /etc/shadow | grep didiaoyuan
didiaoyuan:!$6$le4/DwzY$vUNqGXtv.5DKtm8lE6C7SBAMw9eHMhWtI794lMWSFfxYMuzAZF9KuIxdlK7YrwdJgEDJmYfcBihwd11MvJMfq/:18542:0:99999:7:::
# 解冻用户
[root@centos-7 home]# usermod -U didiaoyuan

[root@centos-7 home]# cat /etc/shadow | grep didiaoyuan
didiaoyuan:$6$le4/DwzY$vUNqGXtv.5DKtm8lE6C7SBAMw9eHMhWtI794lMWSFfxYMuzAZF9KuIxdlK7YrwdJgEDJmYfcBihwd11MvJMfq/:18542:0:99999:7:::

```

- `-L` 冻结用户
- `-U`解冻用户 

### 4 userdel 删除用户

```shell
[root@centos-7 home]# userdel didiaoyuan

# -r 删除用户并删除家目录
[root@centos-7 home]# userdel -r test

```

- 默认删除会删除 `/etc/passwd` 和 `/etc/shadow`的文件，为了安全考虑保留家目录
- `-r`参数删除对应的家目录



### 5 切换用户

**su 默认 root 用户切换**

```shell
[parallels@centos-7 ~]$ su
Password: 
[root@centos-7 parallels]# 

# 添加 - 参数切换并进入 root 环境（其实就是一进去 /root 目录）
[parallels@centos-7 ~]$ su -
Password: 
Last login: Tue Oct 13 21:08:05 CST 2020 on pts/0
[root@centos-7 ~]# 

# su 后面添加指定切换的用户
```

> ⚠️  root 环境下使用 su 切换用户可以不需要输入密码

**sudo 可配置的方式执行的命令**

> sudo 以 root 身份去操作命令

`sudo passwd user` 使用 root 的身份修改 user 的密码。运行这个命令，系统首先检查 `/etc/sudoers` ,判断该用户是否有执行 sudo 的权限，如果有，则 root 身份运行 `passwd user`

```shell
# visudo 修改 sudo 命令，并且保存后 自动检查语法设置
[parallels@centos-7 etc]$ sudo visudo
...
##      user    MACHINE=COMMANDS
##
## The COMMANDS section may have other options added to it.
##
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
...
```

> 在上述 root 下方添加用户，添加的用户就能使用 sudo root身份操作命令
>
> - 第一列：这个用户
> - 第二列：可以任何地方登陆
> - 第三列：任何人
> - 第四列：执行任何命令
>
> `%groupName ALL=(ALL) ALL`表示所属的组...
>
> 如果不想每次添加密码 `xxx ALL=（ALL）NOPASSWD：ALL`
>
> 也可定义专门的命令 `xxx ALL=（ALL）NOPASSWD：/sbin/shutdown /user/bin/reboot` 比如关闭服务器或者重启



## Linux 用户组管理

```shell
# 用户组信息记录在 /etc/group文件中
cat /etc/group
```

### 1 groupadd 增加用户组

```shell
$ groupadd testGroup
$ cat /etc/group
...
testGroup:x:1001:
# 用户组名:密码:数字id:组成员
```

### 2 groupdel 删除用户组



```shell
$ groupdel testGroup
```

> ⚠️ 如果已有用户属于试图删除的组，该操作会失败。



## Linux 检查用户信息

### 1 users、who、w 查看用户

```shell
# 查看当前系统有哪些用户
[root@centos-7 parallels]# users
parallels parallels
# who 查看会话
[root@centos-7 parallels]# who
parallels :0           2020-10-12 21:31 (:0)
parallels pts/0        2020-10-12 21:31 (:0)
# w 查看更详细信息
[root@centos-7 parallels]# w
 21:41:10 up 10 min,  2 users,  load average: 0.02, 0.19, 0.18
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
parallel :0       :0               21:31   ?xdm?  46.41s  0.19s /usr/libexec/gnome-session-binary --session gnome-classic
parallel pts/0    :0               21:31    0.00s  0.20s  2.17s /usr/libexec/gnome-terminal-server

```

**w显示字段**

- 登陆用户的用户名
- 用户登陆终端
- 从网络登陆，显示远程主机或ip地址
- 用户登陆时间
- 用户闲置时间
- 当前 WHAT 列所对应进程消耗的 CPU时间
- 用户当前运行的进程

### 2 finger 调查用户

> 我的没有，需要安装，既然默认系统没有带，那就演示了



- 显示系统登陆用户信息
- 命令跟对应用户显示多一些的信息

## Linux 查看命令

### man 查看命令帮助

```shell
# 查看 ls 命令的帮助信息
man ls

```

- 上下左右查看
- 空格键翻页
- /xxx 查询标亮显示
- 小写 n 向下，大写 N 向上
- q 推出

> Linux 规定 9 个man文件的种类
>
> - 常见命令说明
> - 可调用的系统
> - 函数库
> - 设备文件
> - 文件格式
> - 游戏说明
> - 杂项
> - 系统管理员可用的命令
> - 与内核相关说明
>
> 当我们查询一个命令存放在哪，可以使用
>
> `man -f reboot`
>
> ```shell
> [root@centos-7 boot]# man -f reboot
> reboot (2)           - reboot or enable/disable Ctrl-Alt-Del
> reboot (8)           - Halt, power-off or reboot the machine
> 
> # 选择查看使用 序号
> man 2 reboot
> ```

### ls 查看目录

```shell
# 查看目录
ls
## 详细显示所有目录
ls -l
## 详细显示指定文件
ls -l xxx
## 显示隐藏文件


```

### cat 查看文件内存

```shell
# 显示内容
cat 文件名

```

### id 查看的 uid

```shell
[root@centos-7 ~]# id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

```



### groups 查看自己的组

```shell
[root@centos-7 ~]# groups
root

```



### who 查看在线用户

```shell
[parallels@centos-7 etc]$ who
parallels :0           2020-10-13 21:06 (:0)
parallels pts/0        2020-10-13 21:17 (:0)

```



# Linux 例行任务管理

> Linux 中有两种处理定时任务的方法
>
> - 任务周期性执行 `cron`
> - 某个特定的时间执行一次 `at`



## at

```shell
# 三十分钟后自动关机 第一行回车 第二行执行的动作
# 第三行 ctrl+D 表示输入结束 第四行表示执行时间
at now + 30 minutes 
at> /sbin/shutdown -h now
at> <EOT>
job 1 at Tue Oct 13 21:35:00 2020
# atq 查看队列任务
[parallels@centos-7 sbin]$ atq
2	Tue Oct 13 21:38:00 2020 a parallels
# atrm 2 删除号码为 2 的任务
[parallels@centos-7 sbin]$ atrm 2
[parallels@centos-7 sbin]$ atq
[parallels@centos-7 sbin]$ 

# 如果禁用某些用户这个功能，可以将用户名添加 /etc/at.deny 中
```



## cron

- 首先确定 crond 进程在运行
- 通过 `crontab` 来设置自己的计划任务，`-e`参数编辑任务

```shell
# 首先确定 crond 进程在运行
[parallels@centos-7 sbin]$ service crond start
Redirecting to /bin/systemctl start crond.service


[parallels@centos-7 sbin]$ crontab -e
no crontab for parallels - using an empty one
crontab: installing new crontab

# crontab -l 查看任务
[parallels@centos-7 sbin]$ crontab -l
* * * * * service httpd restart

# crontab 语法格式 分钟 小时 日期 星期 command
# 每分钟启动 httpd 进程



# crontab -r 删除说要任务

# root 用户查看其他用户的任务列表 crontab -u user -l	

# 如果禁用某些用户这个功能，可以将用户名添加 /etc/cron.deny 中
```



**/etc/crontab 的管理**

> 直接在这个文件里定义周期性任务

```shell
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

~                                                                               
~                                              
```

# Linux 文件管理

> FHS 文件系统层次标准，定义了目录下的主要目录以及每个目录应该存放什么文件

| 目录        | 目录的用途                                             |
| ----------- | ------------------------------------------------------ |
| /bin        | 常见的用户指针                                         |
| /boot       | 内核和启动文件                                         |
| /dev        | 设备文件                                               |
| /etc        | 系统和服务的配置文件                                   |
| /home       | 系统默认的普通用户的家目录                             |
| /lib        | 系统函数库目录                                         |
| /lost+found | ext3 文件系统需要的目录，用于磁盘检查(centOS 貌似没有) |
| /mnt        | 系统加载文件系统时常用的挂载点                         |
| /opt        | 第三方软件安装目录                                     |
| /proc       | 虚拟文件系统                                           |
| /root       | root 用户的家目录                                      |
| /sbin       | 存放系统管理命令                                       |
| /tmp        | 临时文件的存放目录                                     |
| /usr        | 存放与用户相关的文件和目录                             |
| /media      | 系统用来挂载光驱等临时文件系统的挂载点                 |

## 1 绝对路径与相对路径

**1 绝对路径**

> Linux 从 `/`根目录开始寻找，从`/`开始的一定是绝对路径



**2 当前目录:pwd**



**3 特殊目录(.)和(..)**

> 每个目录下，都会固定存在两个特殊目录，`(.)`目录代表当前目录，两个`(..)`代表的是当前目录的上层目录。**在 Linux 下，所有以点开始的文件都是隐藏文件**，使用 `ls -a` 才可以看到

**4 相对路径**

> 假设当前目录在 `/usr/local`下，那么它的上层目录(/usr目录) 可以用 `../`表示,而 `/usr/local`下的 src 目录可以用 `./src`

## 2 文件的相关操作

介绍文件的:

- 创建
- 删除
- 移动
- 重命名
- 查看文件

### 1 创建文件：touch

```shell
[parallels@centos-7 Desktop]$ touch test.txt
[parallels@centos-7 Desktop]$ ls -l test.txt 
-rw-rw-r--. 1 parallels parallels 0 Oct 14 20:35 test.txt
[parallels@centos-7 Desktop]$ touch test.txt
[parallels@centos-7 Desktop]$ ls -l test.txt 
-rw-rw-r--. 1 parallels parallels 0 Oct 14 20:43 test.txt

```

> ⚠️：如果已存在文件，不会修改文件的内容，但会**更新文件的创建时间属性。

### 2 删除文件：rm

```shell
[parallels@centos-7 Desktop]$ rm test.txt 
```



### 3 移动或重命名文件：mv

参数1：被移动的文件 参数2 移动到的目录

```shell
# mv 还能重命名文件
[parallels@centos-7 Desktop]$ touch test.txt
[parallels@centos-7 Desktop]$ mv test.txt test.dic
[parallels@centos-7 Desktop]$ ls 
Parallels Shared Folders  test.dic

# mv 还能在移动的同时重命名文件
[parallels@centos-7 Desktop]$ mkdir ADocument
[parallels@centos-7 Desktop]$ mv test.dic ./ADocument/test.txt
[parallels@centos-7 Desktop]$ ls ./ADocument/
test.txt

```

### 4 查看文件：cat



### 5 查看文件头：head

> 有时候文件非常大，使用 cat 命令显示出来的内容太多，可以使用 head 查看具体行数

```shell
# -n 参数指定显示行数
head -n 10
```

### 6 查看文件尾：tail

> tail 命令与 head 命令非常类似。最重要的功能是可以**动态地查看文件尾**。查看日志文件使用 -f 参数就可以做到，一旦有新的日志写入，该命令会立即将新内容显示出来。

```shell
tail -f /var/log/messages
```

### 7 文件格式转换：dos2unix

> DOS to UNIX 简写，可以把 DOS 格式的文本转变为 UNIX 下的文本文件。场景是 Linux 和 Windows 系统是可以通过文件共享的方式共享文件的，当把 Windows 下的文本文件移动到 Linux 下，会由于系统之间**文本文件的换行符不同而造成文件在 Linux 下的读写操作有问题**。命令使用后直接加上需要转换的文件名。

## 3 目录相关的操作

### 1 进入目录：cd

### 2 创建目录：mkdir

> 使用 `-p`参数一次性创建所有目录 `mkdir -p /a/b/c/d`

### 3 删除目录：rmdir和rm

```shell
# 如果存在文件则不能直接删除,需要从下往上删除
[parallels@centos-7 Desktop]$ rmdir test
rmdir: failed to remove ‘test/’: Directory not empty
# 如果层级特别的就很低效，使用 rm 命令 
[parallels@centos-7 Desktop]$ rm -r test/
```

> ⚠️ rm 中 `-r` 递归删除目录层级，单每一层会有提示，所以常用 `-rf`
>
> 删除命令几乎是不可能恢复的，所以权限很高 
>
> 禁止运行`rm -rf /` ，一定是暗示什么...

### 4 文件和目录复制：cp

```shell
# 直接复制,也可以重新命名文件
cp abc.log /test/
# 复制目录增加 -r 参数
cp -r /a /b
```

