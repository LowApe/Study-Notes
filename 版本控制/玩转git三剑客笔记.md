# 目录

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [目录](#目录)
- [版本管理的演变](#版本管理的演变)
	- [集中式 VCS](#集中式-vcs)
	- [分布式 VCS](#分布式-vcs)
		- [git 特点](#git-特点)
- [git 安装](#git-安装)
- [git 最小配置](#git-最小配置)
- [创建第一个仓库并配置 local 用户信息](#创建第一个仓库并配置-local-用户信息)
- [工作区和暂存区](#工作区和暂存区)

<!-- /TOC -->eOnSave:1 orderedList:0 -->

- [目录](#目录)
- [版本管理的演变](#版本管理的演变)
	- [集中式 VCS](#集中式-vcs)
	- [分布式 VCS](#分布式-vcs)
		- [git 特点](#git-特点)
- [git 安装](#git-安装)

<!-- /TOC -->

# 版本管理的演变
## 集中式 VCS
特点:
- 有集中的服务器
- 具备文件版本管理和分支管理能力
- 集成效率有明显地提高
- **客户端必须时刻和服务器相连**

## 分布式 VCS
特点:
- 服务端和客户端都由完整的版本库
- 脱离服务器,客户端照样可以管理版本
- 查看历史和版本比较多数操作,都不需要访问服务器

> 代表:git

### git 特点

- **最优的存储能力**
- **非凡的性能**
- 开源的
- 很容易做备份
- 支持离线操作
- 很容易定制工作流程

> git 衍生产品: gitHub gitLab

# git 安装

> git 安装自行百度

# git 最小配置

配置 user.name 和 user.email

```
$git config --global user.name 'your_name'
$git config --global user.email 'your_email@domain.com'
```
config 三种作用域
```
$ git config --local  // 只对某个仓库有效
$ git config --global // 对当前用户所有仓库有效
$ git config --system // 对系统所有登录的的用户有效
```
显示 config 的配置加 --list
```
$ git config --list --local  // 查看指定仓库的设置,优先级高
$ git config --list --global
$ git config --list --system
```

# 创建第一个仓库并配置 local 用户信息

两种场景:
1. 把已有的项目代码纳入 Git 管理
2. 新建的项目直接用 Git 管理

```
$ git init
$ git init project_name
```

# 工作区和暂存区

可以参考[Git笔记](https://blogcode.top/2017/09/26/Git%E7%BB%A7%E7%BB%AD%E5%AD%A6%E4%B9%A0/)

![参考视频图片](http://ww1.sinaimg.cn/large/006rAlqhly1g4p032fgsdj30lh0bl0ur.jpg)
