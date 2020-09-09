# Linux 进入目录太深

进入系统变量文件：

- ubuntu 是在路径 `~/.bashrc`
- mac 是在路径 `~/.bash_profile`

进入编辑文件,修改 else PS1中 `\w` 为`\W`这样长度名称就变为简略名称

```shell
...
if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\W\$ '
fi
...
# 最后激活这个变量文件
source ~/.bashrc
```

- `\u`是当前用户
- `\h`是主机名
- `\w`是显示完整路径
- `\W`是显示简单路径

# 相关链接

- [Linux终端提示符路径长度的修改方法](http://www.xitongzhijia.net/xtjc/20141216/32744.html)

