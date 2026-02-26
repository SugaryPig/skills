---
name: submit-to-remote
description: Submits changes to remote by project conventions: fix lint and style (and hst-husky pre-commit issues), commit with feat/fix and issue number (user-provided #number first, then branch name, then last remote git log, else YYYYMMDD000), then push. Use when the user wants to commit and push, submit to remote (e.g. 推一下 #6796264766), fix pre-commit or husky errors, or handle lint/style before commit.
---

# 按规范提交到远端

本技能用于按规范把新增或修改的内容提交到远端：提交前处理 lint、style 及 hst-husky 预检问题，按规范生成 commit 并**在通过后直接推送到远端**。

## 整体流程

1. **提交前**：处理 lint、style、hst-husky 等问题，确保 pre-commit 能通过。
2. **提交**：按约定格式写 commit message 并执行 `git commit`。
3. **提交通过后**：直接执行 `git push` 推送到远端。

---

## 一、提交前：处理 pre-commit（hst-husky）

项目使用 **hst-husky**（husky + lint-staged），pre-commit 会跑 lint 和 stylelint。提交前必须让这些检查通过。

**推荐顺序：**

1. **先统一修复 lint 和 style**（避免反复被 hook 拦截）：
   ```bash
   pnpm lint:all:fix
   ```
   等价于先 `pnpm lint:fix` 再 `pnpm stylelint:fix`。

2. 若仍有 **lint 报错**（如 ESLint）：根据报错修代码或配置，必要时再跑一次 `pnpm lint:fix`。

3. 若仍有 **style 报错**（如 stylelint）：根据报错修样式，或再跑一次 `pnpm stylelint:fix`。

4. 若 **commit 时**被 commitlint 拦截：检查 commit message 是否满足下方「Commit 文案格式」。

直到 `git commit` 成功（即 hst-husky / pre-commit 全部通过），再进入下一步。

---

## 二、Commit 文案格式

**格式：** `{type}: #{number} {message}`

- **type**：`feat`（新增功能/内容）或 `fix`（修复 bug）
- **分隔**：英文冒号 `:` + 一个空格
- **number**：见下方「获取 issue 编号」
- **message**：本次提交的简短描述（中文或英文均可）

**示例：**
- `feat: #12345 新增自选股加入自选弹窗`
- `fix: #12345 修复行情页闪屏问题`

### 获取 issue 编号（井号后的数字）

按以下**优先级**取值，用第一个能拿到的即可：

1. **用户/Agent 显式提供**：用户或对话中已给出 `#数字` 时优先用该数字。例如用户说「推一下 #6796264766」则用 `#6796264766`。在 commit 文案里保留井号与数字即可（如 `feat: #6796264766 描述`）。
2. **从当前分支名解析**：分支名中含数字（如 `feature/12345-xxx`、`fix/6796264766`）则从分支名提取该数字作为 `#number`。
3. **从上一次远端 commit 解析**：若前两者都没有，则从**远端**最近一条 commit message 里解析 `#数字`。Agent 可执行以下命令获取并解析：
   - 当前分支对应的远端最近一条：`git log -1 origin/$(git branch --show-current) --pretty=format:"%s"`（若当前分支未推送过可先 `git fetch origin`，或改用 `origin/HEAD` 等）
   - 从输出中匹配 `#` 后连续数字作为 issue 编号。
4. **兜底**：若以上都拿不到数字，则使用 **当前日期 + 000**，格式为 `YYYYMMDD000`。例如 2026 年 2 月 26 日即为 `20260226000`，commit 中写 `feat: #20260226000 描述`。

---

## 三、提交通过后：推送到远端

commit 成功（pre-commit 全部通过）后，**直接执行**：

```bash
git push
```

无需用户再单独说「推一下」；按本技能执行即包含「提交并推送」的完整流程。

---

## 四、工作流小结

1. 运行 `pnpm lint:all:fix`，按报错修完 lint/style，直至 pre-commit 能通过。
2. 确定是 **feat**（新增）还是 **fix**（修 bug），并按优先级得到 issue 编号：用户提供的 `#数字` → 分支名中的数字 → 上次远端 git log 中的 `#数字` → 否则用 `YYYYMMDD000`。
3. 执行 `git add`（若尚未暂存），再 `git commit`，文案格式：`feat: #12345 描述` 或 `fix: #12345 描述`。
4. **Commit 通过后**执行 `git push`，把本次提交推送到远端。
