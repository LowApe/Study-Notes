# git clone 问题

> 问题描述：在处于网络封闭状态下，想通过ssh 进行下载，发现无法连接，针对 mac 进行搜索相关问题并解决

[帖子地址](https://gist.github.com/laispace/666dd7b27e9116faece6)

结合以上各位经验设置成功. 以下以macOS为准.

1. https访问
	仅为github.com设置socks5代理(推荐这种方式, 公司内网就不用设代理了, 多此一举):
	`git config --global http.https://github.com.proxy socks5://127.0.0.1:1086`
	其中1086是socks5的监听端口, 这个可以配置的, 每个人不同, 在macOS上一般为1086.
	设置完成后, ~/.gitconfig文件中会增加以下条目:

	```
	[http "https://github.com"]
	    proxy = socks5://127.0.0.1:1086
	```

2. ssh访问
	需要修改~/.ssh/config文件, 没有的话新建一个. 同样仅为github.com设置代理:

	```
	Host github.com
	    User git
	    ProxyCommand nc -v -x 127.0.0.1:1086 %h %p
	```

	如果是在Windows下, 则需要个性%home%.ssh\config, 其中内容类似于:

	```
	Host github.com
	    User git
	    ProxyCommand connect -S 127.0.0.1:1086 %h %p
	```

	这里-S表示使用socks5代理, 如果是http代理则为-H. connect工具git自带, 在<Git>\mingw64\bin\下面.