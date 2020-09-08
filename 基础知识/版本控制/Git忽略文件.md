#  Git 忽略文件

下面的 Java 的忽略文件:被写入次文本的文件不受 git 版本管理

```shell
# 在版本库下创建忽略文件
touch .gitignore
vi .gitignore

# 内容
# Compiled class file
*.class

# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*

# idea file
target
.idea
JavaLearn.iml

```



# 相关链接

- [GitHub官网忽略文件](https://github.com/github/gitignore)

- [忽略特殊文件](https://www.liaoxuefeng.com/wiki/896043488029600/900004590234208)

