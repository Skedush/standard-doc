# git使用

## 基础操作


### 1.添加修改文件

```
git add
```

### 2.本地提交

```
git commit | cz （cz是前端替代commit的工具）
```

### 3.拉取线上代码（rebase）
```
# 指定具体分支名
git pull origin [branch] --rebase
```

如果遇到冲突，解决冲突后添加修改的文件

```
git add [冲突的文件]
```

继续rebase

```
git rebase --continue
```

### 4.提交到远程
```
git push origin [branch]
```

## 特殊操作

### 回滚本地提交

如果你推送到remote的commit没有被其他人pull过，那么你可以使用

```
git reset --hard HEAD~1
git push -f origin master
```

来撤销之前提交的commit


但是如果有其他人同步过你的push，那么你可以在本地使用revert来还原你提交的commit，然后生成一个新的commit然后再推送到远端

```
$ git revert HEAD
```

### cherry-pick

```
git cherry-pick <commitHash>
```

更多相关知识看[这里](https://www.ruanyifeng.com/blog/2020/04/git-cherry-pick.html)

### stash

把所有未提交的修改（包括暂存的和非暂存的）都保存起来，用于后续恢复当前工作目录

```
git stash
```

重新应用缓存的stash

```
git stash pop 或者 git stash apply（了解这两个的不同）
```
