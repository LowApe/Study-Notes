# 使用Linux环境变量
本章内容
- 什么是环境变量
- 创建自己的局部变量
- 删除环境变量
- 默认 shell 环境变量
- 设置 PATH 环境变量
- 定位环境文件
- 数组变量

Linux环境变量能帮你提升 Linux shell 体验。很多程序和脚本都通过**环境变量**来获取**系统信息**、**存储临时数据**和**配置信息**

# 什么是环境变量
bash shell 用一个叫作环境变量(environment variable)的特性来**存储有关shell会话**和**工作环境的信息**。环境变量运行你在**内存中**存储数据，以便程序或**shell中运行的脚本**能够轻松**访问**到它们。这也是存储持久数据的一种简便方法

环境变量分为两种
- 全局变量
- 局部变量

## 全局环境变量
全局环境变量对shell会话和所有生成的子 shell 都是可见的。局部变量则只对创建它们的shell可见
> Linux 系统在你开始 bash 会话时就设置了一些全局环境变量<br>
系统环境变量基本上都是**全大写字母**，以区别于普通用户的环境变量

```shell
$ printenv HOME //查看全局变量、显示个别环境变量的值
$ env

$ echo $HOME //也可以使用 echo 显示变量的值，引用某个环境变量需要添加美元符
```
> 美元符也能够让变量作为**命令行参数**

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7oawdspjdj30n408cgqr.jpg)

全局环境变量可用于进程的所有子 shell--在shell 中使用`bash` 命令进入子shell，接着使用 `ls $HOME` 显示相同的内容

## 局部环境变量
局部环境变量只能在定义它们的**进程**中可见。在Linux系统也默认定义了标准的局部环境变量。不过你也可以定义自己的局部变量

在Linux系统并没有一个只显示局部环境变量的命令。**set命令会显示特定进程设置的所有环境变量的命令**，包括局部变量、全局变量以及用户自定义变量

# 设置用户定义变量

## 设置局部用户定义变量
可以通过**等号**给环境变量赋值，值可以是数值或字符串

```shell
$ echo $my_variable
$ my_variable = Hello
$ echo $my_variable
HELLO
```
如果要给变量赋一个**含有空格**的字符串值，必须用单引号来界定字符串的首和尾

```shell
$ my_variable='Hello World'
```
> ⚠️注意：没有单引号，bash shell会以为下一个词是另一个要执行的命令。<br>你定义的局部环境变量用的是**小写字母**<br>变量名、等
号和值之间没有空格,这一点非常重要。

## 设置全局环境变量
创建全局环境变量的方法是先创建一个**局部环境变量**，然后再把它**导出**到全局环境中，这个过程通过 export 命令来完成，变量名前面不需要加`$`

```shell
$ my_variable='I am Global now'
$ export my_variable
$ bash
$ my_variable='NULL'
$ echo my_variable
NULL
$ exit
$ echo my_variable
I am Global now
```
修改子shell中全局环境变量并不会影响到父shell中该变量的值

> 甚至子shell甚至无法使用export命令改变父shell中全局环境变量的值

# 删除环境变量
可以用`unset`命令完成这个操作。

```shell
$ echo $my_variable
I am Global now
$ unset my_variable
```
> 如果要**用**变量,使用`$`，如果**操作**变量，不使用`$`,这条规则的一个例外就是使用 `printenv`某个变量的值

在处理全局环境变量时，如果你是在**子进程**中**删除**一个全局环境变量，这只**对子进程有效**,该全局环境变量在父进程中依然可用。

# 默认的shell环境变量
默认情况下，bash shell 会用一些特定的环境变量来定义系统环境。这些变量在你的Linux系统上都已经设置好了


bash shell 支持的 Bourne 变量

| 变量 | 描述     |
| :------------- | :------------- |
| CDPATH       | 冒号分隔的目录列表作为 cd 命令的搜索路径       |
|HOME|当前用户的主目录|
|IFS|shell 用来将文本字符串分隔成字段的一系列字符|
|PATH|shell 查找命令的目录列表，由冒号分隔|
|...|..|

bash shell 环境变量

| 变量 | 描述  |
| :------------- | :------------- |
| BASH       |当前shell 实例的全路径名     |
|BASH_ALIASES|含有当前已设置别名的关联数组|
|BASH_ARGC|含有传入子函数或shell脚本的参数**总数的**数组变量|
|BASH_ARCV|含有传入子函数或shell脚本的参数总数的数组变量|
|...|...|

不是所有的默认环境变量都会在运行 set 命令时列出。尽管这些都是默认环境变量，但并不是每一个都必须又一个值

# 设置 PATH 环境变量
当你在 shell 命令行界面中输入一个外部命令时时，shell必须搜索系统来找到对应的程序。
PATH环境变量**定义了用于进行命令和程序查找的目录**。

PATH中的目录使用冒号分隔，如果命令或者程序的位置没有包括在PATH变量中，那么如果不使用绝对路径的话，shell是没法找到的。如果shell找不到指定的命令或程序，他会产生一饿错误信息：

```shell
$ myprog
-bash:myprog:command not found
```
可以把新的搜索目录添加到现有的PATH环境便来那个中国，无需从头定义PATH中各个目录之间是用冒号分隔的。只需引用原来的PATH值，然后再给这个字符串添加新目录。

```shell
$ echo $PATH
$ PATH=$PATH:/xxxxx
$
```

> ⚠️注意：如果希望子 shell 也能找到你的程序的位置，一定要记得把修改后的PATH环境变量导出。对PATH变量的修改只能持续到退出或重启系统。这种效果并不能一直持续。

# 定位系统环境变量

接下来怎么让环境变量变**持久化**。当你登入Linux系统启动一个 bash shell 时，默认情况下 bash 会在几个**文件中查找命令**。这些文件叫做启动文件或环境文件。bash 检查的启动取决于你启动 bash shell 的方式。启动bash shell 有3种方式：

- 登陆时作为默认登陆shell
- 作为非登陆shell的交互shell
- 作为运行脚本的非交互shell

## 登陆shell

当你登陆Linux系统时，bash shell 会作为登陆 shell 启动。登陆 shell 会从 5 个不同的启动文件里读取命令：

- etc/profile
- $HOME/.bash_profile
- $HOME/.bashrc 
- $HOME/.bash_login
- $HOME/.profile

`/etc/profile`文件时系统默认的 bash shell 的主启动文件。系统上的每个用户登陆时都会执行这个启动文件

另外4个启动文件是针对用户的，可根据个人需求定制。我们来仔细看一下各个文件。

### 1./etc/profile 文件

`/etc/profile `文件是 bash shell 默认的主启动文件。只要你登录了 Linux 系统，bash 就会执行`/etc/profile`启动文件中的命令。不同的Linux发行版在这个文件存放不同的命令

### 2.$HOME目录下的启动文件

剩下的启动文件都起着同一个作用：提供一个用户**专属的启动文件**来定义该用户所用到的环境变量。大多数发行版只用到这四个启动文件中的一到两个：

- $HOME/.bash_profile
- $HOME/.bashrc 
- $HOME/.bash_login
- $HOME/.profile

> ⚠️注意：这四个文件都以点号开头，说明它们是隐藏文件。它们位于用户的HOME目录下，所以每个用户都可以编辑这些文件并添加自己的环境变量，这些环境变量会在每次启动bash shell会话时生效

> 💡 提示：$HOME 表示的是某个用户的主目录。它和波浪号(~)的作用一样

## 交互式shell进程

如果你的bash shell 不是登陆系统时启动的(比如是在命令行提示符下敲入 bash 时启动)，那么你启动的 shell 叫做交互式shell。交互式shell不会像登陆 shell 一样运行，但它依然提供了命令行提示符来输入命令。 

> 如果bash 是作为交互式shell启动的，它就不会访问`/etc/profile`文件，只会检查用户HOME目录中的`.bashrc`文件

`.bashrc`文件有两个作用

- 查看`/etc`目录下通用的 bashrc 文件
- 为用户提供一个定制自己的命令别名和私有脚本函数的地方

## 非交互式shell

系统执行**shell脚本**时用的就是这种shell。不同的地方在于它没有**命令行提示符**。但是当你在系统上运行脚本时，也许希望能够运行一些特定启动命令

为了处理这种情况，bash shell 提供了 BASH_ENV 环境变量。当 shell 启动一个非交互式shell进程时，它会**检查这个环境变量**来查看要**执行的启动文件**。如果有指定的文件，shell会执行该文件里的命令，这通常包括 shell 脚本变量设置。

那如果BASH_ENV变量没有设置，shell脚本到哪里去获得它们的环境变量？**子shell可以继承父shell导出过的变量**。举例来说，如果父shell是登陆shell，在`/etc/profile`等文件中**设置并导出了变量**，用于执行脚本的**子shell**就能够继承这些变量

## 环境变量持久化

对于全局环境变量来说，可能更倾向于将新的或修改过的变量设置放在`/etc/profile`文件中，但这不是什么好主意。**如果升级了所用的发行版，这个文件也会跟着更新，那你所有定制过的变量设置可就都没有了。**

> 建议存储到 `$HOME/.bashrc`文件

# 数组变量

环境变量可以作为数组使用。数组是能够存储多个值的变量。这些值的变量。这些值可以单独引用，也可以作为整个数组来引用

要给某个环境变量设置多个值，可以把值放在括号里，值与值之间用空格分隔

```shell
mytest=(one two three four five)
echo mytest
one
```

只有数组第一个值显示出来了。需要引用单独的数组元素，就必须用代表它在数组中位置的**数值索引值**

```shell
echo ${mytest[2]}
three
```

要显示整个数组变量，可用星号作为通配符放在索引值的位置

```shell
echo ${mytest[*]}
one two three four five
```

也可以改变某个索引值位置的值

```shell
mytest[2]=seven
echo ${mytest[*]}
one two sevene four five
```

> ⚠️注意：通过`unset`命令删除数组中的某个值，可能有些复杂

```shell
unset mytest[2]

echo ${mytest[*]}
one two four five

echo ${mytest[2]}

echo ${mytest[3]}
four
```



# 小结

- 全局环境变量可以再对其作出定义的父进程所创建的子进程使用。局部环境变量只能在定义它们的进程中使用
- 系统通过全局和局部变量存储系统信息
- PATH环境变量定义bash shell 在查找可执行命令时的搜索目录
- bash shell 会在启动时执行几个启动文件
- 环境变量数组