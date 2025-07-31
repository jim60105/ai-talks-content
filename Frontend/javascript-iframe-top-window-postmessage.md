+++
title = "JavaScript 跨窗溝通：iframe 與 top window 雙向傳訊實作"
description = "想知道如何讓 iframe 與 top window 之間安全地互相傳遞訊息並執行程式碼嗎？本文將帶你了解 postMessage 的正確用法與安全注意事項。掌握這些技巧，讓你的前端開發更靈活！"
date = "2025-05-02T18:50:29Z"
updated = "2025-07-27T15:25:38Z"
draft = false

[taxonomies]
tags = [ "JavaScript" ]
providers = [ "Felo Search" ]

[extra]
withAI = "<https://felo.ai/search/ThTqBh8NdjkThJg7gdkvS7?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
js 在 iframe 和 top window 中 js 如何發送通訊，讓 top 執行一段程式碼再回傳結果?
{% end %}

{% chat(speaker="felo") %}
要在 JavaScript 中實現 iframe 和 top window 之間的通訊，通常使用 `postMessage` API。這個方法允許不同來源的窗口之間安全地傳遞消息。以下是如何實現這一功能的詳細步驟：
{% end %}

## **步驟一：從 iframe 發送消息到 top window**

在 iframe 中，你可以使用 `window.parent.postMessage` 方法來發送消息到父窗口（top window）。以下是一個簡單的範例：

```javascript
// 在 iframe 中發送消息
const message = JSON.stringify({
    command: 'executeFunction',
    data: { /* 你想傳遞的數據 */ }
});
window.parent.postMessage(message, '*'); // '*' 表示接受來自任何來源的消息
```

## **步驟二：在 top window 中接收消息**

在 top window 中，你需要設置一個事件監聽器來接收來自 iframe 的消息。這樣可以根據接收到的消息執行相應的操作：

```javascript
// 在 top window 中接收消息
window.addEventListener('message', function(event) {
    // 確認消息來源的安全性
    if (event.origin !== 'https://你的iframe來源.com') {
        return; // 忽略不安全的消息
    }

    const data = JSON.parse(event.data);
    if (data.command === 'executeFunction') {
        // 執行相應的函數並處理數據
        const result = yourFunction(data.data);
        
        // 將結果回傳給 iframe
        event.source.postMessage(JSON.stringify({ result }), event.origin);
    }
});
```

## **步驟三：從 top window 回傳結果到 iframe**

在 top window 中執行完相應的函數後，可以將結果回傳給 iframe。這可以通過 `event.source.postMessage` 方法來實現，如上面的範例所示。

## **步驟四：在 iframe 中接收結果**

在 iframe 中，你也需要設置一個事件監聽器來接收來自 top window 的結果：

```javascript
// 在 iframe 中接收結果
window.addEventListener('message', function(event) {
    // 確認消息來源的安全性
    if (event.origin !== 'https://你的top window來源.com') {
        return; // 忽略不安全的消息
    }

    const result = JSON.parse(event.data).result;
    console.log('Received result from top window:', result);
});
```

## **注意事項**

1. **安全性**：在使用 `postMessage` 時，務必檢查消息的來源（`event.origin`），以防止潛在的安全風險。

2. **同源政策**：如果 iframe 和 top window 不在同一個域名下，則必須使用 `postMessage` 進行通訊，因為直接訪問對方的 DOM 是不被允許的。

3. **數據格式**：通常使用 JSON 格式來傳遞數據，這樣可以方便地傳遞複雜的數據結構。

通過以上步驟，你可以實現 iframe 和 top window 之間的有效通訊，並在需要時執行相應的程式碼。
