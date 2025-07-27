+++
title = "如何判斷 JavaScript 是否運行於 iframe？"
description = "想知道你的 JavaScript 腳本是在 iframe 還是最上層頁面運行嗎？本文介紹三種常見判斷方法，並附上實用範例程式碼。掌握這些技巧，讓你的網頁開發更靈活！"
date = "2025-05-01T00:00:00Z"
updated = "2025-07-27T15:15:56Z"
draft = false

[taxonomies]
tags = [ "JavaScript" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/cBqDGLJbMNt4D9VVGKrAqr?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
js how to know if this script is running inside iframe or on the top most page?
{% end %}

{% chat(speaker="felo") %}
要判斷一段 JavaScript 腳本是否在 iframe 中運行，或者是否在最上層的頁面中，可以使用幾種方法。以下是一些常見的檢查方式：
{% end %}

## **檢查方法**

1. **使用 `window.top` 和 `window.self`**：
   - `window.top` 代表最上層的窗口，而 `window.self` 代表當前窗口。如果這兩者不相等，則表示當前頁面在 iframe 中運行。

   ```javascript
   function isInIframe() {
       return window.self !== window.top;
   }
   ```

2. **使用 `window.frameElement`**：
   - `window.frameElement` 會返回當前窗口的 iframe 元素。如果返回值為 `null`，則表示當前頁面不在任何 iframe 中。

   ```javascript
   function isInIframe() {
       return window.frameElement !== null;
   }
   ```

3. **比較 `document.referrer`**：
   - 檢查 `document.referrer` 可以幫助判斷頁面是直接加載還是通過其他頁面（如 iframe）加載。如果 `document.referrer` 不為空，則可能表示頁面是在 iframe 中加載的。

   ```javascript
   function isInIframe() {
       return document.referrer !== '';
   }
   ```

## **示例代碼**

以下是一個綜合示例，展示如何使用上述方法來檢查當前頁面是否在 iframe 中運行：

```javascript
function checkIframe() {
    if (window.self !== window.top) {
        console.log("這個頁面在 iframe 中運行");
    } else {
        console.log("這個頁面在最上層的頁面中運行");
    }
}

checkIframe();
```

這段代碼會在控制台輸出當前頁面的運行狀態。

## **結論**

使用上述方法可以有效地判斷 JavaScript 腳本是否在 iframe 中運行或在最上層頁面中運行。根據具體需求，可以選擇最適合的檢查方式來實現功能。
