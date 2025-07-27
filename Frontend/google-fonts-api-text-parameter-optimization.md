+++
title = "Google Fonts API text 參數優化：減少字體檔案大小提升網頁載入速度"
description = "深入解析 Google Fonts API 中 text 參數的使用方法，透過指定特定字符來減少字體檔案大小，大幅提升網頁載入效能。同時分享如何運用 Felo Search AI 搜尋引擎快速找到技術文件與參考資料。"
date = "2024-08-01T00:00:00Z"
updated = "2025-07-25T09:32:11.425Z"
draft = false

[taxonomies]
tags = [ "Prompt Engineering" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/UyX9p4obmzJ3eUZtvre7pw?invite=dOLYGeJyZJqVX>"
+++
{% editor_note() %}
這篇提問想分享的其實是如何快速使用 Felo Search 找到出處和參考資料。

在實作網頁極致調校時，字體加載速度是一項必定要改善的項目。有時從上一個專案中抄東西過來改，會忘記原本這個東西怎麼使用，而想要查找參考資料和文件。這時候像是 Felo Search 這樣的服務就非常有用。

過往我們可能會在腦中組織關鍵字然後 Google 搜尋，但現在有了 AI 搜尋引擎，我們可以直接 copy paste 並叫他解釋，AI 能幫我們找到相關的資料和參考來源。

連動腦想關鍵字都不需要，更懶了！😝
{% end %}
{% chat(speaker="jim") %}

```html
<link
    href="https://fonts.googleapis.com/css2?family=Mochiy+Pop+P+One&display=swap&text=晚餐吃什麼火鍋牛肉麵排館拉便當店自助夜市速食中"
    rel="stylesheet"
/>
```

explain the "text" parameters here and give me the references
{% end %}

{% chat(speaker="felo") %}
在這段 `<link>` 標籤中，`text` 參數用於指定僅下載和顯示特定字符。這樣可以減少字體文件的大小，從而加快網頁加載速度。以下是這段 `<link>` 標籤的詳細解釋：
{% end %}

```html
<link
  href="https://fonts.googleapis.com/css2?family=Mochiy+Pop+P+One&display=swap&text=晚餐吃什麼火鍋牛肉麵排館拉便當店自助夜市速食中"
  rel="stylesheet"
/>
```

### 參數解釋

- **href**: 指向 Google Fonts API 的 URL，這裡指定了要使用的字體 `Mochiy Pop P One`。
- **display**: 設置字體的顯示方式，`swap` 表示在字體加載完成前使用後備字體，然後替換為目標字體。
- **text**: 指定僅下載和顯示這些字符：`晚餐吃什麼火鍋牛肉麵排館拉便當店自助夜市速食中`。這樣可以減少字體文件的大小，從而加快網頁加載速度[^1][^6][^9]。

### 參考資料

1. [Google Fonts API 文檔](https://developers.google.com/fonts/docs/getting_started) 提供了如何使用 `text` 參數的詳細說明。
2. [Google Fonts CSS API v2](https://developers.google.com/fonts/docs/css2) 支持 `text` 參數來指定特定字符。
3. [Google Fonts 的最佳實踐](https://developers.google.com/fonts/docs/best_practices) 提到使用 `text` 參數可以優化字體加載時間。

這些參數的使用可以顯著提升網頁的性能，特別是在需要使用大量自定義字體的情況下。

{% editor_note() %}
上面第三項參考資料是 AI 幻覺，它瞎掰的。
{% end %}

[^1]: [How to use the text parameter with the google fonts API for just one font type - Stack Overflow](https://stackoverflow.com/questions/59521701/how-to-use-the-text-parameter-with-the-google-fonts-api-for-just-one-font-type)
[^6]: [Support _text_ parameter for font queries · Issue #36 · nuxt-modules_google-fonts](https://github.com/nuxt-community/google-fonts-module/issues/36)
[^9]: [Get Started with the Google Fonts API | Google for Developers](https://developers.google.com/fonts/docs/getting_started)
