---
name: git-commands
description: Git 基础命令速查
---

# Git Commands Skill

常用 Git 命令速查表。

## 分支操作

| 命令 | 说明 |
|------|------|
| `git branch` | 列出本地分支 |
| `git branch -a` | 列出所有分支（包括远程） |
| `git branch <name>` | 创建新分支 |
| `git branch -d <name>` | 删除分支（已合并） |
| `git branch -D <name>` | 强制删除分支 |
| `git checkout <branch>` | 切换分支 |
| `git checkout -b <branch>` | 创建并切换到新分支 |
| `git switch <branch>` | 切换分支（现代命令） |
| `git switch -c <branch>` | 创建并切换（现代命令） |

## 提交操作

| 命令 | 说明 |
|------|------|
| `git status` | 查看工作区状态 |
| `git add <file>` | 暂存文件 |
| `git add .` | 暂存所有变更 |
| `git commit -m "<msg>"` | 提交 |
| `git commit --amend` | 修改最后一次 commit |
| `git log --oneline -10` | 查看最近 10 条 commit |
| `git diff` | 查看工作区变更 |
| `git diff --staged` | 查看暂存区变更 |

## 同步操作

| 命令 | 说明 |
|------|------|
| `git fetch` | 拉取远程元数据 |
| `git pull` | 拉取并合并 |
| `git push` | 推送到远程 |
| `git push origin <branch>` | 推送到指定分支 |
| `git push -u origin <branch>` | 推送并设置上游分支 |

## 合并操作

| 命令 | 说明 |
|------|------|
| `git merge <branch>` | 合并分支 |
| `git merge --no-ff <branch>` | 合并（禁止快进） |
| `git merge --abort` | 放弃合并 |
| `git merge --continue` | 继续合并（解决冲突后） |