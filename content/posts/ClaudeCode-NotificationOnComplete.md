+++
draft = false
date = 2026-01-17T21:00:00+08:00
title = "如何在 macOS 設定 Claude Code 完成通知"
tags = ["hooks", "notification"]
categories = ['ClaudeCode']
+++

當你使用 Claude Code 執行較長的任務時，可能會想切換到其他視窗工作。本文介紹如何透過 **Hooks 機制** 設定 macOS 系統通知，讓你在 Claude 完成回應時收到提醒。

---

## 功能特色

這個方案具備以下特點：

| 特色            | 說明                                                              |
| --------------- | ----------------------------------------------------------------- |
| Session 識別    | 使用 `session_id` 區分不同 session，支援多個 Claude Code 同時運作 |
| 顯示原始 Prompt | 通知會顯示你當初輸入的 prompt（前 100 字元）                      |
| 自動清理        | 暫存檔案在通知發送後立即刪除，不佔用空間                          |
| 原生通知        | 使用 macOS 內建的通知系統，包含提示音效                           |

---

## 運作原理

這個方案使用兩個 Hook 協同運作：

```
┌─────────────────────┐     ┌─────────────────────┐
│  UserPromptSubmit   │     │        Stop         │
│ (When prompt sent)  │     │ (Claude response    │
│                     │     │      complete)      │
├─────────────────────┤     ├─────────────────────┤
│ Save prompt to      │ ──▶ │ Read temp file,     │
│ temp file           │     │ send notification   │
│                     │     │ Delete temp file    │
│                     │     │ (cleanup)           │
└─────────────────────┘     └─────────────────────┘
```

**Hook 事件說明：**

| Hook 事件          | 觸發時機                 | 用途                       |
| ------------------ | ------------------------ | -------------------------- |
| `UserPromptSubmit` | 你按下 Enter 送出 prompt | 儲存 prompt 內容供稍後使用 |
| `Stop`             | Claude 完成回應          | 發送通知並清理暫存檔       |

---

## 前置需求

確認你的 macOS 已安裝 `jq`（用於解析 JSON）：

```bash
# 檢查是否已安裝
which jq

# 若未安裝，使用 Homebrew 安裝
brew install jq
```

---

## 實作步驟

### 步驟 1：建立儲存 Prompt 的 Script

建立 `~/.claude/save_prompt.sh`：

```bash
touch ~/.claude/save_prompt.sh
chmod +x ~/.claude/save_prompt.sh
```

編輯內容：

```bash
#!/bin/bash

# Saves the user prompt to a temp file when submitted
# Used by UserPromptSubmit hook

INPUT=$(cat)
PROMPT=$(echo "$INPUT" | jq -r ".prompt" | cut -c 1-100)
SESSION_ID=$(echo "$INPUT" | jq -r ".session_id")

echo "$PROMPT" > "/tmp/claude_session_${SESSION_ID}_prompt"
```

**說明：**
- `INPUT=$(cat)` — 從 stdin 讀取 Claude Code 傳入的 JSON
- `jq -r ".prompt"` — 解析 JSON 取得 prompt 內容
- `cut -c 1-100` — 只保留前 100 字元避免通知過長
- 暫存檔以 `session_id` 命名，支援多 session 同時運作

---

### 步驟 2：建立通知 Script

建立 `~/.claude/notify_done.sh`：

```bash
touch ~/.claude/notify_done.sh
chmod +x ~/.claude/notify_done.sh
```

編輯內容：

```bash
#!/bin/bash

# Sends macOS notification when Claude finishes, then cleans up
# Used by Stop hook

INPUT=$(cat)
SESSION_ID=$(echo "$INPUT" | jq -r ".session_id")
PROMPT_FILE="/tmp/claude_session_${SESSION_ID}_prompt"

PROMPT=$(cat "$PROMPT_FILE" 2>/dev/null || echo "Task")

# Send notification
osascript -e "display notification \"$PROMPT\" with title \"Claude Code Finished\" sound name \"Glass\""

# Auto-cleanup
rm -f "$PROMPT_FILE"
```

**說明：**
- 讀取對應 session 的暫存檔取得原始 prompt
- 使用 `osascript` 發送 macOS 原生通知
- `sound name "Glass"` — 播放提示音效（可改為其他系統音效）
- `rm -f` — 通知發送後立即刪除暫存檔

---

### 步驟 3：設定 Hooks

編輯 `~/.claude/settings.json`，加入 hooks 設定：

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "/bin/bash ~/.claude/save_prompt.sh"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "/bin/bash ~/.claude/notify_done.sh"
          }
        ]
      }
    ]
  }
}
```

如果你已有其他設定，將 `hooks` 區塊合併進去即可。

**或者使用 Claude Code 內建指令設定：**

1. 在 Claude Code 中輸入 `/hooks`
2. 選擇 `UserPromptSubmit` 事件，新增 hook：`/bin/bash ~/.claude/save_prompt.sh`
3. 選擇 `Stop` 事件，新增 hook：`/bin/bash ~/.claude/notify_done.sh`
4. 儲存到 User settings

---

### 步驟 4：驗證設定

重新啟動 Claude Code 或開啟新 session，輸入任意 prompt。當 Claude 完成回應後，你應該會：

1. 聽到 "Glass" 提示音
2. 看到 macOS 通知，顯示你的 prompt 內容

---

## 檔案結構總覽

設定完成後，你的 `~/.claude/` 目錄應該包含：

```
~/.claude/
├── settings.json          # Hooks 設定
├── save_prompt.sh         # 儲存 prompt 的 script
└── notify_done.sh         # 發送通知的 script
```

---

## 自訂選項

### 更換提示音效

macOS 內建多種音效，可在 `notify_done.sh` 中修改：

```bash
# 可用的音效名稱
sound name "Glass"      # 預設
sound name "Ping"
sound name "Pop"
sound name "Purr"
sound name "Submarine"
sound name "Blow"
```

### 調整 Prompt 顯示長度

在 `save_prompt.sh` 中修改 `cut` 參數：

```bash
# 顯示前 200 字元
PROMPT=$(echo "$INPUT" | jq -r ".prompt" | cut -c 1-200)
```

### Debug：查看 Hook 收到的 JSON

在 script 中加入以下程式碼可以輸出完整 JSON 供除錯：

```bash
# 在 INPUT=$(cat) 之後加入
echo "$INPUT" > ~/.claude/debug_hook_input.json
```

---

## 停用通知

如果想暫時停用通知功能：

**方法 1：使用 /hooks 指令**

在 Claude Code 中輸入 `/hooks`，刪除相關的 hook 設定。

**方法 2：編輯 settings.json**

移除 `~/.claude/settings.json` 中的 `hooks` 區塊。

**方法 3：移除執行權限**

```bash
chmod -x ~/.claude/save_prompt.sh ~/.claude/notify_done.sh
```

---

## 延伸閱讀

- [Claude Code Hooks 官方文件](https://docs.anthropic.com/en/docs/claude-code/hooks)
- [GitHub 討論串：Add a way to display a notification when a prompt has finished #6454](https://github.com/anthropics/claude-code/issues/6454)

---

## 小結

透過 Claude Code 的 Hooks 機制，你可以輕鬆實現完成通知功能：

- **不需修改 Claude Code 本身** — 純粹透過設定檔和 shell script
- **支援多 session** — 每個 session 獨立追蹤
- **自動清理** — 不會留下垃圾檔案
- **完全可自訂** — 音效、顯示內容都可調整

現在你可以放心地在等待 Claude 回應時切換到其他工作，完成時會自動收到通知提醒。
