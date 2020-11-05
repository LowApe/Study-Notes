## Linux 常用操作





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

### 5 文件时间戳

> 之前 两次 touch 文件，最后一次虽然没有修改文件，但是文件的修改属性，时间戳是最新的，实际上目录一包含时间戳这一特性，利用这一特性可以实现有选择的备份。**比如：我们有两个目录，一个目录是目标目录，包含三个文件，一个目录是备份目录，现在使用 cp 将三个文件复制到备份目录中，如果未来我们希望目标目录变化的文件才进行备份，我们可以利用文件时间戳这一特性，比较两个位置的文件时间戳，如果不相同则进行备份**

## 4 Linux 文件和目录权限

> Linux 系统之所以更安全，是因为对文件权限有着非常严格的控制

### 1 查看文件或目录的权限 ls -al

`ls -al`命令虽然能查看目录文件的详细信息，但其中

- 第一列表示权限的。

- 第二列代表'"连接数"，除了目录文件之外，其他所有文件的连接数都是 1 ，目录文件的连接数是目录包含目录总个数+2(. 和 ..)

- 第三、四列代表文件的所有人、所有组

- 第五列是该文件的大小

- ...

	

```shell
[parallels@centos-7 Desktop]$ ls -al
total 20
drwxr-xr-x.  2 parallels parallels    73 Oct 14 21:18 .
drwx------. 18 parallels parallels  4096 Oct 14 20:55 ..
-rw-------.  1 parallels parallels 12288 Oct 14 20:55 .abc.swp
-rw-rw-r--.  1 parallels parallels  3307 Oct 14 20:55 install.log
lrwxrwxrwx.  1 root      root         10 Oct  6 15:02 Parallels Shared Folders -> /media/psf

```

权限这一列，有 10 个字符组成，每个字符表明该文件的类型。我们分别说明;

**这10个字符，排除第一个字符有特殊值代表的含义，之后每3个字符为一组**

| 第一个字符可能的值 | 含义       |
| ------------------ | ---------- |
| d                  | 目录       |
| -                  | 普通文件   |
| l                  | 链接文件   |
| b                  | 块文件     |
| c                  | 字符文件   |
| s                  | socket文件 |
| p                  | 管道文件   |



- 2~4 **代表该文件所有者(usr)的权限** 
- 5~7 **代表文件所有组(group)的权限**
- 8~10 **代表其他用户(others)拥有的权限**
- 每组都是 `rwx` 的组合，
	- 如果拥有读权限，每组的第一个字符显示 r，否则是小横线
	- 如果拥有写权限，每组第二个字符显示 r，否则是小横线
	- 如果拥有执行权权限，每组第三个字符显示x,否则是小横线

### 2 文件隐藏属性

> `lsattr`查看文件隐藏属性

```shell
[parallels@centos-7 Desktop]$ lsattr install.log 
---------------- install.log

```

结果 13 小短横，每个短横是一个属性。

```shell
[parallels@centos-7 Desktop]$ chattr +a install.log
```

> `chattr` 设置隐藏属性，a 属性的文件，即便 root 用户也不能删除它，单可以尾部继续写入内容
>
> 还有 i 属性，不能写入、改名、删除，文件隐藏属性对提升系统安全性有较大的帮助，更多隐藏属性使用 `man chattr`查看

### 3 改变文件权限: chmod

下面命令 u 、g、o 分别表示**用户、拥有组、其他人**，而具体权限使用 `rwx` 的组合来定义，增加权限`+`，删除权限使用`-`号，详细权限使用`=`号

| 作用                                                     | 命令                   |
| -------------------------------------------------------- | ---------------------- |
| 给某文件添加用户读权限                                   | `chmod u+r fileName`   |
| 给某文件删除用户读权限                                   | `chmod u-r fileName`   |
| 给某文件添加用户写权限                                   | `chmod u+w fileName`   |
| 给某文件删除用户写权限                                   | `chmod u-w fileName`   |
| 给某文件添加用户执行权限                                 | `chmod u+x fileName`   |
| 给某文件删除用户执行权限                                 | `chmod u-x fileName`   |
| 添加用户对某文件的读写执行权限                           | `chmod u+rwx fileName` |
| 删除用户对某文件的读写执行权限                           | `chmod u-rwx fileName` |
| 给某文件设定用户拥有读写执行权限<br />(怎么感觉是一样的) | `chmod u=rwx fileName` |

> ⚠️:如果要给用户或其他人添加或删除相关权限，只需要将上面的u 换成相应的 g 或者 o
>
> - 如果同时设置所有人的权限就需要使用**数字表示法**，我们定义 `r=4,w=2,x=1,`,如果权限是 `rwx`，则数字表示为 7，如果权限是`r-x`,则表示 5
> - 如果设定一个文件的权限是: 拥有者的权限是`读、写、执行(rwx)`，拥有组的权限是读、执行(r-x),其他人的权限是只读(r--)，那么可以使用命令 `chmod 754 fileName`来设置
> - 如果需要修改的不是一个文件而是一个目录，使用 **-R**参数递归赋予目录下的权限

### 4 改变文件的拥有者: chown

> 改变文件拥有者、拥有组的功能。**默认情况下，使用什么用户登陆系统，那么该用户新创建的文件和目录的拥有者就是这个用户**

| 修改                     | 命令                           |
| ------------------------ | ------------------------------ |
| 改变文件的拥有者         | `chown user filename`          |
| 改变文件的拥有组         | `chown :group filename`        |
| 改变文件的拥有者和拥有组 | `chown user:group filename`    |
| 改变目录下的             | `chown -R user:group dirctory` |

### 5 改变文件的用户组 chgrp

### 6 文件特殊属性: SUID/SGID/Sticky

> 由用户修改密码，对 passwd 和 shadow 访问权限，引出 suid权限，意味着用户可以使用 root 权限的身份来执行这个命令

**SUID**

- SUID 权限只能用于二进制
- 给一个二进制文件添加 SUID 权限 `chmod u+s fileName` 之前的 rwx 现在多了一个 s 权限，代表，该二进制文件可以使用 root 权限身份来执行🤔

**SGID**

> 如果某个二进制文件的**用户组设置了s权限**，该用户组的用户都以该文件的用户身份去运行这个命令
>
> 命令：`chmod g+s fileName`

**Sticky**

> **Sticky 权限只能用于设置在目录**上，设置了这种权限的目录，任何用户都可以在该目录中创建或修改文件，但是**只有该文件的创建者和 root 可以删除自己的文件**
>
> - RedHat 或 CentOS 系统中的 `/tmp` 目录就拥有 Sticky 权限(权限部分最后显示 t)

```shell
[parallels@centos-7 Desktop]$ ll -d /tmp/
drwxrwxrwt. 13 root root 4096 Nov  4 08:18 /tmp/

```

### 7 默认权限和遮罩 umask 

- **root 创建文件的默认权限是 644 创建目录的默认权限是 755**
- 普通用户 创建文件的默认权限是 664 创建目录的默认权限是 775

> umask 中文翻译为 遮罩。在 Linux 下，定义目录创建的默认权限的值是 "umask 遮罩 777后的权限"，定义文件创建的默认权限是 "umask 遮罩 666后的权限"

系统 `/etc/profile` 文件中，有段代码设置了不同用户的遮罩值

```sh
if [ $UID -gt 199 ] && [ "`/usr/bin/id -gn`" = "`/usr/bin/id -un`" ]; then
    umask 002
else
    umask 022
fi
```

> UID 大于 199 用户设置了 umask 为 002，否则为 022。所以 umask 值对于 root 用户为 022 ，普通用户为 022

**遮罩计算权限**

- 777 用字符串表示:rwxrwxrwx，如果遮罩值为022 ，用字符串表示 ----w--w-，第五位和第八位遮罩掉，权限就变味 rwxr-xr-x 权限数字表示 755. 002遮罩后为 775

### 8 查看文件类型 file

> file 文件可以查看更多文件信息

- ```shell
	# 文件夹类型
	[parallels@centos-7 Desktop]$ file tesst/
	tesst/: directory
	# /tmp 是一个拥有 sticky 属性的目录
	[parallels@centos-7 Desktop]$ file /tmp/
	/tmp/: sticky directory
	# ls -l 命令查看，显示是一个普通文件
	[parallels@centos-7 Desktop]$ ls -l /etc/passwd
	-rw-r--r--. 1 root root 2314 Oct 12 21:32 /etc/passwd
	
	# file 命令查看，显示这是一个普通的
	[parallels@centos-7 Desktop]$ file /etc/passwd
	/etc/passwd: ASCII text
	# 使用 ls -l 命令查看，显示普通文件
	[parallels@centos-7 Desktop]$ ls -l /usr/bin/passwd 
	-rwsr-xr-x. 1 root root 27832 Jun 10  2014 /usr/bin/passwd
	# file 命令。显示这是一个 64 位的可执行性二进制文件
	[parallels@centos-7 Desktop]$ file /usr/bin/passwd
	/usr/bin/passwd: setuid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=1e5735bf7b317e60bcb907f1989951f6abd50e8d, stripped
	
	```


## 5 Linux 查找文件

### 1 一般查找: find

```shell
# 一般查找 语法(精确查找)
find PATH -name FILENAME

# 在系统中找到, / 该命令的意思是，从根目录开始寻找，遍历所有文件 . 表示从当前目录查找
find / -name filename

# * 星号通配符，模糊查找
[parallels@centos-7 Desktop]$ find . -name install*
./install.log

```

| 参数                  | 含义                                        |
| --------------------- | ------------------------------------------- |
| -name filename        | 查找文件名文件                              |
| -perm 777(数字表示法) | 根据文件权限查找                            |
| -user username        | 根据用户名查找                              |
| -**m**time -n/+n      | 查找 n 天内/ n 天前**更改**过的文件         |
| -**a**time -n/+n      | 查找 n 天内/ n 天前**访问**过的文件         |
| -**c**time -n/+n      | 查找 n 天内/ n 天前**创建**的文件           |
| -newer filename       | 查找更改时间比 filename 新的文件            |
| -type b/d/c/p/l/f/s   | 查找 块/目录/字符/管道/链接/普通/套接字文件 |
| -size                 | 根据文件大小查找                            |
| -depth n              | 最大的查找目录深度                          |

> 🤔：
>
> - 事件查询条件，n天前？？？
>
> - `find . -depth 1` 报错？



### 2 数据库查找: locate

与 find 不同，**locate 命令来依赖与一个数据库文件**

> Linux 系统默认每天会检索一下系统中的所有文件，然后将检索到的文件记录到数据库中。在运行 locate 命令比 find 速度快，因为直接将数据库中的数据打印到屏幕上。
>
> 在使用 locate 前使用 `updatedb` 命令(系统每天会自动运行一次)，将当天创建的信息更新到数据库中

```shell
[parallels@centos-7 Desktop]$ sudo updatedb
[parallels@centos-7 Desktop]$ locate hello.txt
/home/parallels/Desktop/test2/hello.txt
```

### 3 查找执行文件: which/ whereis

> which 用于从系统的 PATH 变量所定义的目录中查找可执行文件的绝对路径。

```shell
[parallels@centos-7 Desktop]$ which passwd
/usr/bin/passwd
```

> 使用 whereis 也能查到其路径但是和 which 不同的是，它**不但能找到其二进制文件，还能找出相关的 man 文件：**

```sh
[parallels@centos-7 Desktop]$ whereis passwd
passwd: /usr/bin/passwd /etc/passwd /usr/share/man/man1/passwd.1.gz /usr/share/man/man5/passwd.5.gz
```

## 6 Linux 文件压缩和打包

### 1 gzip/gunzip

> gzip/gunzip 是用来压缩和解压缩**单个文件的工具**

```sh
[parallels@centos-7 Desktop]$ gzip install.log 
[parallels@centos-7 Desktop]$ ls
install.log.gz  Parallels Shared Folders  tesst  test2
[parallels@centos-7 Desktop]$ gunzip install.log.gz 
install.log  Parallels Shared Folders  tesst  test2
```

### 2 tar

> tar 不但可以打包文件，还可以将**整个目录中的全部文件整合成一个包**,整合包的同时还使用 gzip 的功能进行压缩。整合后的包习惯使用 `.tar`作为其后缀名，使用 gzip 压缩后的后缀使用 `.gz`作为其后缀名。因为 tar 有同时整合和压缩的功能，所以可使用`.tar.gz`作为后缀名，或者简写为`.tgz`

```shell
[parallels@centos-7 Desktop]$ ls
install.log  Parallels Shared Folders  tesst  test2
[parallels@centos-7 Desktop]$ tar -zcvf testTar.tgz test2
test2/
test2/hello.txt
[parallels@centos-7 Desktop]$ ls
install.log  Parallels Shared Folders  tesst  test2  testTar.tgz
```

> - -z 表示使用 gzip 压缩
> - -c 创建压缩文件
> - -v 显示当前被压缩的文件
> - -f 指使用文件名

```shell
[parallels@centos-7 Desktop]$ tar -zxvf testTar.tgz 
test2/
test2/hello.txt
```

> - -z 解压
> - -C 指定目录解压

### 3 bzip2

> 使用 bzip2 压缩文件时,默认会产生以`.bz2`扩展名结尾的文件，这里使用`-z`参数进行压缩，使用`-d`参数进行解压缩

```shell
bzip2 install.log
[parallels@centos-7 Desktop]$ ls -l install.log.bz2 
-rwxrw-r--. 1 parallels parallels 560 Oct 14 20:55 install.log.bz2
[parallels@centos-7 Desktop]$ bzip2 -d install.log.bz2 
[parallels@centos-7 Desktop]$ ls
install.log  Parallels Shared Folders  tesst  test2  testTar.tgz
[parallels@centos-7 Desktop]$ 

```

### 4 cpio

> 该命令和 find 命令一同使用。**当由 find 按照条件找出需要备份的文件列表后，可通过管道的方式传递给 cpio 进行备份，生成 /tmp/conf.cpio 文件夹**，然后再将生成的 `/tmp/conf.cpio` 文件中包含的文件列表完全还原回去

```shell
[parallels@centos-7 Desktop]$ find /etc -name *.conf | cpio -cov > /tmp/conf.cpio
[parallels@centos-7 Desktop]$ cpio --absolute-filenames -icvu < /tmp/conf.cpio
```

> ⚠️：man cpio 查看命令帮助

# Linux 文件系统

## 1 文件系统

Linux 使用了树形文件存储结构，在磁盘上存储文件的时候，使用的则 **目录加文件的形式**。但磁盘的存储是磁性 0 1，这里有个明显的问题 :**磁盘的物理存储方式决定了其根本没有文件和目录的概念**。

> Linux 采用的是文件系统+虚拟文件系统(VFS)的解决方案

### 1 什么是文件系统

> 磁盘上组织文件的方法。在使用前，都需要针对磁盘做初始化操作，并将记录的数据结构写入磁盘上，这种操作就是建立文件系统。Linux 支持多种文件系统格式 ext2、ext3、ext4、zfs、iso9660、vfat、msdos 、smnfs、nfs

大部分Linux 系统都有通用结构

- 超级块(superblock)
- i节点(inode)
- 数据块(data block)是存放真实文件数据 4kb 单位
- 目录块(directory block)等是存放文件名和文件在目录中的位置，并包括文件的 i 节点信息

### 2 ext2 文件系统

弱点：不支持日志功能。这很容易造成一些情况下丢失数据

### 3 ext3 文件系统

兼容 ext2，带有日志功能

## 2 磁盘分区、创建文件系统、挂载

```flow

op=>operation: 磁盘分区
op2=>operation: 创建文件系统
op3=>operation: 挂载
op4=>operation: 设置启动自动挂载

op->op2
op2->op3->op4

```



磁盘分区

- 主分区
- 扩展分区

分区表大小(MBR 512字节，其中分区表站64字节)

> 在完成磁盘分区后，需要进行创建文件系统的操作，最后将该分区挂载到系统中的某个挂在点才可以使用

### 1 创建文件系统：fdisk

```shell
# 查看磁盘在操作系统中对应的设备
fdisk -l

# 分区
fdisk /dev/sdb

Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x99fef6e7.

The device presents a logical sector size that is smaller than
the physical sector size. Aligning to a physical sector (or optimal
I/O) size boundary is recommended, or performance may be impacted.
# 输入 n
Command (m for help): n
# 输入 p
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-132118527, default 2048): 2222
w
```

假设创建了一个 `/dev/sdb2`分区，使用命令`mkfs -t ext3/dev/sdb2`**为创建的分区中格式 ext3 文件系统**

### 2 磁盘挂载：mount

创建了文件系统的分区后，在 Linux 系统下还需要经过挂载才能使用，

```shell
# 语法格式 DEVICE ： 具体的设备 MOUNT_POINT 是指挂载点,挂载点只能是目录
mount DEVICE MOUNT_POINT

# mount 没有参数则显示所有挂载
/dev/disk0s2 on / (hfs, local, journaled)
devfs on /dev (devfs, local, nobrowse)
map -hosts on /net (autofs, nosuid, automounted, nobrowse)
map auto_home on /home (autofs, automounted, nobrowse)
# 显示设备组件
juejianangdemac:~ mac$ df -h | grep disk0s2
/dev/disk0s2   233Gi  195Gi   38Gi    84% 2462560 4292504719    0%   /
juejianangdemac:~ mac$

```

### 3 设置启动自动挂载:/etc/fstab 🌟

> mount 命令挂载的设备在你重启计算机之后就会消失，所以必须通过 `/etc/fstab` 使得系统在重启后能自动挂载

```shell
echo "/dev/sdb1 /root/newDisk ext3 defaults 0 0" >> /etd/fstab
```

这条命令的意思显而易见:`/dev/sdb1`挂载到 `/root/newDisk` ，文件系统是 `ext3`，使用系统默认的挂载参数(defaults)，第五部分是决定 dump 命令在进行备份时是否对该设备进行 `fsck`(命令用于检查与修复 Linux 档案系统)，这个值只可能是 3 种。 1 保留给根分区，其他分区使用 2（检查完根分区后检查）或 0(不检查)

### 4 磁盘检验: fsck 、badblocks

当机器突然掉电时可能引起。该命令的经典使用

```shell
fsck -t TYPE(ext2|ext3) /DEVICE/PATH
```

> ⚠️：fsck 在检查磁盘的时候，**需要磁盘时未挂载的状态，否则会造成文件损坏**
>
> 对于已挂载的设备需要先进行 `umount`操作

```shell
# 解除挂载设备
umount /DEVEICE/PATH
# 解除挂载点
umount MOUNT_POINT
```

**系统的根文件系统出现问题需要fsck怎么办呢?**

> 因为系统在运行时，根时无法被 umount 的。这使用只有重启计算机，因为确认根文件系统出现问题，系统重启会检测到问题，然后提示**用户输入 root 密码进入单用户模式**。然后使用 fsck 来修复根文件系统.🤔：root 用户的文件啥的不都是放在 root 文件吗，有问题还怎么登陆



**badblocks 检测磁盘的物理坏道，一般怀疑磁盘有坏道的时候才使用**

## 3 Linux 逻辑卷

> 在一个分区经过挂载使用后，随着存储文件的不断增多，可用空间越来越少。如果原先分配的磁盘空间不够使用的情况，**这时候是没有办法扩大这个分区**
>
> 为了更好地使用磁盘空间，提高系统空间的可扩展性，此时就需要使用逻辑卷

### 1 什么是逻辑卷

使用逻辑卷组(Logic Volume Manager)创建出来的设备，也是 Linux 下操作系统可以认识的设备。**LVM介于硬盘裸设备和文件系统的中间层**

- 物理卷(Physical Volume PV) 物理磁盘分区，比如说`dev/sdb1`
- 卷组(Volume Group，VG)也就是 PV 的集合
- 逻辑卷(Logic Volume,LV)也就是 PV 中划出来的一 小块逻辑磁盘

> **逻辑卷就是从某个物理卷中抽象出来的一块磁盘空间**

```flow
op1=>operation: PV
op2=>operation: VG
op3=>operation: LV
op1->op2->op3
```

> :sos:目前还不知道依次创建卷组然后到LV 的目的是什么​

### 2 如何制作逻辑卷



**创建物理卷：pvcreate、pvdisplay**

先将物理卷fdisk 创建一个分区，各个分区的 ID 值是 83，还需要更**改 ID 值为 8e**，表明该分区是一个特殊的用于逻辑卷管理的分区

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gkel8qpbxyj310i03ymyd.jpg)

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gkelaswbmgj318u0t2h2d.jpg)

```shell
fdisk /dev/DEVEICE
# t 表示修改当前设备下的分区
t
# 输入西拐的分区数
1 
# 查看当前修改的编码
Hex code (type L to list all codes): L

 0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris        
 1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden C:  c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx         
 5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data    
 6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility   
 8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt         
 9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access     
 a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O        
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor      
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi eb  BeOS fs        
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         ee  GPT            
 f  W95 Ext'd (LBA) 54  OnTrackDM6      a6  OpenBSD         ef  EFI (FAT-12/16/
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        f0  Linux/PA-RISC b
11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f1  SpeedStor      
12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f4  SpeedStor      
14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f2  DOS secondary  
16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      fb  VMware VMFS    
17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fc  VMware VMKCORE 
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fd  Linux raid auto
1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fe  LANstep        
1c  Hidden W95 FAT3 75  PC/IX           be  Solaris boot    ff  BBT            
1e  Hidden W95 FAT1 80  Old Minix   
# w 将修改写入分区表
w
```

> 这样分区就具备称为PV的条件，使用 `pvcreate /dev/DEVEICE`

**pvscan 查看系统中的 PV pvdisplay 查看更详细的内容**

```shell
[parallels@centos-7 Desktop]$ sudo pvscan
  PV /dev/sda2   VG cl              lvm2 [<63.00 GiB / 4.00 MiB free]
  Total: 1 [<63.00 GiB] / in use: 1 [<63.00 GiB] / in no VG: 0 [0   ]
[parallels@centos-7 Desktop]$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               cl
  PV Size               <63.00 GiB / not usable 3.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              16127
  Free PE               1
  Allocated PE          16126
  PV UUID               GZXwoc-7Zia-AdE3-ZFwD-SUgd-8hK3-j003qr
   
```

**创建并查询卷组:vgcreate、vgdispaly**

有了 PV 就可以创建卷组了。命令 vgcreate 的用法如下(其中 VG_NAME) 是创建 VG 名

```shell
# 创建 VG 语法
vgcreate VG_NAME DEVICE1 ... DEVICEn
# 查看当前系统上的所有 VG
[parallels@centos-7 Desktop]$ sudo vgscan
  Reading volume groups from cache.
  Found volume group "cl" using metadata type lvm2
# 查看详细VG 信息
[parallels@centos-7 Desktop]$ sudo vgdisplay
  --- Volume group ---
  VG Name               cl
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <63.00 GiB
  PE Size               4.00 MiB
  Total PE              16127
  Alloc PE / Size       16126 / 62.99 GiB
  Free  PE / Size       1 / 4.00 MiB
  VG UUID               joX1Bf-Eo5C-s0TL-UcAT-2f7q-E3Tf-Ki1xkG

```

**扩容卷组:vgextend**

使用过程中如果要扩大 VG，可以使用 `vgextend`

```shell
vgextend VG_NAME DEVICE1 ... DEVICEn
```

**创建逻辑卷:lvcreate、lvdisplay**

有了卷组(First_VG),可以创建逻辑卷了，在创建逻辑卷时需要 **定义大小、名称，以及使用哪个卷组的空间等信息，**

```shell
# -L 逻辑卷的大小 -n 逻辑卷名称 最后从什么卷组中分配空间
lvcreate -L SIZE -n LV_NAME VG_NAME
```

LV 创建后和物理分区一样，逻辑卷也需要在创建文件系统、挂载后才能被系统使用， **对逻辑卷创建文件系统的时候，其全路径是 /dev/卷组名/逻辑卷名**



```shell
# 创建文件系统
mkfs.ext3 /dev/VG/LV 
# 创建一个挂载点
mkdir /root/newLV
# 挂载
mount /dev/VG/LV /root/newLV 
```

## 4 硬链接和软链接

### 1 什么是硬链接

> 硬链接又称为实际链接，是指**通过索引节点**来进行链接。在 Linux 文件系统中，所有的文件都会有一个编号，称为 inode ，多个文件名指向同一索引节点被允许的，这种链接就是硬链接。**硬链接的作用是允许一个文件拥有多个有效路径名，这样用户就可以创建硬链接指向同一文件，删除一个链接并不会影响索引节点本身和其他的链接，只有最后一个链接被删除时，文件的数据块及目录的链接才会被释放。**

硬链接有两个限制：

- 不允许给目录创建硬链接
- 只有在同一文件系统中的文件之间才能创建链接，即不同分区上的两个文件之间不能够建立硬链接

```shell
[parallels@centos-7 test2]$ ls -li
total 0
322 -rw-rw-r--. 1 parallels parallels  0 Nov  5 21:37 hard01
[parallels@centos-7 test2]$ ln hard01 hard01_hlink
[parallels@centos-7 test2]$ ls -li
total 0
322 -rw-rw-r--. 2 parallels parallels  0 Nov  5 21:37 hard01
322 -rw-rw-r--. 2 parallels parallels  0 Nov  5 21:37 hard01_hlink

```

> 创建硬链接后发现，第三列的值发生了变化从 1 变为 2 **这个值时源文件的关联数，当次数为 0 的时候该文件才能正真地被文件系统创建**

## 2 什么是软链接

是一个包含了另一个文件路径名的文件，**可以指向任意文件或目录**，也可以跨不同的文件系统。。

> 软链接和 Windows 下的 "快捷方式"十分类似，删除软链接并不会删除其所指向的源文件，如果删除了源文件则会出现"断链"

```shell
[parallels@centos-7 test2]$ ln -s file01 file01_slink
[parallels@centos-7 test2]$ ls -l
total 0
-rw-rw-r--. 1 parallels parallels  0 Nov  5 21:55 file01
lrwxrwxrwx. 1 parallels parallels  6 Nov  5 21:58 file01_slink -> file01

```

> 创建软链接需要使用 -s 参数 

