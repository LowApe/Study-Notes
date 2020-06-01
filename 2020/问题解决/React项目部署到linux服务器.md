# React项目部署到 linux

```shell
# npm 也行 ,构建项目在当前生成 dist 目录
cnpm run build  
```

在 nginx 中的 `conf` 目录下修改`nginx.conf`配置文件

```properties
server {
		# 暴露的端口号，注意服务器也要打开这个端口
        listen       8000; 
        # 绑定的域名
        server_name  csgo.blogcode.top;
		# 路径映射
        location / {
        	# 绑定上 react 项目build 的目录
            root   /WEB/antd_pro/dist;
            index  index.html index.htm;
        }
 }

```

> 注意⚠️：因为没有备案，所以域名访问 80 端口，就无法进行，因为怎么配置都会显示对应服务商提示需要备案的信息，之前一直以为自己配置错误(😢坑了我三个小时！！)

因为我是下载安装包，所以使用下面命令启动，如果使用包管理，可能会简单些

```shell
# 结束 nginx
./nginx -s stop
# 重启 nginx
./nginx -s reload
# 开启 nginx
./nginx 
```

上面配置可以通过`csgo.blogcode.top:8000` 进行访问

# 相关链接

- [前端React项目部署到阿里云-linux 服务器](http://www.mamicode.com/info-detail-2950382.html)

