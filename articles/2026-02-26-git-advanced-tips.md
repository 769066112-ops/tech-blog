# Git 高级技巧：90%的人不知道的10个实用命令

> 用了这么久 Git，你可能只用了 add/commit/push。这10个命令让你效率翻倍。

## 1. git stash — 临时保存工作

正在写代码，突然要切分支修 bug？

```bash
# 保存当前修改
git stash

# 切分支修 bug
git checkout hotfix
# ... 修完了

# 切回来，恢复修改
git checkout feature
git stash pop
```

高级用法：

```bash
git stash save "正在做登录功能"   # 带备注
git stash list                    # 查看所有 stash
git stash apply stash@{2}        # 恢复指定 stash
git stash drop stash@{0}         # 删除指定 stash
```

## 2. git rebase -i — 整理提交历史

提交了一堆乱七八糟的 commit？合并整理一下：

```bash
# 整理最近 3 个 commit
git rebase -i HEAD~3
```

编辑器里把 `pick` 改成：
- `squash` (s)：合并到上一个 commit
- `reword` (r)：修改 commit message
- `drop` (d)：删除这个 commit

```
pick abc1234 feat: 用户登录
squash def5678 fix: 修复登录bug
squash ghi9012 fix: 又修了一个bug
```

结果：三个 commit 合并成一个干净的提交。

## 3. git bisect — 二分法找 bug

不知道哪个 commit 引入了 bug？让 Git 帮你二分查找：

```bash
git bisect start
git bisect bad              # 当前版本有 bug
git bisect good v1.0        # v1.0 没有 bug

# Git 会自动 checkout 中间的 commit
# 你测试一下，告诉它好还是坏
git bisect good  # 或 git bisect bad

# 重复几次，Git 就能定位到引入 bug 的 commit
git bisect reset  # 结束
```

## 4. git cherry-pick — 摘取特定提交

只想要某个分支的某个 commit？

```bash
git cherry-pick abc1234
```

多个 commit：

```bash
git cherry-pick abc1234 def5678
git cherry-pick abc1234..def5678  # 范围
```

## 5. git reflog — 后悔药

误删了分支？reset 错了？reflog 记录了你的所有操作：

```bash
git reflog

# 输出类似：
# abc1234 HEAD@{0}: reset: moving to HEAD~3
# def5678 HEAD@{1}: commit: 重要的代码
# ghi9012 HEAD@{2}: checkout: moving from main to feature

# 恢复到任意状态
git reset --hard def5678
```

## 6. git worktree — 同时操作多个分支

不用来回切分支了，直接开多个工作目录：

```bash
# 在新目录打开另一个分支
git worktree add ../hotfix hotfix-branch

# 两个目录同时工作，互不影响
cd ../hotfix
# 修 bug...

# 用完删掉
git worktree remove ../hotfix
```

## 7. git blame — 谁写的这行代码

```bash
git blame src/app.py

# 输出每一行的作者和时间
# abc1234 (张三 2026-01-15) def calculate():
# def5678 (李四 2026-02-01)     return x + y
```

VS Code 里装 GitLens 插件更方便。

## 8. git log 高级用法

```bash
# 图形化显示分支
git log --oneline --graph --all

# 搜索 commit message
git log --grep="登录"

# 查看某个文件的修改历史
git log --follow -p src/auth.py

# 查看某人的提交
git log --author="张三" --since="2026-01-01"

# 统计每人提交数
git shortlog -sn
```

## 9. git clean — 清理未跟踪文件

```bash
git clean -n    # 预览要删除的文件
git clean -fd   # 删除未跟踪的文件和目录
git clean -fX   # 只删除 .gitignore 忽略的文件
```

## 10. git config 实用配置

```bash
# 设置常用别名
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --all"

# 自动纠错
git config --global help.autocorrect 1

# 默认推送当前分支
git config --global push.default current

# 更好的 diff 算法
git config --global diff.algorithm histogram
```

## 总结

这10个命令覆盖了日常开发中90%的高级场景。建议收藏，用到的时候翻出来看。

---

*觉得有用就点赞，关注我获取更多开发技巧 👍*
