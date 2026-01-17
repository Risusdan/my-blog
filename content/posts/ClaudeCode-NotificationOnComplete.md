+++
draft = false
date = 2026-01-17T21:00:00+08:00
title = "如何在 macOS 設定 Claude Code 完成通知"
tags = []
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
| 原生通知        | 使用 macOS 內建的通知系統（預設靜音，可自訂音效）                 |
| 時間門檻        | 只有執行超過 10 秒的任務才會發送通知，避免快速回應打擾            |
| 防重複通知      | 使用標記檔案防止同一個 prompt 重複發送通知                        |

---

## 運作原理

這個方案使用兩個 Hook 協同運作：

```
┌─────────────────────┐     ┌─────────────────────┐
│  UserPromptSubmit   │     │        Stop         │
│ (When prompt sent)  │     │ (Claude response    │
│                     │     │      complete)      │
├─────────────────────┤     ├─────────────────────┤
│ Save prompt to      │ ──▶ │ Check elapsed time  │
│ temp file           │     │ (skip if < 10 sec)  │
│ Save timestamp      │     │ Check sent marker   │
│ Remove sent marker  │     │ Send notification   │
│                     │     │ Mark as sent        │
└─────────────────────┘     └─────────────────────┘
```

**Hook 事件說明：**

| Hook 事件          | 觸發時機                 | 用途                                   |
| ------------------ | ------------------------ | -------------------------------------- |
| `UserPromptSubmit` | 你按下 Enter 送出 prompt | 儲存 prompt 內容與時間戳，重置發送標記 |
| `Stop`             | Claude 完成回應          | 檢查時間門檻，發送通知並清理暫存檔     |

---

## 前置需求

使用 Homebrew 安裝所需工具：

```bash
brew install jq terminal-notifier
```

- `jq` — 用於解析 JSON
- `terminal-notifier` — 用於發送 macOS 通知

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

# Save prompt and timestamp
echo "$PROMPT" > "/tmp/claude_session_${SESSION_ID}_prompt"
date +%s > "/tmp/claude_session_${SESSION_ID}_time"

# Remove sent marker so new prompt can trigger notification
rm -f "/tmp/claude_session_${SESSION_ID}_sent"
```

**說明：**
- `INPUT=$(cat)` — 從 stdin 讀取 Claude Code 傳入的 JSON
- `jq -r ".prompt"` — 解析 JSON 取得 prompt 內容
- `cut -c 1-100` — 只保留前 100 字元避免通知過長
- `date +%s` — 儲存 Unix 時間戳，用於計算執行時間
- `rm -f ..._sent` — 移除發送標記，允許新 prompt 觸發通知
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
TIME_FILE="/tmp/claude_session_${SESSION_ID}_time"
SENT_MARKER="/tmp/claude_session_${SESSION_ID}_sent"

# Skip if no prompt file or already sent
if [ ! -f "$PROMPT_FILE" ] || [ -f "$SENT_MARKER" ]; then
    exit 0
fi

# Check elapsed time - only notify if task took more than 10 seconds
START_TIME=$(cat "$TIME_FILE" 2>/dev/null || echo "0")
NOW=$(date +%s)
ELAPSED=$((NOW - START_TIME))

if [ "$ELAPSED" -lt 10 ]; then
    rm -f "$PROMPT_FILE" "$TIME_FILE"
    exit 0
fi

PROMPT=$(cat "$PROMPT_FILE" 2>/dev/null || echo "Task")

# Send notification using terminal-notifier (silent by default)
terminal-notifier -title "Claude Code Finished" -message "$PROMPT"

# Mark as sent and cleanup
touch "$SENT_MARKER"
rm -f "$PROMPT_FILE" "$TIME_FILE"
```

**說明：**
- 檢查是否有 prompt 檔案且尚未發送過通知
- 計算執行時間，只有超過 10 秒才發送通知（避免快速回應打擾）
- 使用 `terminal-notifier` 發送 macOS 通知（預設靜音）
- `touch ..._sent` — 標記已發送，防止重複通知
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

重新啟動 Claude Code 或開啟新 session，輸入任意 prompt。當 Claude 完成回應且執行時間超過 10 秒後，你應該會看到 macOS 通知，顯示你的 prompt 內容（預設為靜音通知，可在「自訂選項」啟用音效）。

---

## 檔案結構總覽

設定完成後，你的 `~/.claude/` 目錄應該包含：

```
~/.claude/
├── settings.json          # Hooks 設定
├── save_prompt.sh         # 儲存 prompt 的 script
├── notify_done.sh         # 發送通知的 script
└── notify_permission.sh   # 權限請求通知的 script（選用）
```

---

## 進階：權限請求時也發送通知

上述設定只會在 Claude **完全結束回應**時發送通知。但當 Claude 請求工具權限時，它處於**等待狀態**而非結束狀態，因此不會觸發 `Stop` hook。

| 狀態            | 觸發的 Hook  | 會發送通知？ |
| --------------- | ------------ | ------------ |
| Claude 完成回應 | `Stop`       | ✅ 是         |
| Claude 等待權限 | `PreToolUse` | ✅ 是（需設定）|

若要在 Claude 請求權限時也收到通知，可以加入 `PreToolUse` hook。

### 建立權限通知 Script

建立 `~/.claude/notify_permission.sh`：

```bash
touch ~/.claude/notify_permission.sh
chmod +x ~/.claude/notify_permission.sh
```

編輯內容：

```bash
#!/bin/bash

# Sends notification when Claude requests tool permission
# Used by PreToolUse hook

INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | jq -r ".tool_name")

terminal-notifier -title "Claude Code Needs Permission" -message "Tool: $TOOL_NAME"
```

### 更新 settings.json

在 `~/.claude/settings.json` 的 `hooks` 區塊中加入 `PreToolUse`：

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
    ],
    "PreToolUse": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "/bin/bash ~/.claude/notify_permission.sh"
          }
        ]
      }
    ]
  }
}
```

這樣當 Claude 請求工具權限時，你也會收到通知。

> **注意**：`PreToolUse` 會在每次工具呼叫前觸發。如果你已設定某些工具為自動允許，這些工具仍會觸發通知但不會等待你確認。你可以使用 `matcher` 來過濾特定工具，例如 `"matcher": "Bash"` 只針對 Bash 指令發送通知。

---

## 自訂選項

### 啟用提示音效

預設為靜音通知。若要加入音效，在 `notify_done.sh` 中加入 `-sound` 參數：

```bash
# 加入音效（在 terminal-notifier 指令後加入）
terminal-notifier -title "Claude Code Finished" -message "$PROMPT" -sound Glass

# 可用的音效名稱
-sound Glass
-sound Ping
-sound Pop
-sound Purr
-sound Submarine
-sound Blow
```

### 調整時間門檻

在 `notify_done.sh` 中修改時間門檻（預設 10 秒）：

```bash
# 改為 30 秒才通知
if [ "$ELAPSED" -lt 30 ]; then

# 改為 0 秒（總是通知）
if [ "$ELAPSED" -lt 0 ]; then
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
