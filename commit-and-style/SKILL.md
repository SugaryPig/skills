---
name: commit-and-style
description: Generates commit messages following project conventions (feat/fix, issue number from branch or last remote commit) and runs pnpm stylelint:fix before commit when style does not comply. Use when committing code, writing commit messages, fixing style errors, or when the user mentions commit format or stylelint.
---

# Commit 与样式规范

本技能用于在任意项目中按规范生成 commit 文案，并在提交前处理样式格式问题。

## Commit 文案格式

**格式：** `{type}: #{number} {message}`

- **type**：`feat`（新增功能/内容）或 `fix`（修复 bug）
- **分隔**：英文冒号 `:` + 一个空格
- **number**：见下方「获取 issue 编号」
- **message**：本次提交的简短描述（中文或英文均可）

**示例：**
- `feat: #12345 新增自选股加入自选弹窗`
- `fix: #12345 修复行情页闪屏问题`

## 获取 issue 编号

1. **优先从当前分支名取数**：若分支名中包含数字（如 `feature/12345-xxx`、`fix/12345`），则使用该数字。
2. **分支无数字时**：使用**上一次远程提交**的 commit 中的 issue 编号（从 `origin` 当前分支或默认分支最近一条 commit message 里解析 `#数字`）。

若无法从分支或远程历史解析到数字，可询问用户提供 issue 编号，或说明无法自动获取需手动填写。

## 提交前样式处理

- 若本次修改包含样式文件（如 `.css`、`.less`、`.scss`、`.vue` 等），且提交时出现 **style 相关规范报错**，先执行样式自动修复：
  ```bash
  pnpm stylelint:fix
  ```
- 修复完成后再执行 `git add` 与 `git commit`，commit 文案仍按上述格式书写。

## 工作流小结

1. 确定本次是**新增**（feat）还是**修 bug**（fix）。
2. 按「获取 issue 编号」得到 `#{number}`。
3. 若有样式报错，先执行 `pnpm stylelint:fix`，再提交。
4. 最终 commit：`feat: #12345 简短描述` 或 `fix: #12345 简短描述`。
