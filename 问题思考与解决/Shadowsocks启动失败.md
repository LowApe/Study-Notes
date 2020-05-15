# Shadowsocks 启动失败

[安装的步骤](https://morning.work/page/2015-12/install-shadowsocks-on-centos-7.html)

> 出现错误

```shell
root@iZuf6aytwpcvzkwiisgaxaZ:/etc/systemd/system# systemctl status shadowsocks.service
● shadowsocks.service - Shadowsocks
   Loaded: loaded (/etc/systemd/system/shadowsocks.service; enabled; vendor preset: enabled)
   Active: failed (Result: exit-code) since Sat 2020-02-15 13:37:24 CST; 1s ago
  Process: 1108 ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json (code=exited, status=203/EXEC)
 Main PID: 1108 (code=exited, status=203/EXEC)

Feb 15 13:37:24 iZuf6aytwpcvzkwiisgaxaZ systemd[1]: Stopped Shadowsocks.
Feb 15 13:37:24 iZuf6aytwpcvzkwiisgaxaZ systemd[1]: Started Shadowsocks.
Feb 15 13:37:24 iZuf6aytwpcvzkwiisgaxaZ systemd[1108]: shadowsocks.service: Failed at step EXEC spawning /usr/bin
Feb 15 13:37:24 iZuf6aytwpcvzkwiisgaxaZ systemd[1]: shadowsocks.service: Main process exited, code=exited, status
Feb 15 13:37:24 iZuf6aytwpcvzkwiisgaxaZ systemd[1]: shadowsocks.service: Unit entered failed state.
Feb 15 13:37:24 iZuf6aytwpcvzkwiisgaxaZ systemd[1]: shadowsocks.service: Failed with result 'exit-code'.
lines 1-12/12 (END)
```

## 解决

想了很久，发现无法启动 Shadowsocks，然后在思考 shadowsocks.service 启动

```shell
[Unit]
Description=Shadowsocks

[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json

[Install]
WantedBy=multi-user.target
```

然后去 `/usr/bin/ssserver` 并不存在，然后重新 `pip unuinstall shadowsocks`发现，安装路径不对

```shell
root@iZuf6aytwpcvzkwiisgaxaZ:/usr/bin# pip uninstall shadowsocks
DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. A future version of pip will drop support for Python 2.7. More details about Python 2 support in pip, can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support
Found existing installation: shadowsocks 2.8.2
Uninstalling shadowsocks-2.8.2:
  Would remove:
    /usr/local/bin/sslocal
    /usr/local/bin/ssserver
    /usr/local/lib/python2.7/dist-packages/shadowsocks-2.8.2.dist-info/*
    /usr/local/lib/python2.7/dist-packages/shadowsocks/*
Proceed (y/n)? n
```

所以解决方式是把 `/etc/systemed/system/shadowsocks.service` 中的配置修改为 `/usr/local/bin/ssserver`