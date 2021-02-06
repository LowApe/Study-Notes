# 02 Kubernetes架构和部署

## 1 环境准备

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gmz0os0i91j30jq0imacj.jpg)

根据书中的要求创建四个 centos 7 的虚拟机，IP 以此静态设置。

软件版本

| 软件       | 版本  |
| ---------- | ----- |
| Kubernetes | 1.1.1 |
| Docker     | 1.7.1 |
| Etcd       | 2.2.2 |
| Flannel    | 9.5.1 |

 

## 2 运行 Etcd

> 因为2.2.2版本翻github太长了，决定装最新版 3.4.14

```shell
wget ${github Url}
tar -zxvf xxx.tar.gz
cd etcd-v3.4
$ cp etcd /usr/bin/etcd
$ cp etcdctl /usr/etcdctl

# 运行 Etcd
$  etcd -name etcd \ 
-data-dir /var/lib/etcd \
-listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001 \
-advertise-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001 \
>> /var/log/etcd.log 2>&1 &

# Etcd 运行后其健康状态
$ etcdctl endpoint health
127.0.0.1:2379 is healthy: successfully committed proposal: took = 2.293816ms

$ etcdctl endpoint health --endpoints=[127.0.0.1:2379]
{"level":"warn",
"ts":"2021-01-24T14:15:40.117Z",
"caller":"clientv3/retry_interceptor.go:62",
"msg":"retrying of unary invoker failed",
"target":"endpoint://client-e39f3159-d56e-460a-809c-964067ae5730/[127.0.0.1:2379]",
"attempt":0,
"error":"rpc error: code = DeadlineExceeded desc = latest balancer error: all SubConns are in TransientFailure, latest connection error: connection error: desc = \"transport: Error while dialing dial tcp: address [127.0.0.1:2379]: missing port in address\""}
[127.0.0.1:2379] is unhealthy: failed to commit proposal: context deadline exceeded
Error: unhealthy cluster
```

> 暂时停止，因为健康状态有些问题

## 3 获取 Kubernetes 发布包

从[GitHub 下载 当前时间最新V1.20.2](https://github.com/kubernetes/kubernetes/releases/tag/v1.20.2),

```shell
$ tar zxvf kubernetes.tar.gz
```



# 验证

```shell
$ kubectl version
$ kubectl config current-context
$ kubectl cluster-info
$ kubectl get nodes
```

