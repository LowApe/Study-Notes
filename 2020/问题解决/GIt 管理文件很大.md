# Git 管理文件很大

貌似下面两个链接并没解决，该问题出现场景：

> 当误上传了 很大 jar 包，虽然在版本控制里删除了 jar 包，但是因为 git 版本记录里还是存出jar 文件，以至于.git 文件很大，因为git 不同分支不同内容其实都存储到了.git 文件中，所有还需要处理，将远程的信息删除

# 相关链接

- [如何解决 GitHub 提交次数过多 .git 文件过大的问题？](https://www.zhihu.com/question/29769130)
- [git filter-branch](https://git-scm.com/docs/git-filter-branch)

