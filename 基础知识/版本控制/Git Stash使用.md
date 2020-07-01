# Git Stash 使用

> 相关场景：当我们正在编辑，或者开发过程中，自己分支需要更新主要分支最新的内容，才能保证你能继续进行下去，直接的合并是不允许的，需要你提交，但如果你修改的内容还不足以提交，则可以使用 Git stash 暂存

## 基本用法

```shell
# 存储信息，之后你修改的文件就会移除到 stash 中
# 之后干净的分支就能进行合并了
git stash save "message"

# 查看存储列表
git stash list
stash@{0}: On brachName: message
# 查看存储内容做的改动，
# 默认显示第一个，后缀添加 stash@{number} 显示其他
git stash show
# 恢复工作目录，感觉跟 apply 相同...
git stash pop
# 移除具体
git stash drop stash@{number}
# 清除所有缓存
git stash clear
```

> 如果没有 git add 到git 中的文件是不同 stash 存储的

# 相关连接

- [[git stash 用法总结和注意点](https://www.cnblogs.com/zndxall/p/9586088.html)](https://www.cnblogs.com/zndxall/archive/2018/09/04/9586088.html)

