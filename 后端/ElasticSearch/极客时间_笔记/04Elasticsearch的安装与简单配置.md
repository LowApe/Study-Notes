# Elasticsearch的安装与简单配置

## 本地部署 & 水平扩展

- 开发环境部署
- 单节点，一个节点承担多种角色
- 单机部署多个节点，便于学习了解分布式集群的工作机制



## 安装 Java

- 运行 Elasticsearch，需安装并配置 JDK
	- 设置 $JAVA_HOME
- 各个版本对 Java 的依赖
	- Elasticsearch 需要 Java 8 以上的版本
	- Elasticsearch 从 6.5 开始支持 Java 11
	- https://www.elastic.co/support/matrix#matrix_jvm
	- 7.0 开始，内置了 Java 环境

## 获取 Elasticsearch 安装包

- 下载二进制文件
- 支持 Docker 本地运行
- Helm chart for kubernetes
- Puppet Module

## 安装并运行 Elasticsearch

1. 下载并解压 Elasticsearch
2. 运行`bin/elasticsearch`（或`bin\elasticsearch.bat`在 Windows 上）
3. 运行 `curl http://localhost:9200/`或 `Invoke-RestMethod  http://localhost:9200` 使用 PowerShell
4. 深入了解入门指南和视频

## Elasticsearch 的文件目录结构

| 目录    | 配置文件          | 描述                                                      |
| ------- | ----------------- | --------------------------------------------------------- |
| bin     |                   | 脚本文件，包括启动 elasticsearch,安装插件。运行统计数据等 |
| config  | elasticsearch.yml | 集群配置文件，user，role based 相关配置                   |
| JDK     |                   | Java 运行环境                                             |
| data    | path.data         | 数据文件                                                  |
| lib     |                   | Java 类库                                                 |
| logs    | path.log          | 日志文件                                                  |
| modules |                   | 包含所有 ES 模块                                          |
| plugins |                   | 包含所有已安装插件                                        |

> ⚠️注意：mac下brew安装的目录路径
>
> ```
> elasticsearch:  /usr/local/Cellar/elasticsearch/x.x.0
> (待定)Data:    /usr/local/var/elasticsearch/elasticsearch_xuchen/
> Logs:    /usr/local/var/log/elasticsearch/elasticsearch_xuchen.log
> Plugins: /usr/local/Cellar/elasticsearch/x.x.0/libexec/plugins/
> Config:  /usr/local/Cellar/elasticsearch/x.x.0/libexec/config
> plugin script: /usr/local/opt/elasticsearch/libexec/bin/elasticsearch-plugin
> ```
>
> 

## JVM配置

- 修改 JVM - config/jvm.options
	- 7.1 下载的默认设置是 1 GB
- 配置的建议
	- Xms 和 Xms 设置成一样
	- Xmx 不要超过机器内存的 50%
	- 不要超过 30 GB -[https://www.elastic.co/blog/a-heap-of-trouble](https://www.elastic.co/cn/blog/a-heap-of-trouble)

## 安装与查看插件

Mac 下的安装方式

```shell
elasticsearch-7.0.1 bin/elasticsearch-plugin install analysis-icu
...
elasticsearch-7.0.1 bin/elasticsearch-plugin list
analysis-icu

```

- Elasticsearch 提供插件的机制对系统进行扩展
	- Discovery Plugin
	- Analysis Plugin 分析
	- Security Plugin 安全
	- Management Plugin
	- Ingest Plugin
	- Mapper Plugin
	- Backup Plugin

## 如何在开发机上运行多个 Elasticsearch 实例

```shell
bin/elasticsearch -E node.name=geektime2 -E path.data=node2_data -d
bin/elasticsearch -E node.name=geektime1 -E path.data=node1_data -d
bin/elasticsearch -E node.name=geektime3 -E path.data=node3_data -d
```

- 删除进程 `ps grep | elasticsearch / kill pid`

