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



# 打包部署后后端请求失效

因为前端开发使用了代理，但打包后，无法代理，如果使用 nginx，则需要下面配置

```properties
server {
        listen       80;
        server_name  csgo.blogcode.top;
        location / {
            root   /root/project/CSGO/WEB/antd_pro/dist;
            index index.html index.htm;
            #proxy_set_header X-Real-IP $remote_addr;
            #proxy_set_header Host $http_host;
            #proxy_pass http://106.15.202.130;
        }
		location /my/api {
            proxy_pass http://106.15.202.130:8089/page;
            proxy_set_header   X-Forwarded-Proto $scheme;
            proxy_set_header   X-Real-IP         $remote_addr;
 		}
}
```

# 跳转页面失效

因为根据 react-router 跳转需要设置 ngixn

```shell
server {
  ...
  location / {
    try_files $uri /index.html
  }
}
```



# 页面不显示

因为umi 打包生成 dist 静态文件还是存在问题，虽然本地开发环境没有出现问题，但是部署上去缺出现为止错误，因为编译过后是有混淆的，但是通过混淆前后的部分代码，从源码中搜索相关问题，并进行注释重编译后解决

# 相关链接

- [前端React项目部署到阿里云-linux 服务器](http://www.mamicode.com/info-detail-2950382.html)
- [React Router browserHistory](https://react-guide.github.io/react-router-cn/docs/guides/basics/Histories.html)

