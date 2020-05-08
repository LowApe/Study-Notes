# ubuntu修改apt-get源为国内镜像源

## 原文件备份

```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak 
```

## 编辑源列表文件

```shell
sudo vim /etc/apt/sources.list 
```

## 将原来的列表删除，添加如下内容

阿里云源(2020年5月可用)

```shell
deb http://mirrors.aliyun.com/ubuntu/ xenial main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security universe
```

> 网易：http://mirrors.163.com/
> 搜狐：http://mirrors.sohu.com/
> 阿里云：http://mirrors.aliyun.com/
> 中国科技大学：https://mirrors.ustc.edu.cn/
> 清华大学：https://mirrors.tuna.tsinghua.edu.cn/

## 更新源

更新软件包列表

```shell
sudo apt-get update
```

 

休复损坏的软件包，尝试卸载出错的包，重新安装正确版本的(暂未尝试)

```shell
sudo apt-get -f install
```

 

升级系统中的所有软件包(暂未尝试)

```shell
sudo apt-get -y upgrade
```



