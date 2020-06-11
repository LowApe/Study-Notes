# 阿里云无法启动mysql

问题：服务器一段时间mysql 锁住？

猜测：难道是以前ssh 连接没有正常关闭 mysql ？



错误信息：

```shell
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)
```



查看状态

```
systemctl status mysql.service
● mysql.service - MySQL Community Server
   Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
   Active: activating (start-post) (Result: exit-code) since Thu 2020-06-11 11:04:17 CST; 21s ago
  Process: 32643 ExecStart=/usr/sbin/mysqld (code=exited, status=203/EXEC)
  Process: 32635 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
 Main PID: 32643 (code=exited, status=203/EXEC);         : 32644 (mysql-systemd-s)
    Tasks: 2
   Memory: 312.0K
      CPU: 111ms
   CGroup: /system.slice/mysql.service
           └─control
             ├─32644 /bin/bash /usr/share/mysql/mysql-systemd-start post
             └─32696 sleep 1

Jun 11 11:04:17 iZuf6aytwpcvzkwiisgaxaZ systemd[1]: mysql.service: Service hold-off time over, scheduling restart.
Jun 11 11:04:17 iZuf6aytwpcvzkwiisgaxaZ systemd[1]: Stopped MySQL Community Server.
Jun 11 11:04:17 iZuf6aytwpcvzkwiisgaxaZ systemd[1]: Starting MySQL Community Server...
Jun 11 11:04:17 iZuf6aytwpcvzkwiisgaxaZ systemd[32643]: mysql.service: Failed at step EXEC spawning /usr/sbin/mysqld: Exec format error
Jun 11 11:04:17 iZuf6aytwpcvzkwiisgaxaZ systemd[1]: mysql.service: Main process exited, code=exited, status=203/EXEC
```

```shell
systemctl restart mysql.service
# 重启提示
Job for mysql.service failed because the control process exited with error code. See "systemctl status mysql.service" and "journalctl -xe" for details.
# 使用这两个命令查看问题
systemctl status mysql.service
journalctl -xe
```

其中一条错误

```shell
mysql.service: Failed at step EXEC spawning /usr/sbin/mysqld: Exec format error
```

## 最后实在找不到问题，重新安装

```shell
# 删除软件和配置
sudo apt-get purge mysql.server
# 重新下载
```

