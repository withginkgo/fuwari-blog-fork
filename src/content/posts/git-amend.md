---
title: Git 提交中的 chore 类型及修复错误提交的实战
published: 2026-04-13
tags: [Git, Conventional Commits, version control]
category: utilities
draft: false
---

在日常开发或维护个人博客时，我们经常需要写 Git 提交信息。你是否见过 `chore` 这样的提交类型？又是否曾经因为匆忙提交并推送了一个包含错误的代码而懊恼？本文将从两方面帮助你：一是理解 `chore` 的含义，二是学会如何优雅地修复已经推送的错误提交。

## 一、`chore` 是什么？

在 [Conventional Commits（约定式提交）](https://www.conventionalcommits.org/) 规范中，`chore` 表示**既不修复错误（`fix`），也不添加新功能（`feat`）** 的杂项工作。可以把它理解为“项目杂务”或“日常维护”。

### 常见使用场景

- **依赖管理**：升级、添加或删除 npm / pip / Maven 依赖包  
  `chore: bump lodash to 4.17.21`

- **配置文件修改**：`.gitignore`、`.eslintrc`、`webpack.config.js` 等  
  `chore: ignore .env file`

- **构建脚本与 CI/CD**：修改 `Makefile`、`package.json` 的 scripts、GitHub Actions 配置  
  `chore: add cache to GitHub Actions`

- **项目脚手架**：初始化项目、添加版权头、更新许可证文件

- **自动生成代码**：protobuf、GraphQL types 等

### 与其他提交类型的区别

| 类型 | 含义 | 是否影响业务逻辑 |
|------|------|----------------|
| `feat` | 新功能 | ✅ 是 |
| `fix` | 修复 bug | ✅ 是 |
| `refactor` | 重构（行为不变） | ✅ 是（代码变化但功能不变） |
| `docs` | 仅文档修改 | ❌ 否 |
| `style` | 代码格式（空格、分号） | ❌ 否 |
| `test` | 添加或修正测试 | ❌ 否（不影响生产代码） |
| **`chore`** | **构建过程、工具、依赖更新** | **❌ 否** |

> **一句话总结**：`chore` 是“做杂活”的提交 —— 用户不关心，但开发者需要维护项目卫生和工具链。

## 二、实战：提交错误后如何修复？

假设你修改了博客项目的头像路径，但粗心在路径末尾多了一个空格：

```typescript
avatar: "assets/images/demo.jpg ",  // 末尾有空格
```

你没有检查就直接 `git commit` 并 `git push` 到了远程仓库。TypeScript 编译报错，你很快在本地修正了空格。现在远程仓库包含一个错误提交，你希望用**最干净**的方式修复它（而不是新增一个“修复错误”的提交）。

### 解决方案：`git commit --amend` + `--force-with-lease`

```bash
# 1. 将修正后的文件加入暂存区
git add <修改的头像文件路径>   # 例如 git add src/config/avatar.ts

# 2. 修正上一次提交（保留原提交信息）
git commit --amend --no-edit

# 3. 强制推送到远程分支（安全模式）
git push --force-with-lease origin <你的分支名>
```

#### 参数说明
- `--amend`：把当前暂存区的修改合并到**上一次提交**中，而不是创建新提交。
- `--no-edit`：不修改原来的提交信息。如果你想顺便改提交信息，去掉此参数即可。
- `--force-with-lease`：比 `--force` 更安全，它会检查远程分支是否和你本地记录一致，避免覆盖他人新推送的提交。

### 为什么不用新建一个修复提交？

你也可以这样做：
```bash
git add ...
git commit -m "fix: 修复头像路径空格"
git push
```

但这样会在 Git 历史中留下两次提交：一次错误 + 一次修正。对于个人博客或追求整洁历史的小项目，使用 `--amend` 能保持“一次提交解决一个完整功能”，历史更清晰。

### 注意风险

- 如果其他人已经基于你错误的提交进行了开发（比如团队协作），**不要**改写历史。此时应新增一个修复提交并推送。
- 对于个人博客项目，通常只有你一个人操作，`--force-with-lease` 非常安全。

## 三、最佳实践：避免类似错误

1. **提交前检查**：
   ```bash
   git diff --cached        # 查看暂存区的变更
   git commit -v            # 提交时显示 diff
   ```

2. **使用 Git 钩子**：在 `.git/hooks/pre-commit` 中运行代码检查（如 TypeScript 编译、ESLint）。

3. **遵循约定式提交**：让提交信息规范统一，方便日后查看和自动化生成 Changelog。

## 四、总结

- `chore` 是维护类提交，不影响用户可见的逻辑。
- 如果错误提交已经推送，使用 `git commit --amend` + `git push --force-with-lease` 可以改写最后一次提交，保持历史干净。
- 提交前养成检查的习惯，可以避免大部分不必要的修正操作。

希望这篇小文章能帮助你更好地使用 Git，写出更专业的提交历史。如果你有任何疑问或更好的实践，欢迎交流讨论！
