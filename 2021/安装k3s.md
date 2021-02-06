# 安装k3s

[k3s官方](https://www.rancher.cn/k3s/)

```shell
# 下载 k3s 并运行安装 
$ curl -sfL https://get.k3s.io | sh -
# 启动(默认自动启动)
$ sudo k3s server &
# 复制 msater token 到节点
$ /var/lib/rancher/k3s/server/node-token
```

# 安装 Rancher

```shell

$ sudo docker run --privileged -d --restart=unless-stopped -p 80:80 -p 443:443 -v <主机路径IP>:/var/lib/rancher/ rancher/rancher:stable
```



