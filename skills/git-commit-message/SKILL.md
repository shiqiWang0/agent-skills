---
name: git-commit-message
description: Git Commit Message 生成与校验
---

# Git Commit Message Skill

校验和生成符合规范的 commit message。

## 格式

`<type>: <commit message>`

| 字段 | 说明 |
|------|------|
| **type** | `feat`, `fix`, `hotfix`, `docs`, `style`, `refactor`, `test`, `build`, `ci`, `chore` |
| **commit message** | 简洁描述本次变更，建议 ≤80 字符 |

## 示例

```
feat: add user authentication feature
fix: #19210, resolve memory leak in data processor
chore: update dependencies and refactor build config
refactor: simplify workflow tool conversion
```

## 规则

- 当前分支为 main / release → 禁止直接 commit
- 不强制添加 scope
- 不强制添加 issue / bug id；如适用，可按项目习惯写成 `fix: #19210, ...`
- **绝对不能执行 git push**

## type 推断

- 新功能 → feat
- Bug 修复 → fix
- 紧急线上问题 → hotfix
- 配置/依赖 → chore / build
- 重构 → refactor
