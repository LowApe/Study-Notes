# 命令行 Bash 脚本

Bash（或shell）脚本是自动执行重复任务的好方法，可以为开发人员节省大量时间。Bash脚本在Bash shell解释器终端内执行。您可以在终端中运行的任何命令都可以在Bash脚本中运行。如果您有经常使用的命令或命令集，请考虑编写Bash脚本来执行它。

要遵循一些约定，以确保您的计算机能够找到并执行您的Bash脚本。
- 脚本文件的开头应该从`#!/bin/bash`它自己的行开始。这告诉计算机将哪种解释器用于脚本。保存脚本文件时，最好将常用脚本放在`~/bin/`目录中。

- 脚本文件还需要具有“执行” 权限才能运行它们。要将此权限添加到具有filename的文件：`script.shuse`：

```shell
$ chmod +x script.sh
```

您的终端每次打开时都会运行一个文件来加载其配置。在Linux风格的shell上，这是`~/.bashrc`
在OSX上，这是`~/.bash_profile`。要确保脚本~/bin/可用，您必须将此目录添加到PATH配置文件中：

- `PATH=~/bin:$PATH`

现在`~/bin`，通过键入文件名，可以从任何地方运行目录中的任何脚本。

## 变量(Variables)

在bash脚本（或该事件的终端）中，通过将变量名称设置为等于另一个值来声明变量。例如，要将变量设置greeting为“Hello”，可以使用以下语法：

```shell
$ greeting="Hello"
```

请注意，变量名称，等号或“Hello”之间没有空格。

要访问变量的值，我们使用前缀为美元符号（$）的变量名称。在前面的示例中，如果我们要将变量值打印到屏幕，我们使用以下语法：

```shell
$ echo $greeting
```

```sh
#!/bin/bash
phrase="Hello to you!"
echo $phrase
```

## 条件语句(Conditionals)

在使用bash脚本时，您可以使用条件来控制脚本中的哪组命令运行。使用`if`启动条件，然后在方括号中的条件（`[ ]`）。`then`开始满足条件时将运行的代码。`else`开始在不满足条件时将运行的代码。最后，有条件的用向后封闭`if`，`fi`。

bash脚本中的完整条件使用以下语法：

```.sh
if [ $index -lt 5 ]
then
  echo $index
else
  echo 5
fi
```
> **一定要注意if后面有`空格space`**

Bash脚本使用特定的运算符列表进行比较。这里我们使用的`-lt`是“小于”。此条件的结果是，如果`$index`小于5，它将打印到屏幕。如果为5或更大，则将在屏幕上打印“5”。

以下是您可以在bash脚本中使用的数字的比较运算符列表：

- 等于： -eq
- 不相等： -ne
- 小于或等于： -le
- 少于： -lt
- 大于或等于： -ge
- 比...更棒： -gt
- 一片空白： -z

比较字符串时，最好将变量放入quotes（`"`）。如果变量为null或包含空格，则可以防止错误。比较字符串的常用运算符是：

- 等于： ==
- 不相等： !=

例如，如果变量比较foo，并bar含有相同的字符串：

```.sh
 if [ "$foo" == "$bar"]
```

## 循环(loops)
有3点不同的方式来循环bash脚本中：`for`，`while`和`until`。
for循环用于遍历列表并在每个步骤执行操作。例如，如果我们有一个存储在变量中的单词列表paragraph，我们可以使用以下语法来打印每个单词：

```.sh
for word in $paragraph
do
  echo $word
done
```
请注意，`word`在`for`循环的顶部被“定义”，因此没有$前置。请记住，我们$在访问变量值时会有前缀。因此，当访问do块中的变量时，我们$word照常使用。

在bash脚本中`until`并且`while`非常相似。while循环保持循环，而提供的条件为真，而until循环循环直到条件为真。条件的建立方式与if方块内方块之间的条件相同。如果我们想要打印index变量，只要它小于5，我们将使用以下while循环：

```.sh
while [ $index -lt 5 ]
do
  echo $index
  index=$((index + 1))
done
```
请注意，bash脚本编写中的算术使用`$((...))`语法，并且在括号内，变量名称不会添加前缀$。

同样的循环也可以写成`until`循环，如下所示：

```.sh
until [ $index -eq 5 ]
do
  echo $index
  index=$((index + 1))
done
```

## 输入(inputs)

为了使bash脚本更有用，我们需要能够访问bash脚本文件本身外部的数据。第一种方法是通过提示用户输入。为此，我们使用read语法。要询问用户输入并将其保存到number变量，我们将使用以下代码：
```.sh
echo "Guess a number"
read number
echo "You guessed $number"
```
访问外部数据的另一种方法是让用户在运行脚本时添加输入参数。这些参数在脚本名称后输入，并以空格分隔。例如：

```.sh
saycolors red green blue
```
在脚本中，使用$1，$2等访问它们，其中$1第一个参数（此处为“红色”）等等。请注意，这些是1索引。

如果您的脚本需要接受无限数量的输入参数，则可以使用"$@"语法对它们进行迭代。对于我们的saycolors示例，我们可以使用以下方

```.sh
for color in "$@"
do
  echo $color
done
```
最后，我们可以访问我们脚本的外部文件。您可以使用正则表达式使用标准bash模式匹配将一组文件分配给变量名称。例如，要获取目录中的所有文件，可以使用以下*字符：

```.sh
files=/some/directory/*
```
然后，您可以遍历每个文件并执行某些操作。在这里，让我们打印完整的路径和文件名：

```.sh

for file in $files
do
  echo $file
done
```

## 别名(Aliases)

您可以在您`.bashrc`或`.bash_profile`文件中为bash脚本设置别名，以允许在没有完整文件名的情况下调用脚本。例如，如果我们有saycolors.sh脚本，我们可以saycolors使用以下语法将其别名为单词：
```shell
alias saycolors='./saycolors.sh'
```
您甚至可以将标准输入参数添加到别名中。例如，如果我们总是希望将“green”作为第一个输入包含在内saycolors，我们可以将别名修改为：

```shell
alias saycolors='./saycolors.sh "green"'
```

# 总结

花一点时间来回顾一下你对bash脚本的了解。

- 可以在终端中运行的任何命令都可以在bash脚本中运行。
- 使用没有空格（`greeting="hello"`）的等号分配变量。
- 使用美元符号（`echo $greeting`）访问变量。
- 条件语句使用`if`，`then`，`else`，`fi`语法。
- 三种类型的循环可以使用：for，while，和until。
- Bash脚本使用一组独特的比较运算符：
    - 等于： `-eq`
    - 不相等： `-ne`
    - 小于或等于： `-le`
    - 少于： `-lt`
    - 大于或等于： `-ge`
    - 比...更棒： `-gt`
    - 一片空白： `-z`
- 输入参数可以在脚本名称后传递给bash脚本，用空格分隔（myScript.sh“hello”“你好吗”）。
- 可以使用`read`关键字从脚本用户请求输入。
- 别名可以在`.bashrc`或`.bash_profile`使用`alias`关键字创建。
