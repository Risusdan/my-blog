+++
draft = false
date = 2026-01-17T21:30:00+08:00
title = "如何即時監控你的 Claude Code 用量"
tags = []
categories = ['ClaudeCode']
+++

## 前言

使用 Claude Code 進行開發時，你是否曾經遇過這樣的情況：正在專注寫程式，突然收到訊息說已達到用量上限？這種情況不僅打斷工作流程，還讓人措手不及。

Claude Code Usage Monitor 就是為了解決這個問題而生的工具。它能即時監控你的 token 消耗、費用支出和訊息數量，讓你隨時掌握使用狀況，提前做好規劃。

## 功能特色

| 功能       | 說明                            |
| ---------- | ------------------------------- |
| 即時監控   | 在終端機中即時顯示用量資訊      |
| Token 追蹤 | 追蹤 input/output tokens 消耗量 |
| 費用計算   | 自動計算目前花費金額            |
| P90 預測   | 使用機器學習預測 session 上限   |
| 彩色進度條 | 視覺化顯示用量百分比            |
| 多級警告   | 根據用量提供不同等級的警示      |

## 運作原理

Claude Code 會將使用資料儲存在 `~/.claude/projects/` 目錄下，以 JSONL 格式記錄每一次的請求。每筆記錄包含：

```json
{
  "timestamp": "2026-01-17T10:30:00Z",
  "sessionId": "abc123",
  "model": "claude-sonnet-4-20250514",
  "usage": {
    "input_tokens": 1500,
    "output_tokens": 800,
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 500
  },
  "requestId": "req_xyz",
  "message": {
    "id": "msg_123"
  }
}
```

Claude Code Usage Monitor 會讀取這些 JSONL 檔案，即時解析並彙整用量資訊，然後在終端機中以友善的介面呈現。

## 安裝步驟

### 推薦方式：使用 uv

[uv](https://github.com/astral-sh/uv) 是一個快速的 Python 套件管理工具，推薦使用它來安裝：

```bash
uv tool install claude-monitor
```

安裝完成後，直接執行：

```bash
claude-monitor
```

### 替代方式

使用 pip：

```bash
pip install claude-monitor
```

使用 pipx（隔離環境安裝）：

```bash
pipx install claude-monitor
```

## 基本使用

### 啟動監控

```bash
# 自動偵測方案（預設）
claude-monitor

# 使用短指令
cmonitor
# 或
ccm
```

### 指定訂閱方案

```bash
# Pro 方案
claude-monitor --plan pro

# Max5 方案
claude-monitor --plan max5

# Max20 方案
claude-monitor --plan max20
```

### 切換檢視模式

```bash
# 即時監控（預設）
claude-monitor --view realtime

# 每日統計
claude-monitor --view daily

# 每月統計
claude-monitor --view monthly
```

## 方案選擇

不同訂閱方案有不同的用量上限，以下是各方案的限制：

| 方案   | Token 上限     | 費用上限 | 訊息上限 |
| ------ | -------------- | -------- | -------- |
| Pro    | 19,000         | $18.00   | 250      |
| Max5   | 88,000         | $35.00   | 1,000    |
| Max20  | 220,000        | $140.00  | 2,000    |
| Custom | 自動偵測 (P90) | $50.00   | 250      |

如果不確定自己的方案，可以使用預設的自動偵測功能，工具會根據歷史使用資料透過 P90 演算法推測你的上限。

## 進階設定

### 時區設定

預設使用系統時區，也可以手動指定：

```bash
claude-monitor --timezone "Asia/Taipei"
```

### 更新頻率

調整監控畫面的刷新速度（單位：秒）：

```bash
claude-monitor --refresh 2
```

### 主題設定

```bash
# 深色主題（預設）
claude-monitor --theme dark

# 淺色主題
claude-monitor --theme light
```

### 組合使用

參數可以組合使用：

```bash
claude-monitor --plan max5 --view daily --timezone "Asia/Taipei" --refresh 3
```

## 實際畫面

執行 `claude-monitor` 後，終端機會顯示一個即時更新的監控面板，包含：

- **標題列**：顯示目前選擇的方案和時區
- **Token 用量**：以進度條顯示 input/output tokens 消耗比例
- **費用統計**：目前已花費金額和預估上限
- **訊息計數**：已使用的訊息數量
- **Session 資訊**：目前 session 的開始時間和持續時間
- **警告區域**：當用量接近上限時會顯示彩色警告

進度條會根據用量百分比變換顏色：
- 🟢 綠色：用量低於 50%
- 🟡 黃色：用量介於 50% - 80%
- 🟠 橘色：用量介於 80% - 95%
- 🔴 紅色：用量超過 95%

## 小結

Claude Code Usage Monitor 是一個簡單實用的監控工具，能幫助你：

1. **掌握用量**：即時了解 token 消耗情況
2. **預估花費**：清楚知道目前的費用支出
3. **避免中斷**：在接近上限前提早收到警告
4. **優化使用**：透過每日/每月統計分析使用模式

安裝只需一行指令，使用也非常直覺。如果你是 Claude Code 的重度使用者，這個工具絕對值得一試。

### 相關連結

- 專案首頁：[PyPI - claude-monitor](https://pypi.org/project/claude-monitor/)
- 原始碼：[GitHub](https://github.com/anthropics/claude-code-usage-monitor)
