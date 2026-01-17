+++
draft = false
date = 2026-01-17T21:30:00+08:00
title = "如何即時監控你的 Claude Code 用量"
tags = []
categories = ['ClaudeCode']
+++

使用 Claude Code 時，是否曾突然達到用量上限而措手不及？Claude Code Usage Monitor 能即時監控你的 token 消耗和費用，讓你提前掌握使用狀況。

## 快速安裝

推薦使用 [uv](https://github.com/astral-sh/uv) 安裝：

```bash
uv tool install claude-monitor
```

如果還沒安裝 uv，先執行：

```bash
# Linux/macOS
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

安裝後請重新開啟終端機。

## 使用方式

```bash
# 啟動監控
claude-monitor

# 短指令
ccm
```

指定訂閱方案：

```bash
claude-monitor --plan pro    # Pro 方案
claude-monitor --plan max5   # Max5 方案
claude-monitor --plan max20  # Max20 方案
```

## 相關連結

- [PyPI - claude-monitor](https://pypi.org/project/claude-monitor/)
- [GitHub 原始碼](https://github.com/anthropics/claude-code-usage-monitor)
