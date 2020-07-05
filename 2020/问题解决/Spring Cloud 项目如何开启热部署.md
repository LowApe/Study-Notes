# Spring Cloud 项目如何开启热部署

## 使用的 devtools

添加依赖

```properties
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
```

> mac 快捷键`Command`+`fn`+`F9`  其实就是编译我们修改的类，下面相关连接，有使用设置idea 可以自动编译，之前设置过，但不知道为啥在 Spring Cloud 项目貌似都不生效。总的来说 devtools 还是通过编译修改过的文件实现项目**重启**这种方式其实跟手动重启所用的时间一样。

## 使用 JRebel

1. 打开 IDEA `Command`+`,` settings 的 `Plugins`
2. 搜索 `JRebel` 安装重启

因为我是 mac 需要安装 nignx 反向代理去激活才能使用

```shell
brew install nginx
sudo niginx
vim /usr/local/etc/nginx/nginx.conf
server {
        listen       8080;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            #root   html;
            #index  index.html index.htm;
            proxy_pass http://idea.lanyus.com:80;
        }
}
# 重启 
sudo nignx -s reload 
```

- 生成一个 [guid code](https://www.guidgen.com/)



打开 `Plugins` 选择

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gggghpodtxj3270140wms.jpg)

![image-20200705221949423](/Users/mac/Library/Application Support/typora-user-images/image-20200705221949423.png)



- 激活成功

### 配置一下

JRebel 会在平时运行状态栏上出现两个标识，其次文件右键也绑定了，点击启动：

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1ggggs80yzvj32640qin4b.jpg)



目前修改后，刷新两次页面，就ok 相对手动 `Build `还是比较快的，可能按照教程应该可以调节响应速度。先留两个官网文档吧，后期有空了看看能不能缩短响应时间，或者设置触发条件啥的，比如保存。

- [JRebel Document](https://manuals.jrebel.com/jrebel/)
- [XRebel](https://manuals.jrebel.com/xrebel/)

# 相关连接

- [SpringBoot SpringCloud 热部署 热加载 热调试](https://www.cnblogs.com/crazymakercircle/p/12077373.html)
- [Mac idea激活jrebel](https://www.jianshu.com/p/e744565398ed)
- [热部署神器-JRebel的简单使用](https://www.cnblogs.com/pangyangqi/p/10208791.html)
- 



