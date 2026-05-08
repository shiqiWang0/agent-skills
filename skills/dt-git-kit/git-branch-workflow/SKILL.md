---
name: git-branch-workflow
description: Git 分支流程与合并策略
---

# Git Branch Workflow Skill

指导 feat / fix / hotfix / daily sync 分支的创建、合并和删除流程。

## feat 分支流程

```bash
# 1. 从 release 创建 feat 分支
git checkout release
git pull origin release
git checkout -b console/feat_7.0.x_{id}

# 2. 开发完成后合并到 test（需 MR）
git checkout console/feat_7.0.x_{id}
git merge release
git push origin console/feat_7.0.x_{id}

# 3. 删除 feat 分支
git branch -d console/feat_7.0.x_{id}
git push origin --delete console/feat_7.0.x_{id}
```

## fix 分支流程

```bash
# 1. 从 test 创建 fix 分支
git checkout test/7.0.x
git pull origin test/7.0.x
git checkout -b console/fix_7.0.x_{id}

# 2. 修复完成后合并回 release（需 MR）
git checkout console/fix_7.0.x_{id}
git merge test
git push origin console/fix_7.0.x_{id}

# 3. 删除 fix 分支
git branch -d console/fix_7.0.x_{id}
```

## hotfix 分支流程

```bash
# 1. 从 release 创建 hotfix 分支
git checkout release
git pull origin release
git checkout -b console/hotfix_7.0.x_{id}

# 2. 修复完成后同时合并到 release（需 MR）
git checkout console/hotfix_7.0.x_{id}
git merge release
git push origin console/hotfix_7.0.x_{id}

# 3. 删除 hotfix 分支
git branch -d console/hotfix_7.0.x_{id}
```

## daily sync 分支流程（同步 dt-common）

```bash
# 从 test 分支合并对应版本的 dt-common
git checkout test/7.0.x
git pull origin dt-common/dev_15.x.y
git merge dt-common/dev_15.x.y
```