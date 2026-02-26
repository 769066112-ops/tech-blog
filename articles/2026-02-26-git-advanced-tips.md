# Git 高级技巧：90%的人不知道的10个实用命令

> 用了这么久 Git，你可能只用了 add/commit/push。这10个命令让你效率翻倍。

## 1. git stash — 临时保存工作

```bash
git stash                          # 保存当前修改
git stash save "正在做登录功能"     # 带备注
git stash pop                      # 恢复并删除
git stash list                     # 查看所有 stash
```

## 2. git rebase -i — 整理提交历史

```bash
git rebase -i HEAD~3  # 整理最近3个commit，squash合并
```

## 3. git bisect — 二分法找 bug

```bash
git bisect start
git bisect bad          # 当前有bug
git bisect good v1.0    # v1.0没bug
# Git自动checkout中间commit，你测试后标记good/bad
```

## 4. git cherry-pick — 摘取特定提交

```bash
git cherry-pick abc1234  # 只要某个commit
```

## 5. git reflog — 后悔药

```bash
git reflog                    # 查看所有操作记录
git reset --hard HEAD@{2}     # 恢复到任意状态
```

## 6. git worktree — 同时操作多个分支

```bash
git worktree add ../hotfix hotfix-branch  # 新目录开分支
```

## 7. git blame — 谁写的这行代码

```bash
git blame src/app.py  # 每行显示作者和时间
```

## 8. git log 高级用法

```bash
git log --oneline --graph --all   # 图形化
git log --grep="登录"              # 搜索
git log --author="张三"            # 按作者
git shortlog -sn                   # 统计提交数
```

## 9. git clean — 清理未跟踪文件

```bash
git clean -n    # 预览
git clean -fd   # 删除
```

## 10. git config 实用别名

```bash
git config --global alias.co checkout
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --all"
```

收藏备用，用到的时候翻出来看 👍
