+++
title = "主流 AI 模型價格全面比較：GPT-4.1、Claude 3.7 等"
description = "想了解 GPT-4.1、Claude Sonnet 3.7、GPT-4.1-mini 與 Claude 3.5 Haiku 的最新價格嗎？本文以表格清楚比較各模型的輸入與輸出費用。掌握最新 AI 模型定價，選擇最適合你的解決方案！"
date = "2025-04-24T10:06:03Z"
updated = "2025-07-27T14:18:23.084Z"
draft = false

[taxonomies]
tags = [ "LLM" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/NjLh7c46LKJwnYkHyoN5Hf?invite=dOLYGeJyZJqVX>"
+++
{% alert(edit=true) %}
這篇想展示的是 Felo Search 擁有很好的網頁查詢能力，可以基於多個網路來源整理資料。
{% end %}

{% chat(speaker="jim") %}
比較 openai gpt-4.1, Claude Sonnet 3.7, gpt-4.1-mini, claude 3.5 haiku 的 pricing, 以表格呈現. 注意單位需要換算為相同單位。
{% end %}

{% chat(speaker="felo") %}
根據搜尋結果，以下是 OpenAI 和 Anthropic 主要 AI 模型的價格比較表格：
{% end %}

## 模型價格比較表

| 模型名稱 | 輸入價格 (每百萬 tokens) | 輸出價格 (每百萬 tokens) | 備註 |
|---------|----------------------|----------------------|------|
| GPT-4.1 | $2.00 | $8.00 | 具有 100 萬 token 上下文窗口，專注於編碼能力[^1][^7] |
| Claude 3.7 Sonnet | $3.00 | $15.00 | 包含思考 tokens 的費用，支援延伸思考模式[^8][^9] |
| GPT-4.1 mini | $0.40 | $1.60 | 比 GPT-4.1 更高效但準確性較低[^1][^12] |
| Claude 3.5 Haiku | $1.00 | $5.00 | 價格於 2024 年 11 月調整，比前代提高 4 倍[^4][^6] |

## 額外資訊

- GPT-4.1 比 GPT-4o 便宜約 26%，GPT-4o 的價格為每百萬輸入 tokens $2.50，每百萬輸出 tokens $10.00[^7][^34]
- Claude 3.7 Sonnet 提供延伸思考模式，但價格與標準模式相同[^9]
- GPT-4.1 系列還包含 GPT-4.1 nano，是 OpenAI 目前最便宜和最快的模型，每百萬輸入 tokens $0.10，每百萬輸出 tokens $0.40[^1][^12]
- Claude 3.5 Haiku 原始價格為每百萬輸入 tokens $0.25，每百萬輸出 tokens $1.25，後來調整為每百萬輸入 tokens $1.00，每百萬輸出 tokens $5.00[^4][^15]
- 部分模型提供快取折扣，例如 GPT-4.1 提供 75% 的提示快取折扣[^39]，而 Claude 3.7 Sonnet 提供高達 90% 的提示快取成本節省[^8]

這些價格資訊截至 2025 年 4 月的最新數據，實際使用時建議參考官方網站的最新定價。

[^1]: [OpenAI's new GPT-4.1 AI models focus on coding | TechCrunch](https://techcrunch.com/2025/04/14/openais-new-gpt-4-1-models-focus-on-coding/)
[^4]: [Anthropic hikes the price of its Haiku model - TechCrunch](https://techcrunch.com/2024/11/04/anthropic-hikes-the-price-of-its-haiku-model/)
[^6]: [Anthropic hikes the price of its Haiku model - Yahoo Finance](https://ca.finance.yahoo.com/news/anthropic-hikes-price-haiku-model-182408114.html)
[^7]: [GPT-4o vs GPT-4.1 - Detailed Performance & Feature ...](https://docsbot.ai/models/compare/gpt-4o/gpt-4-1)
[^8]: [Claude 3.7 Sonnet - Anthropic](https://www.anthropic.com/claude/sonnet)
[^9]: [Claude 3.7 Sonnet and Claude Code - Anthropic](https://www.anthropic.com/news/claude-3-7-sonnet)
[^12]: [Model - OpenAI API](https://platform.openai.com/docs/models/gpt-4.1-nano)
[^15]: [Haiku 3.5 released! : r/ClaudeAI - Reddit](https://www.reddit.com/r/ClaudeAI/comments/1gjl6yl/haiku_35_released/)
[^34]: [Model - OpenAI API](https://platform.openai.com/docs/models/gpt-4.1)
[^39]: [Introducing GPT-4.1 in the API - OpenAI](https://openai.com/index/gpt-4-1/)
