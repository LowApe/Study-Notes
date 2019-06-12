
# 目录

# git 工作流

1. 主干开发
2. GitFlow (多分支)
3. GitHub Flow (集成)
4. GitLab Flow (带生产分支:时间)(带环境分布:环境)(带发布分支:设备)

# 分支集成策略(Merge)

- Commit (多个提交:结合主分支)
    ![](http://ww1.sinaimg.cn/large/006rAlqhly1g3w6k61w0nj30mj05r3zd.jpg)
- Squash (提交组合成一个:放到主分支)
    ![](http://ww1.sinaimg.cn/large/006rAlqhly1g3w6kmfi3hj30j807kmy7.jpg)
- Rebase (多个提交放到主分支)
    ![](http://ww1.sinaimg.cn/large/006rAlqhly1g3w6kyxxfaj30r509nwg2.jpg)

# project 管理 issue

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g3yol8kjh8j30xr0mejsw.jpg)

想项目相同特性放到一起,可以选择模板

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g3yoocjqk8j30bv0f9my4.jpg)

- 基本看板(Todo,progress,Done)
- 自动看板(自动移动 issues and pull 请求 到基本三个列)
- 自动看板的 review
- bug 看板

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g3yor5jw62j31c10lztb7.jpg)

# 项目实施 codereview

settings 设置分支规则,一般一类分支必须 review 后

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g3yoyi76e1j30sc0lswgr.jpg)

对特殊分支添加 review 人员,这样会发送邮件。

# 团队如何做多分支的集成

特性分支往主干合,`pull requests`,检查 conflicts。有冲突,手动解决冲突。会在特性分支解决冲突后合并

![](http://ww1.sinaimg.cn/large/006rAlqhly1g3ypdl19j6j30cj065q3m.jpg)
> 上面是 merge Commit

![](http://ww1.sinaimg.cn/large/006rAlqhly1g3ypkf6mi6j30a004gaag.jpg)
> 上面是 Squash merge


Rebase merge 如何使用本地合并
- 先拉取远端到本地(fetch)
- `git reabase orgin/master` 产生 conflict 修改
- git add .
- git rebase --continue
> 因为编辑,重复

> `gitk --all` 打开git 图像界面

![](http://ww1.sinaimg.cn/large/006rAlqhly1g3ypxa75h0j30ma0f575i.jpg)


![](http://ww1.sinaimg.cn/large/006rAlqhly1g3yq55cwebj30n0060aay.jpg)

如何处理反复提交的问题

`Rerere`命令
- `git config --global rerere.enable true`
- `git merge master` 解决冲突
- `git add .`
- `git commit -am`

- `git rebase --continue`
- `git add .`

> 重复上面两个,产生冲突,但是不需要有冲突标志,然后每次添加 add ,在继续
