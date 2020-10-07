# Linux文件说明

> CentOS 7 系统文件说明。Linux 有一句“一切皆文件”概念。说明文件的重要性



# Linux启动文件

## 系统引导概述

`/etc/inittab`

```properties
cat inittab
# 翻译：不再使用 inittab 文件使用系统
# inittab is no longer used when using systemd.
# 
# 翻译：在这里添加配置，系统将不会有影响
# ADDING CONFIGURATION HERE WILL HAVE NO EFFECT ON YOUR SYSTEM.
# 
# 翻译：Ctrl-Alt-Delete 操作在 /usr/lib/systemd/system/ctrl-alt-del.target 设置
# Ctrl-Alt-Delete is handled by /usr/lib/systemd/system/ctrl-alt-del.target
# 
# 翻译：系统使用 “targets” 代替 runlevel 。默认，这里有两个主要 targets
# systemd uses 'targets' instead of runlevels. By default, there are two main targets:
# 
# 翻译：multi-user.target: 类似 runlevel 3 (默认不启动图形界面，文件在上面提到的路径下)
# multi-user.target: analogous to runlevel 3
# 
# 翻译：graphical.target: 类似 runlevel 5 
# graphical.target: analogous to runlevel 5
# 
# 翻译：查看当前默认 target (默认 graphical.target)
# To view current default target, run:
# systemctl get-default
# 
# 翻译：设置默认target systemctl set-default 文件名.target
# To set a default target, run:
# systemctl set-default TARGET.target
#

```



`/usr/lib/systemd/system/multi-user.target`

```properties
[parallels@centos-7 system]$ cat multi-user.target
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
Conflicts=rescue.service rescue.target
After=basic.target rescue.service rescue.target
AllowIsolate=yes

```



```properties
cat graphical.target
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Graphical Interface
Documentation=man:systemd.special(7)
Requires=multi-user.target
Wants=display-manager.service
Conflicts=rescue.service rescue.target
After=multi-user.target rescue.service rescue.target display-manager.service
AllowIsolate=yes

```

## 系统运行级别

> Linux 默认有七个运行级别（runlevel）0～6
>
> - 0:关机
> - 1:单用户模式，系统出现问题，可使用这个模式进入系统维护，经典忘记root密码可进入该模式修改密码
> - 2:多用户模式，但是没有网络连接
> - 3:完全多用户模式，这也是Linux服务器最常见的运行级别
> - 4:保留
> - 5:窗口模式
> - 6:重启
>
> 注意⚠️：任何时候 Linux 只能在一种 runlevel 下运行。

虽然 inittab 不使用了，但是`/etc`下还是用对应rc0～rc1脚本

```shell
[parallels@centos-7 etc]$ ll rc3.d/
total 0
lrwxrwxrwx. 1 root root 20 Aug 10  2017 K50netconsole -> ../init.d/netconsole
lrwxrwxrwx. 1 root root 17 Aug 10  2017 S10network -> ../init.d/network
[parallels@centos-7 etc]$ ll rc1.d/
total 0
lrwxrwxrwx. 1 root root 20 Aug 10  2017 K50netconsole -> ../init.d/netconsole
lrwxrwxrwx. 1 root root 17 Aug 10  2017 K90network -> ../init.d/network

```

- 如上，K50、S10。系统以K开头的脚本运行，然后在运行 S开头

- 字母后面的数字，先小后大运行

- K 表示停止kill，S表示启动start

- 上面 运行级别3 关闭 netConsole 开启 network，运行级别1 关闭 network

	

## 服务启动脚本

> 上面介绍了 Linux 运行级别，会使用 K 或 S 开头的脚本启动或关闭相应的服务.
>
> 有关脚本

`/etc/init.d/network` :sos:后续学明白 shell 基本，再把加上加上

```sh
# bash 脚本开始的标识 /bin/bash 作为该文本的解释器
#! /bin/bash
# 说明自己的相对路径 
# network       Bring up/down networking
# 运行级别是 2345 默认开启 network start
# 默认为 on 优先级为 10
# 默认为 off 优先级为 90
# chkconfig: 2345 10 90
# description: Activates/Deactivates all network interfaces configured to \
#              start at boot time.
#
### BEGIN INIT INFO
# Provides: $network
# Should-Start: iptables ip6tables NetworkManager-wait-online NetworkManager $network-pre
# Short-Description: Bring up/down networking
# Description: Bring up/down networking
### END INIT INFO

# Source function library.
# "." 命令包含文件，可以使用该路径下定义的函数
. /etc/init.d/functions

if [ ! -f /etc/sysconfig/network ]; then
    exit 6
fi

. /etc/sysconfig/network

if [ -f /etc/sysconfig/pcmcia ]; then
    . /etc/sysconfig/pcmcia
fi


# Check that networking is up.
[ "${NETWORKING}" = "no" ] && exit 6

# if the ip configuration utility isn't around we can't function.
[ -x /sbin/ip ] || exit 1


CWD=$(pwd)
cd /etc/sysconfig/network-scripts

. ./network-functions

# find all the interfaces besides loopback.
# ignore aliases, alternative configurations, and editor backup files
interfaces=$(ls ifcfg-* | \
        LC_ALL=C sed -e "$__sed_discard_ignored_files" \
               -e '/\(ifcfg-lo$\|:\|ifcfg-.*-range\)/d' \
               -e '{ s/^ifcfg-//g;s/[0-9]/ &/}' | \
        LC_ALL=C sort -k 1,1 -k 2n | \
        LC_ALL=C sed 's/ //')
rc=0

# See how we were called.
case "$1" in
start)
    [ "$EUID" != "0" ] && exit 4
    rc=0
    # IPv6 hook (pre IPv4 start)
    if [ -x /etc/sysconfig/network-scripts/init.ipv6-global ]; then
        /etc/sysconfig/network-scripts/init.ipv6-global start pre
    fi

    apply_sysctl

    #tell NM to reload its configuration
    if [ "$(LANG=C nmcli -t --fields running general status 2>/dev/null)" = "running" ]; then
        nmcli connection reload
    fi

    # bring up loopback interface
    action $"Bringing up loopback interface: " ./ifup ifcfg-lo

    case "$VLAN" in
    yes)
        if [ ! -d /proc/net/vlan ] && ! modprobe 8021q >/dev/null 2>&1 ; then
            net_log $"No 802.1Q VLAN support available in kernel."
        fi
        ;;
    esac

    vlaninterfaces=""
    vpninterfaces=""
    xdslinterfaces=""
    bridgeinterfaces=""

    # bring up all other interfaces configured to come up at boot time
    for i in $interfaces; do
        unset DEVICE TYPE SLAVE NM_CONTROLLED
        eval $(LANG=C grep -F "DEVICE=" ifcfg-$i)
        eval $(LANG=C grep -F "TYPE=" ifcfg-$i)
        eval $(LANG=C grep -F "SLAVE=" ifcfg-$i)
        eval $(LANG=C grep -F "NM_CONTROLLED=" ifcfg-$i)

        if [ -z "$DEVICE" ] ; then DEVICE="$i"; fi

        if [ "$SLAVE" = "yes" ] && ( ! is_nm_running || is_false $NM_CONTROLLED ) ; then
            continue
        fi

        if [ "${DEVICE##cipcb}" != "$DEVICE" ] ; then
            vpninterfaces="$vpninterfaces $i"
            continue
        fi
        if [ "$TYPE" = "xDSL"  -o  "$TYPE" = "Modem" ]; then
            xdslinterfaces="$xdslinterfaces $i"
            continue
        fi

        if [ "$TYPE" = "Bridge" ]; then
            bridgeinterfaces="$bridgeinterfaces $i"
            continue
        fi
        if [ "$TYPE" = "IPSEC" ] || [ "$TYPE" = "IPIP" ] || [ "$TYPE" = "GRE" ]; then
            vpninterfaces="$vpninterfaces $i"
            continue
        fi

        if [ "${DEVICE%%.*}" != "$DEVICE"  -o  "${DEVICE##vlan}" != "$DEVICE" ] ; then
            vlaninterfaces="$vlaninterfaces $i"
            continue
        fi

        if LANG=C grep -EL "^ONBOOT=['\"]?[Nn][Oo]['\"]?" ifcfg-$i > /dev/null ; then
            # this loads the module, to preserve ordering
            is_available $i
            continue
        fi
        action $"Bringing up interface $i: " ./ifup $i boot
        [ $? -ne 0 ] && rc=1
    done

    # Bring up xDSL and VPN interfaces
    for i in $vlaninterfaces $bridgeinterfaces $xdslinterfaces $vpninterfaces ; do
        if ! LANG=C grep -EL "^ONBOOT=['\"]?[Nn][Oo]['\"]?" ifcfg-$i >/dev/null 2>&1 ; then
            action $"Bringing up interface $i: " ./ifup $i boot
            [ $? -ne 0 ] && rc=1
        fi
    done

    # Add non interface-specific static-routes.
    if [ -f /etc/sysconfig/static-routes ]; then
        if [ -x /sbin/route ]; then
            grep "^any" /etc/sysconfig/static-routes | while read ignore args ; do
                /sbin/route add -$args
            done
        else
            net_log $"Legacy static-route support not available: /sbin/route not found"
        fi
    fi

    # IPv6 hook (post IPv4 start)
    if [ -x /etc/sysconfig/network-scripts/init.ipv6-global ]; then
        /etc/sysconfig/network-scripts/init.ipv6-global start post
    fi
    # Run this again to catch any interface-specific actions
    apply_sysctl

    touch /var/lock/subsys/network

    [ -n "${NETWORKDELAY}" ] && /bin/sleep ${NETWORKDELAY}
    ;;
stop)
    [ "$EUID" != "0" ] && exit 4
    # Don't shut the network down if root or /usr is on NFS or a network
    # block device.
    if systemctl show --property=RequiredBy -- -.mount usr.mount | grep -q 'remote-fs.target' ; then
        net_log $"rootfs or /usr is on network filesystem, leaving network up"
        exit 1
    fi

    vlaninterfaces=""
    vpninterfaces=""
    xdslinterfaces=""
    bridgeinterfaces=""
    remaining=""
    rc=0

    # get list of bonding, vpn, and xdsl interfaces
    for i in $interfaces; do
        unset DEVICE TYPE
        eval $(LANG=C grep -F "DEVICE=" ifcfg-$i)
        eval $(LANG=C grep -F "TYPE=" ifcfg-$i)

        if [ -z "$DEVICE" ] ; then DEVICE="$i"; fi

        if [ "${DEVICE##cipcb}" != "$DEVICE" ] ; then
            vpninterfaces="$vpninterfaces $i"
            continue
        fi
        if [ "$TYPE" = "IPSEC" ] || [ "$TYPE" = "IPIP" ] || [ "$TYPE" = "GRE" ]; then
            vpninterfaces="$vpninterfaces $i"
            continue
        fi
        if [ "$TYPE" = "Bridge" ]; then
            bridgeinterfaces="$bridgeinterfaces $i"
            continue
        fi
        if [ "$TYPE" = "xDSL"  -o  "$TYPE" = "Modem" ]; then
            xdslinterfaces="$xdslinterfaces $i"
            continue
        fi

        if [ "${DEVICE%%.*}" != "$DEVICE"  -o  "${DEVICE##vlan}" != "$DEVICE" ] ; then
            vlaninterfaces="$vlaninterfaces $i"
            continue
        fi
        remaining="$remaining $i"
    done

    for i in $vpninterfaces $xdslinterfaces $bridgeinterfaces $vlaninterfaces $remaining; do
        unset DEVICE TYPE
        (. ./ifcfg-$i
        if [ -z "$DEVICE" ] ; then DEVICE="$i"; fi

        if ! check_device_down $DEVICE; then
            action $"Shutting down interface $i: " ./ifdown $i boot
            [ $? -ne 0 ] && rc=1
        fi
        )
    done

    action $"Shutting down loopback interface: " ./ifdown ifcfg-lo

    sysctl -w net.ipv4.ip_forward=0 > /dev/null 2>&1

    # IPv6 hook (post IPv4 stop)
    if [ -x /etc/sysconfig/network-scripts/init.ipv6-global ]; then
        /etc/sysconfig/network-scripts/init.ipv6-global stop post
    fi

    rm -f /var/lock/subsys/network
    ;;
status)
    echo $"Configured devices:"
    echo lo $interfaces

    echo $"Currently active devices:"
    echo $(/sbin/ip -o link show up | awk -F ": " '{ print $2 }')
    ;;
restart|reload|force-reload)
    cd "$CWD"
    $0 stop
    $0 start
    rc=$?
    ;;
*)
    echo $"Usage: $0 {start|stop|status|restart|reload|force-reload}"
    exit 2
esac

exit $rc

```



`/etc/init.d/functions`

```sh
[parallels@centos-7 init.d]$ cat functions 
# -*-Shell-script-*-
#
# functions This file contains functions to be used by most or all
#       shell scripts in the /etc/init.d directory.
#

TEXTDOMAIN=initscripts

# Make sure umask is sane
umask 022

# Set up a default search path.
PATH="/sbin:/usr/sbin:/bin:/usr/bin"
export PATH

if [ $PPID -ne 1 -a -z "$SYSTEMCTL_SKIP_REDIRECT" ] && \
        [ -d /run/systemd/system ] ; then
    case "$0" in
    /etc/init.d/*|/etc/rc.d/init.d/*)
        _use_systemctl=1
        ;;
    esac
fi

systemctl_redirect () {
    local s
    local prog=${1##*/}
    local command=$2
    local options=""

    case "$command" in
    start)
        s=$"Starting $prog (via systemctl): "
        ;;
    stop)
        s=$"Stopping $prog (via systemctl): "
        ;;
    reload|try-reload)
        s=$"Reloading $prog configuration (via systemctl): "
        ;;
    restart|try-restart|condrestart)
        s=$"Restarting $prog (via systemctl): "
        ;;
    esac

    if [ -n "$SYSTEMCTL_IGNORE_DEPENDENCIES" ] ; then
        options="--ignore-dependencies"
    fi

    if ! systemctl show "$prog.service" > /dev/null 2>&1 || \
            systemctl show -p LoadState "$prog.service" | grep -q 'not-found' ; then
        action $"Reloading systemd: " /bin/systemctl daemon-reload
    fi

    action "$s" /bin/systemctl $options $command "$prog.service"
}

# Get a sane screen width
[ -z "${COLUMNS:-}" ] && COLUMNS=80

if [ -z "${CONSOLETYPE:-}" ]; then
    if [ -c "/dev/stderr" -a -r "/dev/stderr" ]; then
        CONSOLETYPE="$(/sbin/consoletype < /dev/stderr 2>/dev/null)"
    else
        CONSOLETYPE="serial"
    fi
fi

if [ -z "${NOLOCALE:-}" ] && [ -z "${LANGSH_SOURCED:-}" ] && \
        [ -f /etc/sysconfig/i18n -o -f /etc/locale.conf ] ; then
    . /etc/profile.d/lang.sh 2>/dev/null
    # avoid propagating LANGSH_SOURCED any further
    unset LANGSH_SOURCED
fi

# Read in our configuration
if [ -z "${BOOTUP:-}" ]; then
    if [ -f /etc/sysconfig/init ]; then
        . /etc/sysconfig/init
    else
        # This all seem confusing? Look in /etc/sysconfig/init,
        # or in /usr/share/doc/initscripts-*/sysconfig.txt
        BOOTUP=color
        RES_COL=60
        MOVE_TO_COL="echo -en \\033[${RES_COL}G"
        SETCOLOR_SUCCESS="echo -en \\033[1;32m"
        SETCOLOR_FAILURE="echo -en \\033[1;31m"
        SETCOLOR_WARNING="echo -en \\033[1;33m"
        SETCOLOR_NORMAL="echo -en \\033[0;39m"
        LOGLEVEL=1
    fi
    if [ "$CONSOLETYPE" = "serial" ]; then
        BOOTUP=serial
        MOVE_TO_COL=
        SETCOLOR_SUCCESS=
        SETCOLOR_FAILURE=
        SETCOLOR_WARNING=
        SETCOLOR_NORMAL=
    fi
fi

# Check if any of $pid (could be plural) are running
checkpid() {
    local i

    for i in $* ; do
        [ -d "/proc/$i" ] && return 0
    done
    return 1
}

__kill_pids_term_kill_checkpids() {
    local base_stime=$1
    shift 1
    local pid=
    local pids=$*
    local remaining=
    local stat=
    local stime=

    for pid in $pids ; do
        [ ! -e  "/proc/$pid" ] && continue
        read -r line < "/proc/$pid/stat" 2> /dev/null

        stat=($line)
        stime=${stat[21]}

        [ -n "$stime" ] && [ "$base_stime" -lt "$stime" ] && continue
        remaining+="$pid "
    done

    echo "$remaining"
    [ -n "$remaining" ] && return 1

    return 0
}

__kill_pids_term_kill() {
    local try=0
    local delay=3;
    local pid=
    local stat=($(< /proc/self/stat))
    local base_stime=${stat[21]}

    if [ "$1" = "-d" ]; then
        delay=$2
        shift 2
    fi

    local kill_list=$*

    kill_list=$(__kill_pids_term_kill_checkpids $base_stime $kill_list)

    [ -z "$kill_list" ] && return 0

    kill -TERM $kill_list >/dev/null 2>&1
    sleep 0.1

    kill_list=$(__kill_pids_term_kill_checkpids $base_stime $kill_list)
    if [ -n "$kill_list" ] ; then
        while [ $try -lt $delay ] ; do
            sleep 1
            kill_list=$(__kill_pids_term_kill_checkpids $base_stime $kill_list)
            [ -z "$kill_list" ] && break
            let try+=1
        done
        if [ -n "$kill_list" ] ; then
            kill -KILL $kill_list >/dev/null 2>&1
            sleep 0.1
            kill_list=$(__kill_pids_term_kill_checkpids $base_stime $kill_list)
        fi
    fi

    [ -n "$kill_list" ] && return 1
    return 0
}

# __proc_pids {program} [pidfile]
# Set $pid to pids from /var/run* for {program}.  $pid should be declared
# local in the caller.
# Returns LSB exit code for the 'status' action.
__pids_var_run() {
    local base=${1##*/}
    local pid_file=${2:-/var/run/$base.pid}
    local pid_dir=$(/usr/bin/dirname $pid_file > /dev/null)
    local binary=$3

    [ -d "$pid_dir" -a ! -r "$pid_dir" ] && return 4

    pid=
    if [ -f "$pid_file" ] ; then
            local line p

        [ ! -r "$pid_file" ] && return 4 # "user had insufficient privilege"
        while : ; do
            read line
            [ -z "$line" ] && break
            for p in $line ; do
                if [ -z "${p//[0-9]/}" ] && [ -d "/proc/$p" ] ; then
                    if [ -n "$binary" ] ; then
                        local b=$(readlink /proc/$p/exe | sed -e 's/\s*(deleted)$//')
                        [ "$b" != "$binary" ] && continue
                    fi
                    pid="$pid $p"
                fi
            done
        done < "$pid_file"

            if [ -n "$pid" ]; then
                    return 0
            fi
        return 1 # "Program is dead and /var/run pid file exists"
    fi
    return 3 # "Program is not running"
}

# Output PIDs of matching processes, found using pidof
__pids_pidof() {
    pidof -c -m -o $$ -o $PPID -o %PPID -x "$1" || \
        pidof -c -m -o $$ -o $PPID -o %PPID -x "${1##*/}"
}


# A function to start a program.
daemon() {
    # Test syntax.
    local gotbase= force= nicelevel corelimit
    local pid base= user= nice= bg= pid_file=
    local cgroup=
    nicelevel=0
    while [ "$1" != "${1##[-+]}" ]; do
        case $1 in
        '')
            echo $"$0: Usage: daemon [+/-nicelevel] {program}" "[arg1]..."
            return 1
            ;;
        --check)
            base=$2
            gotbase="yes"
            shift 2
            ;;
        --check=?*)
            base=${1#--check=}
            gotbase="yes"
            shift
            ;;
        --user)
            user=$2
            shift 2
            ;;
        --user=?*)
            user=${1#--user=}
            shift
            ;;
        --pidfile)
            pid_file=$2
            shift 2
            ;;
        --pidfile=?*)
            pid_file=${1#--pidfile=}
            shift
            ;;
        --force)
            force="force"
            shift
            ;;
        [-+][0-9]*)
            nice="nice -n $1"
            shift
            ;;
        *)
            echo $"$0: Usage: daemon [+/-nicelevel] {program}" "[arg1]..."
            return 1
            ;;
      esac
    done

    # Save basename.
    [ -z "$gotbase" ] && base=${1##*/}

    # See if it's already running. Look *only* at the pid file.
    __pids_var_run "$base" "$pid_file"

    [ -n "$pid" -a -z "$force" ] && return

    # make sure it doesn't core dump anywhere unless requested
    corelimit="ulimit -S -c ${DAEMON_COREFILE_LIMIT:-0}"

    # if they set NICELEVEL in /etc/sysconfig/foo, honor it
    [ -n "${NICELEVEL:-}" ] && nice="nice -n $NICELEVEL"

    # if they set CGROUP_DAEMON in /etc/sysconfig/foo, honor it
    if [ -n "${CGROUP_DAEMON}" ]; then
        if [ ! -x /bin/cgexec ]; then
            echo -n "Cgroups not installed"; warning
            echo
        else
            cgroup="/bin/cgexec";
            for i in $CGROUP_DAEMON; do
                cgroup="$cgroup -g $i";
            done
        fi
    fi

    # Echo daemon
    [ "${BOOTUP:-}" = "verbose" -a -z "${LSB:-}" ] && echo -n " $base"

    # And start it up.
    if [ -z "$user" ]; then
       $cgroup $nice /bin/bash -c "$corelimit >/dev/null 2>&1 ; $*"
    else
       $cgroup $nice runuser -s /bin/bash $user -c "$corelimit >/dev/null 2>&1 ; $*"
    fi

    [ "$?" -eq 0 ] && success $"$base startup" || failure $"$base startup"
}

# A function to stop a program.
killproc() {
    local RC killlevel= base pid pid_file= delay try binary=

    RC=0; delay=3; try=0
    # Test syntax.
    if [ "$#" -eq 0 ]; then
        echo $"Usage: killproc [-p pidfile] [ -d delay] {program} [-signal]"
        return 1
    fi
    if [ "$1" = "-p" ]; then
        pid_file=$2
        shift 2
    fi
    if [ "$1" = "-b" ]; then
        if [ -z $pid_file ]; then
            echo $"-b option can be used only with -p"
            echo $"Usage: killproc -p pidfile -b binary program"
            return 1
        fi
        binary=$2
        shift 2
    fi
    if [ "$1" = "-d" ]; then
        delay=$(echo $2 | awk -v RS=' ' -v IGNORECASE=1 '{if($1!~/^[0-9.]+[smhd]?$/) exit 1;d=$1~/s$|^[0-9.]*$/?1:$1~/m$/?60:$1~/h$/?60*60:$1~/d$/?24*60*60:-1;if(d==-1) exit 1;delay+=d*$1} END {printf("%d",delay+0.5)}')
        if [ "$?" -eq 1 ]; then
            echo $"Usage: killproc [-p pidfile] [ -d delay] {program} [-signal]"
            return 1
        fi
        shift 2
    fi


    # check for second arg to be kill level
    [ -n "${2:-}" ] && killlevel=$2

    # Save basename.
    base=${1##*/}

    # Find pid.
    __pids_var_run "$1" "$pid_file" "$binary"
    RC=$?
    if [ -z "$pid" ]; then
        if [ -z "$pid_file" ]; then
            pid="$(__pids_pidof "$1")"
        else
            [ "$RC" = "4" ] && { failure $"$base shutdown" ; return $RC ;}
        fi
    fi

    # Kill it.
    if [ -n "$pid" ] ; then
        [ "$BOOTUP" = "verbose" -a -z "${LSB:-}" ] && echo -n "$base "
        if [ -z "$killlevel" ] ; then
            __kill_pids_term_kill -d $delay $pid
            RC=$?
            [ "$RC" -eq 0 ] && success $"$base shutdown" || failure $"$base shutdown"
        # use specified level only
        else
            if checkpid $pid; then
                kill $killlevel $pid >/dev/null 2>&1
                RC=$?
                [ "$RC" -eq 0 ] && success $"$base $killlevel" || failure $"$base $killlevel"
            elif [ -n "${LSB:-}" ]; then
                RC=7 # Program is not running
            fi
        fi
    else
        if [ -n "${LSB:-}" -a -n "$killlevel" ]; then
            RC=7 # Program is not running
        else
            failure $"$base shutdown"
            RC=0
        fi
    fi

    # Remove pid file if any.
    if [ -z "$killlevel" ]; then
        rm -f "${pid_file:-/var/run/$base.pid}"
    fi
    return $RC
}

# A function to find the pid of a program. Looks *only* at the pidfile
pidfileofproc() {
    local pid

    # Test syntax.
    if [ "$#" = 0 ] ; then
        echo $"Usage: pidfileofproc {program}"
        return 1
    fi

    __pids_var_run "$1"
    [ -n "$pid" ] && echo $pid
    return 0
}

# A function to find the pid of a program.
pidofproc() {
    local RC pid pid_file=

    # Test syntax.
    if [ "$#" = 0 ]; then
        echo $"Usage: pidofproc [-p pidfile] {program}"
        return 1
    fi
    if [ "$1" = "-p" ]; then
        pid_file=$2
        shift 2
    fi
    fail_code=3 # "Program is not running"

    # First try "/var/run/*.pid" files
    __pids_var_run "$1" "$pid_file"
    RC=$?
    if [ -n "$pid" ]; then
        echo $pid
        return 0
    fi

    [ -n "$pid_file" ] && return $RC
    __pids_pidof "$1" || return $RC
}

status() {
    local base pid lock_file= pid_file= binary=

    # Test syntax.
    if [ "$#" = 0 ] ; then
        echo $"Usage: status [-p pidfile] {program}"
        return 1
    fi
    if [ "$1" = "-p" ]; then
        pid_file=$2
        shift 2
    fi
    if [ "$1" = "-l" ]; then
        lock_file=$2
        shift 2
    fi
    if [ "$1" = "-b" ]; then
        if [ -z $pid_file ]; then
            echo $"-b option can be used only with -p"
            echo $"Usage: status -p pidfile -b binary program"
            return 1
        fi
        binary=$2
        shift 2
    fi
    base=${1##*/}

    if [ "$_use_systemctl" = "1" ]; then
        systemctl status ${0##*/}.service
        ret=$?
        # LSB daemons that dies abnormally in systemd looks alive in systemd's eyes due to RemainAfterExit=yes
        # lets adjust the reality a little bit
        if systemctl show -p ActiveState ${0##*/}.service | grep -q '=active$' && \
        systemctl show -p SubState ${0##*/}.service | grep -q '=exited$' ; then
            ret=3
        fi
        return $ret
    fi

    # First try "pidof"
    __pids_var_run "$1" "$pid_file" "$binary"
    RC=$?
    if [ -z "$pid_file" -a -z "$pid" ]; then
        pid="$(__pids_pidof "$1")"
    fi
    if [ -n "$pid" ]; then
        echo $"${base} (pid $pid) is running..."
        return 0
    fi

    case "$RC" in
    0)
        echo $"${base} (pid $pid) is running..."
        return 0
        ;;
    1)
        echo $"${base} dead but pid file exists"
        return 1
        ;;
    4)
        echo $"${base} status unknown due to insufficient privileges."
        return 4
        ;;
    esac
    if [ -z "${lock_file}" ]; then
        lock_file=${base}
    fi
    # See if /var/lock/subsys/${lock_file} exists
    if [ -f /var/lock/subsys/${lock_file} ]; then
        echo $"${base} dead but subsys locked"
        return 2
    fi
    echo $"${base} is stopped"
    return 3
}

echo_success() {
    [ "$BOOTUP" = "color" ] && $MOVE_TO_COL
    echo -n "["
    [ "$BOOTUP" = "color" ] && $SETCOLOR_SUCCESS
    echo -n $"  OK  "
    [ "$BOOTUP" = "color" ] && $SETCOLOR_NORMAL
    echo -n "]"
    echo -ne "\r"
    return 0
}

echo_failure() {
    [ "$BOOTUP" = "color" ] && $MOVE_TO_COL
    echo -n "["
    [ "$BOOTUP" = "color" ] && $SETCOLOR_FAILURE
    echo -n $"FAILED"
    [ "$BOOTUP" = "color" ] && $SETCOLOR_NORMAL
    echo -n "]"
    echo -ne "\r"
    return 1
}

echo_passed() {
    [ "$BOOTUP" = "color" ] && $MOVE_TO_COL
    echo -n "["
    [ "$BOOTUP" = "color" ] && $SETCOLOR_WARNING
    echo -n $"PASSED"
    [ "$BOOTUP" = "color" ] && $SETCOLOR_NORMAL
    echo -n "]"
    echo -ne "\r"
    return 1
}

echo_warning() {
    [ "$BOOTUP" = "color" ] && $MOVE_TO_COL
    echo -n "["
    [ "$BOOTUP" = "color" ] && $SETCOLOR_WARNING
    echo -n $"WARNING"
    [ "$BOOTUP" = "color" ] && $SETCOLOR_NORMAL
    echo -n "]"
    echo -ne "\r"
    return 1
}

# Inform the graphical boot of our current state
update_boot_stage() {
    if [ -x /bin/plymouth ]; then
        /bin/plymouth --update="$1"
    fi
    return 0
}

# Log that something succeeded
success() {
    [ "$BOOTUP" != "verbose" -a -z "${LSB:-}" ] && echo_success
    return 0
}

# Log that something failed
failure() {
    local rc=$?
    [ "$BOOTUP" != "verbose" -a -z "${LSB:-}" ] && echo_failure
    [ -x /bin/plymouth ] && /bin/plymouth --details
    return $rc
}

# Log that something passed, but may have had errors. Useful for fsck
passed() {
    local rc=$?
    [ "$BOOTUP" != "verbose" -a -z "${LSB:-}" ] && echo_passed
    return $rc
}

# Log a warning
warning() {
    local rc=$?
    [ "$BOOTUP" != "verbose" -a -z "${LSB:-}" ] && echo_warning
    return $rc
}

# Run some action. Log its output.
action() {
    local STRING rc

    STRING=$1
    echo -n "$STRING "
    shift
    "$@" && success $"$STRING" || failure $"$STRING"
    rc=$?
    echo
    return $rc
}

# returns OK if $1 contains $2
strstr() {
    [ "${1#*$2*}" = "$1" ] && return 1
    return 0
}

# Check whether file $1 is a backup or rpm-generated file and should be ignored
is_ignored_file() {
    case "$1" in
    *~ | *.bak | *.old | *.orig | *.rpmnew | *.rpmorig | *.rpmsave)
        return 0
        ;;
    esac
    return 1
}

# Convert the value ${1} of time unit ${2}-seconds into seconds:
convert2sec() {
  local retval=""

  case "${2}" in
    deci)   retval=$(awk "BEGIN {printf \"%.1f\", ${1} / 10}") ;;
    centi)  retval=$(awk "BEGIN {printf \"%.2f\", ${1} / 100}") ;;
    mili)   retval=$(awk "BEGIN {printf \"%.3f\", ${1} / 1000}") ;;
    micro)  retval=$(awk "BEGIN {printf \"%.6f\", ${1} / 1000000}") ;;
    nano)   retval=$(awk "BEGIN {printf \"%.9f\", ${1} / 1000000000}") ;;
    piko)   retval=$(awk "BEGIN {printf \"%.12f\", ${1} / 1000000000000}") ;;
  esac

  echo "${retval}"
}

# Evaluate shvar-style booleans
is_true() {
    case "$1" in
    [tT] | [yY] | [yY][eE][sS] | [oO][nN] | [tT][rR][uU][eE] | 1)
        return 0
        ;;
    esac
    return 1
}

# Evaluate shvar-style booleans
is_false() {
    case "$1" in
    [fF] | [nN] | [nN][oO] | [oO][fF][fF] | [fF][aA][lL][sS][eE] | 0)
        return 0
        ;;
    esac
    return 1
}

# Apply sysctl settings, including files in /etc/sysctl.d
apply_sysctl() {
    if [ -x /lib/systemd/systemd-sysctl ]; then
    /lib/systemd/systemd-sysctl
    else
        for file in /usr/lib/sysctl.d/*.conf ; do
            is_ignored_file "$file" && continue
            [ -f /run/sysctl.d/${file##*/} ] && continue
            [ -f /etc/sysctl.d/${file##*/} ] && continue
            test -f "$file" && sysctl -e -p "$file" >/dev/null 2>&1
        done
        for file in /run/sysctl.d/*.conf ; do
            is_ignored_file "$file" && continue
            [ -f /etc/sysctl.d/${file##*/} ] && continue
            test -f "$file" && sysctl -e -p "$file" >/dev/null 2>&1
        done
        for file in /etc/sysctl.d/*.conf ; do
            is_ignored_file "$file" && continue
            test -f "$file" && sysctl -e -p "$file" >/dev/null 2>&1
        done
        sysctl -e -p /etc/sysctl.conf >/dev/null 2>&1
    fi
}

# A sed expression to filter out the files that is_ignored_file recognizes
__sed_discard_ignored_files='/\(~\|\.bak\|\.old\|\.orig\|\.rpmnew\|\.rpmorig\|\.rpmsave\)$/d'

if [ "$_use_systemctl" = "1" ]; then
        if  [ "x$1" = xstart -o \
              "x$1" = xstop -o \
              "x$1" = xrestart -o \
              "x$1" = xreload -o \
              "x$1" = xtry-restart -o \
              "x$1" = xforce-reload -o \
              "x$1" = xcondrestart ] ; then

        systemctl_redirect $0 $1
        exit $?
    fi
fi

strstr "$(cat /proc/cmdline)" "rc.debug" && set -x
return 0


```

## Grub介绍

`/boot`

```shell
[root@centos-7 boot]# ll
total 130952
-rw-r--r--. 1 root root   147859 Aug 15  2018 config-3.10.0-862.11.6.el7.x86_64
drwxr-xr-x. 3 root root       17 Oct 31  2017 efi
drwxr-xr-x. 2 root root       27 Aug 10  2017 grub
drwx------. 5 root root      167 Aug 20  2018 grub2
-rw-------. 1 root root 64535399 Aug 10  2017 initramfs-0-rescue-06b16d48947a4b42817d1cc2aad546cd.img
-rw-------. 1 root root 30842237 Oct  6 15:04 initramfs-3.10.0-862.11.6.el7.x86_64.img
-rw-------. 1 root root 13014531 Oct  6 15:04 initramfs-3.10.0-862.11.6.el7.x86_64kdump.img
-rw-r--r--. 1 root root 10184140 Oct 31  2017 initrd-plymouth.img
-rw-r--r--. 1 root root   305158 Aug 15  2018 symvers-3.10.0-862.11.6.el7.x86_64.gz
-rw-------. 1 root root  3414344 Aug 15  2018 System.map-3.10.0-862.11.6.el7.x86_64
-rwxr-xr-x. 1 root root  5392080 Aug 10  2017 vmlinuz-0-rescue-06b16d48947a4b42817d1cc2aad546cd
-rwxr-xr-x. 1 root root  6242208 Aug 15  2018 vmlinuz-3.10.0-862.11.6.el7.x86_64

```

- initrd 初始化内存磁盘
- vmlinuz.. 内核
- config ... 配置文件

`/boot/grub2`

```shell
[root@centos-7 grub2]# ls -l
total 48
-rw-r--r--. 1 root root   64 Aug 10  2017 device.map
drwxr-xr-x. 2 root root   25 Aug 10  2017 fonts
-rw-r--r--. 1 root root 4258 Aug 20  2018 grub.cfg
-rw-r--r--. 1 root root 4243 Aug 10  2017 grub.cfg.1502366733.rpmsave
-rw-r--r--. 1 root root 4258 Aug 10  2017 grub.cfg.1509454303.rpmsave
-rw-r--r--. 1 root root 1024 Aug 20  2018 grubenv
drwxr-xr-x. 2 root root 8192 Aug 10  2017 i386-pc
drwxr-xr-x. 2 root root 4096 Aug 10  2017 locale

```

## 获得帮助





# Linux 密码文件

## /etc/passwd

- 每一行 六个 `:`分割
	- 用户名
	- 密码
	- UID
	- GID
	- 说明栏
	- 家路径
	- 登陆后使用 shell
- 

```properties
[root@centos-7 etc]# cat passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
... 省略 ...
```



## /etc/shadow

> Linux 将密码移动到 `/etc/shadow`,默认只用 root 用户有权限读取。



- 八个`:`隔开的
	- 用户名
	- 加密后的密码
	- 密码最近修改日
	- 密码修改后多少天不可修改 0 表示都可以
	- 密码重新修改的天数，考虑密码泄漏
	- 密码失效前提示天数
	- 密码宽限天数后密码失效
	- 账号失效日期一般为空
	- 保留字段

```shell
[root@centos-7 etc]# cat shadow
root:$6$CdzvjRxb$VLuWeiNf0GviM1W4nKkFaqV8zKvVCej4ovhTIW9AVwvlaIS3QcF5jefotAdIT/nADPok0cD2Z2jDQ7NYyeV.q0:18541:0:99999:7:::
bin:*:17110:0:99999:7:::
daemon:*:17110:0:99999:7:::
adm:*:17110:0:99999:7:::
lp:*:17110:0:99999:7:::
sync:*:17110:0:99999:7:::
shutdown:*:17110:0:99999:7:::
halt:*:17110:0:99999:7:::
mail:*:17110:0:99999:7:::
operator:*:17110:0:99999:7:::
games:*:17110:0:99999:7:::
ftp:*:17110:0:99999:7:::
nobody:*:17110:0:99999:7:::
systemd-bus-proxy:!!:17388::::::
systemd-network:!!:17388::::::
dbus:!!:17388::::::

```

