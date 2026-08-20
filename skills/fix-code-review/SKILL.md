---
name: fix-code-review
description: GitLab MR 代码审查修复技能。用于获取指定 MR 的 review comments/discussions、定位相关变更文件并修复问题，输出可人工校验的修改说明与 diff，在用户明确确认后再执行 commit，不自动 push 或 merge。用户提到“修复 MR review”“处理 code review comments”“根据 MR 链接修复审查意见”时使用。
---

# GitLab Code Review 修复技能

执行 GitLab MR review 问题处理闭环：拉取评论、定位代码、逐条修复、人工校验、确认后提交。

## 工作流

1. 识别 MR 来源（MR 编号或 MR 链接）。
2. 获取 discussions/comments 并筛选待处理项。
3. 获取 MR 变更文件和对应文件内容。
4. 逐条修复 review 问题并保持最小改动。
5. 生成人工校验材料（问题清单、修改说明、diff、建议 commit message）。
6. 仅在用户明确确认后执行 commit；不自动 push/merge。

## 1) 获取 MR Review Comments

使用 `mr_discussions`：

```json
{
  "project_id": "2351",
  "mr_iid": "MR编号"
}
```

## 2) 获取 MR 变更文件

使用 `gitlab_mr_listner`：

```json
{
  "project_id": "2351",
  "mr_iid": "MR编号"
}
```

## 3) 获取文件内容

使用 `get_file_contents`：

```json
{
  "project_id": "2351",
  "file_path": "src/components/Button.tsx",
  "ref": "分支名"
}
```

## 4) 执行修复

修复原则：
- 逐条处理，不遗漏 comment
- 只修复明确问题，不引入额外改动
- 简单问题直接修复；复杂逻辑先分析再改
- 改动较大或需求不清时先向用户确认
- 保持原项目代码风格和边界

## 5) 人工校验阶段（必选）

commit 前必须先向用户展示：
- 待修复 comment 列表
- 每条 comment 的修复说明
- 修改前/修改后 diff
- 建议 commit message

并要求用户明确指令之一后再继续：
- `确认提交`
- `仅提交第 X 条修改`
- `修改后再提交`

## 6) 提交规则

使用 frontend-dev-mcp 的 `generate-commit-message` 执行提交动作；禁止自动 push。

```json
{
  "project_id": "2351",
  "branch": "修复分支名",
  "message": "fix: code review to fix,#MR",
  "actions": [
    {
      "action": "update",
      "file_path": "src/components/Button.tsx",
      "content": "修复后的代码内容"
    }
  ]
}
```

提交粒度默认按“每个文件或每组相关问题”拆分。

## 输入识别

场景 A：用户直接给 MR 编号，例如“修复 MR #123 的代码审查问题”。

场景 B：用户给 MR 链接，例如 `https://gitlab.prod.dtstack.cn/dt-ai-works/dt-ai-works-front/-/merge_requests/456`，需先提取 `mr_iid=456`。

## 开始处理时的响应模板

```text
开始处理 MR #XXX 的代码审查问题，共发现 N 个待处理项。

待修复问题：
1. [文件名:行号] 问题描述
2. [文件名:行号] 问题描述
...

正在获取文件内容并开始修复...
```
