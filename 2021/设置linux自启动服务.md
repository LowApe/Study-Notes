# 设置Linux自启动服务

# CentOS 设置自启动

```shell
# 编辑
vim /etc/rc.local

# 添加需要启动的命令
systemctl start xxx

# 激活
chmod +x /etc/rc.d/rc.local

# 重启
reboot
```

