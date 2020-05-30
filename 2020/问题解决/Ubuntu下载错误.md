# Ubunut下载错误

当使用`apt-get install xxx` 的命令会出现下面的错误:

```shell
# sudo apt-get install git
E: Could not get lock /var/lib/dpkg/lock-frontend - open (11: Resource temporarily unavailable)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?
```

> 为什么会出现？
>
> 因为 apt 还在运行。



## 解决：

1. 结束掉apt 相关进程`ps -A | grep apt`  kill
2. 删除`/var/lib/dpkg/lock-frontend`和 `/var/lib/dpkg/lock`锁定文件

继续执行下载命令出现

```shell
# apt-get install subversion
E: dpkg was interrupted, you must manually run 'dpkg --configure -a' to correct the problem.
# dpkg --configure -a 
```

根据提示执行,解决。

