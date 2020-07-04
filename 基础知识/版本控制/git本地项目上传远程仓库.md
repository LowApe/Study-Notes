# git本地项目上传远程仓库

1. 初始化项目

2. 添加到暂存区

3. 提交代码

4. 关联远程库：`git remote add origin(可修改) branch_Name(为空时默认为master) url`

	关联之后可以用`git remote -v `来检查是否关联成功

5. 一般情况需要先pull一下：`git pull origin master`

	一般情况下含有共同文件时需要执行 `git merge origin/master --allow-unrelated-histories`

	这之后解决一下冲突

6. `git push`

```shell
git init # 初始化项目
git add . # 添加项目到缓存区
git commit -m ""  # 提交代码
git remote add orgin url
git fetch  # 获取
git merge # 合并
git push # 上传至远程仓库
git remote rm xxx # 删除远程地址
```

如果没有添加公钥匙，需要添加

- [参考链接](https://www.runoob.com/git/git-remote-repo.html)







