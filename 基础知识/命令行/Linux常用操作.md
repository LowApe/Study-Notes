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