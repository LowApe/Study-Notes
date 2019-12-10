# 理解Linux文件权限

本章内容

- 理解Linux 的安全性
- 解读文件权限
- 使用 Linux 组

Linux 沿用了 Unix 文件权限的办法，即允许用户和组根据每个**文件和目录**的安全性**设置**来访问文件。如何在必要时利用 Linux 文件安全系统保护和共享数据

## Linux 的安全性

Linux 安全系统的核心是**用户账户**。每个能进入 Linux 系统的用户都会被分配唯一的用户账户。用户对系统中各种对象的访问权限取决于他们登陆系统时用的账户

用户权限是通过创建用户时分配的用户 ID (User ID)，Linux 系统使用特定的文件和工具来跟踪和管理系统上的用户账户。在我们讨论文件权限之前，先来看一下 Linux 是怎么**处理用户账户**

### /etc/passwd文件

Linux 系统使用一个专门的文件来将用户的登录名匹配到对应的 UID 值。这个文件就是`/etc/passwd`，它包含了一些与用户有关的信息

```shell
nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false
root:*:0:0:System Administrator:/var/root:/bin/sh
daemon:*:1:1:System Services:/var/root:/usr/bin/false
_uucp:*:4:4:Unix to Unix Copy Protocol:/var/spool/uucp:/usr/sbin/uucico
_taskgated:*:13:13:Task Gate Daemon:/var/empty:/usr/bin/false
_networkd:*:24:24:Network Services:/var/networkd:/usr/bin/false
_installassistant:*:25:25:Install Assistant:/var/empty:/usr/bin/false
_lp:*:26:26:Printing Services:/var/spool/cups:/usr/bin/false
...
```

root 用户账户是 Linux 系统的管理员，固定分配给它的 UID 是 0 .上面显示的，Linux 系统会为各种各样的功能创建不同的用户账户，而这些账户并不是真的用户。这些账户叫做**系统账户**，是系统上运行的各种服务进程访问资源用的特殊账户。

Linux 为系统账户预留了 500 以下的UID 值有些服务甚至要用特定的 UID 才能正常工作，为普通用户创建账户时，大多数 Linux 系统会从 500 开始，将第一个可用 UID 分配给这个账户(并非所有的 Linux 发行版都是这样)

你可能已经注意到 `/etc/passwd` 文件中还有很多用户登陆名和 UID 之外的信息：

- 登陆用户名
- 用户密码
- 用户账户的 UID
- 用户账户的组 ID (GID) (数字形式)
- 用户账户的文本描述(备注字段)
- 用户 HOME 目录的位置
- 用户的默认 shell

`etc/passwd` 文件中的密码都被设置成了 `x`，为了安全考虑 Linux 开发人员重新考虑这个策略

**现在，绝大多数 Linux 系统都将用户密码保存在另一个单独的文件中(叫做 shadow 文件，位置在 `/etc/shadow`)**,只有特定的程序(比如登陆程序)才能访问这个文件

### /etc/shadow文件

`/etc/shadow` 文件对 Linux 系统密码管理提供了更多的控制.只有 root 用户才能访问 `/etc/shadow` 文件。文件中每条记录都有 9 个字段:

- 与 `/etc/passwd` 文件中的登录名字段对应的登录名
- 加密后的密码
- 自上次修改密码后过去的天数密码(1970.1.1开始)
- 多少天后才能更改密码
- 多少天后必须更改密码
- 密码过期前提前多少天提醒用户更改密码
- 密码过期后多少天禁用用户账户
- 用户账户被禁用的日期
- 预留字段给将来使用

### 添加新用户

用来向 Linux 系统添加新用户的主要工具是 **useradd** 这个命令简单快捷。可以一次创建新用户账户即设置用户 HOME 目录结构。useradd 命令使用系统的默认值以及命令行参数来设置用户账户。系统默认值被设置在`/etc/default/useradd` 文件中。可以加入 `-D` 选项的 useradd 命令查看所用 Linux 系统中的这些默认值

```shell
# /usr/sbin/useradd -D
GROUP=100  # 新用户被添加到 GID 为 100 的公共组
HOME=/home # 新用户的 HOME 目录将会位于 /home/loginname
INACTIVE=-1 #新用户账户密码在过期后不会被禁用
EXPIRE= #新用户账户未被设置过期日期
SHELL=/bin/bash #将 bash shell 作为默认 shell
SKEL=/etc/skel #系统会将 /etc/skel 目录下的内容复制到用户的 HOME 目录下
CREATE_MAIL_SPOOL=yes #该用户账户在 mail 目录下创建一个用于接受邮件的文件
```

/etc/skel 文件列表

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g94m0o4y29j30ss08stey.jpg)

它们是 bash shell 环境的标准启动文件。系统会自动将这些默认文件复制到你创建的每个用户的 HOME 目录

> Su 切换 用户，sudo 以管理员命令运行命令

**useradd**命令行参数

| 参数             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| -c comment       | 给新用户添加备注                                             |
| -d home_dir      | 为主目录指定一个名字(如果不想用登录名作为主目录名的话)       |
| -e expire_date   | 用 YYYY-MM-DD 格式指定一个账户过期的日期                     |
| -f inactive_days | 指定这个账户密码过期后多少天这个账户被禁用；<br />0 表示密码一过期就立即禁用<br />1 表示禁用这个功能 |
| -g initial_group | 指定用户登录组的 GID 或组名                                  |
| -G group ..      | 指定用户除登录组之外所属的一个或多个附加组                   |
| -k               | 必须和 -m 一起使用，将/etc/skel 目录的内容复制用户的 HOME 目录(默认添加了?) |
| **-m**           | **创建用户的 HOME 目录**                                     |
| -M               | 不设置用户的 HOME 目录(当默认设置里要求创建时才使用这个选项) |
| -n               | 创建一个与用户登录名同名的新组                               |
| -r               | 创建系统账户                                                 |
| -p passed        | 为用户账户指定默认密码                                       |
| -s shell         | 指定默认的登录 shell                                         |
| -u did           | 为账户指定唯一的 UID                                         |

使用命令行参数可以更改系统指定的默认值。在 **-D之后跟上一个指定的值来修改系统默认的新用户设置**

```shell
/usr/sbin/useradd -m 用户名
```

-m 命令会创建用户的 HOME 目录，并把 `/etc/skel` 目录中的文件复制了过来。

### 删除用户

如果想从系统中删除用户，userdel 可以满足这个需求。

- 默认情况下，userdel 命令会只删除`/etc/passwd`文件中的用户信息，而不会删除系统中属于该账户的任何文件。

- 如果加上 `-r`参数，userdel 会删除用户的 HOME 目录以及邮件目录。仍可能存有已删除用户的其他文件。

比如：`/user/sbin/userdel -r test` 删除test 用户



### 修改用户

Linux 提供了一些不同的工具来修改已有用户账户的信息

| 命令     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| usermod  | 修改用户账户的字段，还可以指定主要组以及附加组的所属关系     |
| passwd   | 修改已有用户的密码                                           |
| chpasswd | 从文件中读取登录名密码对，并更新密码。比如 username:password 存入txt 然后使用 chpsswd < xx.txt (未测试) |
| chage    | 修改密码的过期日期                                           |
| chfn     | 修改用户账户的备注信息                                       |
| chsh     | 修改用户账户的默认登录shell                                  |

#### usermod

usermod 命令是用户修改工具中最强大的一个。它能用来修改 `/etc/passwd`文件中的大部分字段，只需用与想修改的字段对应的命令行参数就行

- `-l` 修改用户账户的登录名
- `-L`锁定账户，使用户无法登录
- `-p` 修改账户的密码
- `-U` 接触锁定，使用户能够登录

```shell
# 实例
```

#### passwd 和 chpasswd



```mysql
# 改变用户密码简单命令
passwd test 

# 使用 chpasswd 为大量用户修改密码
chpasswd < users.txt
```

#### chsh、chfn 和 chage

修改特定的账户信息。chsh 命令用来快速修改默认的用户登录 shell 。使用时必须用 shell 的全路径名

```shell
chsh -s /bin/csh user
```



Chfn 命令提供备注字段存储信息的标准方法

```shell
chfn user
Changing finger information for testuser.
Name []: testuser txt 
Office []: no office
Office Phone []: 110
Home Phone []: 110
Finger information changed.

```



查看 `/etc/passwd`文件中的记录

```shell
grep testuser /etc/passwd
testuser:x:1002:1002:testuser txt,no office,110,110:/home/testuser:/bin/bash
```



chage 命令用来帮助管理用户账户的有效期。

| 参数 | 描述                               |
| ---- | ---------------------------------- |
| -d   | 设置上次修改密码到现在的天数       |
| -E   | 设置密码过期的日期                 |
| -I   | 设置密码过期到锁定账户的天数       |
| -m   | 设置修改密码之间最少要多少天       |
| -W   | 设置密码过期前多久开始出现提醒信息 |

日期值方式

- YYYY-MM-DD 格式的日期
- 代表从 1970.1.1 到该日期天数的数值

## 使用 Linux 组

为了解决共享资源，Linux 系统采用了另外一个安全概念——组

组权限运行多个用户对系统中的对象(比如**文件、目录或设备等**)共享一组共用的权限

> ⚠️注意：Linux 发行版在处理默认组的成员关系时略有差异。有些 Linux 发行版会创建一个组，把所有用户都当作这个组的成员。遇到这种情况要特别小心，因为文件hen

