# 安装cnpm问题

Ubuntu 下命令:

```shell
apt-get install npm 
npm install cnpm -g # npm 国内网络问题
npm install cnpm -g --registry=https://r.npm.taobao.org # 使用淘宝镜像
```

# nodejs 版本太旧问题

```shell
WARN engine cnpm@6.1.1: wanted: {"node":">= 6.0.0"} (current: {"node":"4.2.6","npm":"3.5.2"})
npm install -g n # 安装 n 版本控制模块
n stable # 下载稳定版本
node -v
```

查看并为修改版本，尝试修改 node 环境变量刚才`n stable` 命令安装的node在`/usr/local/n/versions/node/12.17.0`（此时下载最新版本是 12.17.0）

```shell
# 当前 node 路径
which node 
# 添加 node 环境变量
vim /etc/profile
```

编辑并保存内容如下：

```properties
export NODE_HOME=/usr/local/n/versions/node/12.17.0
export PATH=$JAVA_HOME/bin:$MYSQL_HOME/bin:$MAVEN_HOME/bin:$NODE_HOME/bin:$PATH
```

`source /etc/profile` 激活配置 `node -v` 查看，版本最新的话，继续`npm install cnpm -g --registry=https://r.npm.taobao.org`命令

# 参考链接

- [如何正确的升级node版本(已解决)](https://www.cnblogs.com/chaoyangya/p/10484513.html)