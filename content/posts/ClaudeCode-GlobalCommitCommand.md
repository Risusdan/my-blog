+++
draft = false
date = 2026-01-17T16:46:22+08:00
title = "如何在 macOS 設定 Claude Code 全域自訂指令"
tags = []
categories = ['ClaudeCode']
+++

Claude Code 支援自訂 slash command，讓你可以建立常用的指令模板。本文介紹如何在 macOS 上設定**全域指令**，讓你在任何專案中都能使用，無需在每個專案重複設定。

---

## 專案指令 vs 全域指令

Claude Code 的自訂指令有兩種層級：

| 類型     | 路徑                  | 適用範圍     |
| -------- | --------------------- | ------------ |
| 專案指令 | `.claude/commands/`   | 僅限該專案   |
| 全域指令 | `~/.claude/commands/` | 所有專案通用 |

如果你有一個常用的指令（例如 `/commit`），放在全域路徑會更方便。

---

## 設定步驟

### 1. 建立全域指令目錄

```bash
mkdir -p ~/.claude/commands
```

### 2. 建立指令檔案

以 `/commit` 指令為例，建立 `commit.md`：

```bash
touch ~/.claude/commands/commit.md
```

### 3. 編輯指令內容

使用你習慣的編輯器開啟檔案：

```bash
code ~/.claude/commands/commit.md   # VS Code
# 或
nano ~/.claude/commands/commit.md   # Terminal
```

貼上以下內容：

```markdown
Review all changes in the working directory via git.

Guidelines:
1. Analyze ALL changes in the repository:
   - Staged changes
   - Unstaged changes
   - Untracked files
2. Determine which changes should be committed and how to group them logically
3. If changes span multiple unrelated features, fixes, or concerns, split them into separate commits as appropriate. Stage changes incrementally using `git add -p`, `git add <specific files>`, or `git add .` as needed.
4. Write commit messages following the Conventional Commits format:
   - Type: feat, fix, docs, style, refactor, test, chore, etc.
   - Optional scope in parentheses
   - Short description (imperative mood, lowercase, no period)
   - Optional body for more details in bullet points if the change is complex

Format:
```
<type>(<optional scope>): <description>

[optional body]
```

Examples:
- feat(auth): add login with Google OAuth
- fix: resolve null pointer exception in user service
- docs: update API documentation for v2 endpoints
- refactor(database): simplify connection pooling logic

When splitting commits:
- Group related changes together logically
- Each commit should be atomic and represent a single logical change
- Commit in a sensible order (e.g., refactoring before new features that depend on it)

After analyzing, propose the commit plan (single or multiple commits), including which files to stage for each commit, and ask for confirmation before proceeding.

Note:
- Do NOT append any AI-generated signatures, co-author tags, or tool attribution to commit messages. For example:
    - "🤖 Generated with [Claude Code](https://claude.com/claude-code)"
    - "Co-Authored-By: Claude <noreply@anthropic.com>"
```

### 4. 驗證設定

在任意專案目錄中啟動 Claude Code，輸入 `/commit`，應該就能看到你的自訂指令被觸發。

---

## 指令檔案格式說明

- **檔名**：決定指令名稱（`commit.md` → `/commit`）
- **內容**：純 Markdown 格式，作為 prompt 傳送給 Claude
- **參數**：可使用 `$ARGUMENTS` 佔位符接收使用者輸入

### 帶參數的指令範例

如果你想建立一個可接收參數的指令，例如 `/review <file>`：

```markdown
# ~/.claude/commands/review.md

Review the following file and provide feedback:

File: $ARGUMENTS

Please check for:
1. Code quality and readability
2. Potential bugs or edge cases
3. Performance considerations
4. Security issues
```

使用時輸入 `/review src/main.js`，`$ARGUMENTS` 會被替換為 `src/main.js`。

---

## 常用全域指令建議

除了 `/commit`，以下是一些適合設為全域指令的範例：

| 指令        | 用途            |
| ----------- | --------------- |
| `/commit`   | 智慧 git commit |
| `/review`   | Code review     |
| `/explain`  | 解釋程式碼      |
| `/refactor` | 重構建議        |
| `/test`     | 生成測試案例    |

---

## 目錄結構範例

設定完成後，你的全域指令目錄結構會像這樣：

```
~/.claude/
└── commands/
    ├── commit.md
    ├── review.md
    └── explain.md
```

---

## 小結

透過全域指令設定，你可以：

- 在所有專案中使用一致的工作流程
- 避免重複在每個專案建立相同的指令
- 隨時新增、修改指令，立即在所有專案生效
