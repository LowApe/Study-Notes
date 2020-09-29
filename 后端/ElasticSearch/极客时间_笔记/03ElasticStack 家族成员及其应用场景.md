# ElasticStack 家族成员及其应用



## Elastic Stack 生态圈

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g8qho22k6qj30uk0fogq2.jpg)

### Logstash:数据处理管道

- 开源的服务器**数据处理管道**，支持从不同来源采集数据，转换数据，并将数据发送到不同的存储库中
- Logstash 诞生于 2009 年，最初用来做日志的采集与处理
- Logstash 创始人 Jordan Sisel
- 2013 年被 Elasticsearch 收购 

### Logstash 特性

- 实时解析和转换数据
	- 从 IP 地址破译出地理坐标
	- 将 PII 数据匿名化，完全排除敏感字段
- 可扩展
	- 200 多个插件 (日志/数据库/Acsigh/Netflow)
- 可靠性安全性
	- Logstash 会通过**持久化队列**来保证至少将运行的事件送达一次
	- 数据传输加密
- 监控



### Kibana：可视化分析利器

- Kibana = Kiwifruit + Banana
- 数据可视化工具，帮助用户解开对数据的任何疑问
- 基于 Logstash 的工具，2013 年加入 Elastic 公司



## Elastic 的发展

- 2015 年 3 月收购 Elastic Cloud，提供 Cloud 服务
- 2015 年 3 月收购 PacketBeat
- 2016 年 9 月 收购 PreAlert - Machine Learning 异常检测
- 2017 年 6 月 收购 Opbeat 进军 APM
- 2017 年 11 月 收购 SaaS 厂商 Swiftype，提供网站和 App 搜索
- 2018 年 X-Pack 开源

### BEATS - 轻量的数据采集器



### X-Pack：商业化套件

-  6.3 之前，X-Pack 以插件方式安装
- X-Pack 开源之后，ElasticSearch & Kibana 支持 OSS 版和 Basic 两种版本
	- 部分 X-Pack 功能支持免费使用，6.8 和 7.1 开始，Security 功能免费
- OSS，Basic，黄金级，白金级



## ELK 客户及应用场景

- 应用场景
	- 网站搜索/垂直搜索/代码搜索
	- 日志管理与分析/安全指标监控/应用性能监控/WEB 抓取舆情分

### 日志的重要性

- 为什么重要
	- 运维：医生给病人看病。日志就是病人对自己的描述
	- 恶意攻击，恶意注册，刷单，恶意密码猜测
- 挑战
	- 关注点很多，任何一个点都有可能引起问题
	- 日志分散在跟多机器，出了问题时，才发现日志被删了
	- 很多运维人员时消防员，哪里有问题去哪里

### 日志管理

日志搜索 -> 格式化分析 -> 全文检索 -> 风险告警



## ElasticSearch 与数据库的集成

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g8qisxiverj30ka0aqdhn.jpg)

- 单独使用 ElasticSearch 存储
- 以下情况可考虑与数据库集成
	- 与现有系统的集成
	- 需考虑**事务性**
	- 数据更新频繁

## 指标分析 / 日志分析

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g8qj5ekir0j30ui0f0afy.jpg)

- Beats 进行数据的收集
- 数据量比较大的情况下，通过 redis 和 kafka 或者 RabbitMQ 进行数据缓冲
- 然后通过 logstash 进行数据的转化和集成发送到 es
- kibana 或者 grafana 进行数据可视化和数据分析