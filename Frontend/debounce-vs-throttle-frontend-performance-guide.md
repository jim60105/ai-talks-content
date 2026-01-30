+++
title = "Debounce vs Throttle（防抖 / 節流）比較：原理、差異與常見應用說明"
description = "比較 Debounce（防抖）與 Throttle（節流），說明兩者的原理、主要差異與適用情境，並提供實務建議以協助選擇適合的方案。"
date = "2026-01-30T03:50:58Z"
updated = "2026-01-30T07:01:33.564Z"
draft = false

[taxonomies]
tags = [ "JavaScript" ]
providers = [ "Felo Search" ]

[extra]
withAI = "<https://felo.ai/search/EHdoweS5MPdUfe8KAqFAfB?invite=dOLYGeJyZJqVX>"
+++
{% alert(edit=true) %}
我又犯了一次需要 Throttle 卻實作成 Debounce 的錯誤，這篇文章來釐清一下兩者差異 😅
{% end %}

{% chat(speaker="jim") %}
debounce vs throttle
{% end %}

{% chat(speaker="felo") %}
在前端開發中，當處理使用者輸入或瀏覽器事件等會頻繁觸發的事件時，為了優化效能、避免不必要的處理或伺服器請求，會使用兩種常見的技術：**debounce**（防抖動）和 **throttle**（節流）。這兩種技術都可以限制程式碼的呼叫頻率，但其工作原理和應用場景有所不同。
{% end %}

## Debounce（防抖動）

**Debounce** 是一種延遲函式執行的技術，它會等待直到使用者停止執行某個動作一段指定時間後，才會呼叫該函式[^1][^4]。這意味著，如果在設定的延遲期間內事件再次被觸發，之前的計時器會被取消，並重新啟動一個新的計時器，確保函式只在事件「穩定」下來之後才被呼叫一次[^4]。

* **工作原理**：
    每當事件被觸發時，`debounce` 會先清除前一個計時器（如果有的話），然後設定一個新的計時器。只有當這個計時器在指定延遲時間內沒有被重新設定時，關聯的函式才會被執行[^3][^4]。

* **舉例**：
  * 搜尋列建議：當使用者在搜尋框中輸入文字時，不希望每按一次鍵就向後端發送請求。使用 `debounce` 可以確保只有在使用者停止輸入一段時間後，才發送一次包含完整搜尋值的請求，從而減少不必要的 API 呼叫[^1][^3][^4]。例如，輸入 "React"，若沒有防抖動，會發送 "R", "Re", "Rea", "Reac", "React" 五次請求；使用防抖動後，只會發送一次 "React" 的請求[^3][^5]。
  * 表單驗證[^4]。
  * 視窗大小調整事件：只在使用者停止調整視窗大小後才觸發更新介面佈局的程式碼[^1][^4]。

## Throttle（節流）

**Throttle** 是一種限制函式執行頻率的技術，它確保函式在指定的時間間隔內最多只會被呼叫一次，無論事件在該間隔內被觸發多少次[^4][^5][^11][^12]。

* **工作原理**：
    `throttle` 會設定一個時間間隔。當事件首次觸發時，函式會立即執行。在接下來的指定時間間隔內，即使事件再次被觸發，函式也不會執行。直到該時間間隔結束，函式才能再次被執行[^4][^5]。可以將其理解為將快速連續觸發的事件佇列起來，每隔一段時間處理佇列中的一個項目[^6]。

* **舉例**：
  * 滾動事件（無限捲動）：當使用者滾動頁面時，可以使用 `throttle` 來控制載入更多內容的函式呼叫頻率，避免因過於頻繁的觸發而導致效能問題[^4][^5]。
  * 自動儲存功能：在編輯器中，當使用者快速輸入時，使用 `throttle` 可以定期（例如每隔幾秒）觸發儲存函式，即使發生故障，也只會遺失最近的輸入[^3]。
  * 遊戲中的射擊按鈕：限制在短時間內無法連續射擊多次。

## 主要差異

`debounce` 和 `throttle` 的主要區別在於：

* **執行時機**：`debounce` 在一系列事件結束並經過一段冷卻時間後才執行函式[^1][^2]。而 `throttle` 則以固定的時間間隔執行函式，無論事件在該間隔內觸發多少次[^1][^4][^5]。
* **關注點**：`debounce` 關注的是事件停止後的「最終狀態」，確保函式只在最後一次事件觸發後才執行[^8]。`throttle` 關注的是「執行頻率」，確保函式在一定時間內不會被呼叫超過一次[^4][^11]。
* **延遲**：`debounce` 會顯著延遲函式的執行，直到使用者動作停止[^5]。`throttle` 則以規律的間隔執行回呼函式，不會長時間延遲[^5]。

## 何時使用

* **使用 `debounce` 的情況**：
    當你只關心一系列事件中的「最後一個」事件，並且希望在事件停止觸發一段時間後才執行操作時，`debounce` 是最佳選擇[^4][^8]。例如，搜尋輸入框、表單驗證、或只需在視窗調整大小完成後才觸發一次的操作[^4][^8]。

* **使用 `throttle` 的情況**：
    當你需要確保函式以固定的速率或間隔持續執行，即使事件被高頻率觸發時，`throttle` 是最佳選擇[^4]。例如，無限捲動、遊戲中的射擊按鈕、監聽滑鼠移動、或自動儲存功能等需要定期更新的操作[^4][^5]。

[^1]: [Debounce and Throttling: What They Are and When to Use ...](https://medium.com/@bs903944/debounce-and-throttling-what-they-are-and-when-to-use-them-eadd272fe0be)
[^2]: [Debounce - Glossary - MDN Web Docs](https://developer.mozilla.org/en-US/docs/Glossary/Debounce)
[^3]: [How to debounce and throttle in React without losing your ...](https://www.developerway.com/posts/debouncing-in-react)
[^4]: [javascript - Debounce vs Throttle Explained](https://dev.to/nilebits/javascript-performance-optimization-debounce-vs-throttle-explained-5768)
[^5]: [JavaScript Debounce vs. Throttle](https://www.syncfusion.com/blogs/post/javascript-debounce-vs-throttle)
[^6]: [Difference between debounce and throttle : r/javascript](https://www.reddit.com/r/javascript/comments/73vkl2/difference_between_debounce_and_throttle/)
[^8]: [Debounce vs Throttle: Definitive Visual Guide](https://kettanaito.com/blog/debounce-vs-throttle)
[^11]: [The Difference Between Throttling and Debouncing](https://css-tricks.com/the-difference-between-throttling-and-debouncing/)
[^12]: [Difference between Debouncing and Throttling](https://www.geeksforgeeks.org/javascript/difference-between-debouncing-and-throttling/)
