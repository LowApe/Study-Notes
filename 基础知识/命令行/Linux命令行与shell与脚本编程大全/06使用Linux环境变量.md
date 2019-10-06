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
