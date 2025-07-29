+++
title = "網頁字型載入效能：<style> 與 <link preload> 的最佳實踐"
description = "探討兩種網頁字型載入方式的效能差異，解析載入順序對 FCP、LCP、CLS 的影響。深入分析 preload 的優勢與注意事項。想提升網頁速度，這篇不可錯過！"
date = "2025-06-02T20:56:52Z"
updated = "2025-07-28T09:38:33.403Z"
draft = false

[taxonomies]
tags = [ ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/ktKNE2aX8F4uJpftfbCDnS?invite=dOLYGeJyZJqVX>"
+++
{% editor_note() %}
這是我在調整 Duckquill template (本站點使用的範本) 時生成的參考資料。Duckquill 原本的做法是方法一，而我將它優化為方法二，在提出 PR 時我需要一個明確的解釋來支撐這個改變。

這是成品 PR: [feat: Optimize head performance and enhance SEO #165 - daudix/duckquill](https://codeberg.org/daudix/duckquill/pulls/165)
{% end %}

{% chat(speaker="jim") %}
請問在網頁效能極致調校技術中，僅使用 `<style>` 來載入字型檔案，以及搭配 `<link>` 來載入字型檔案兩種做法是否有區別? 我知道它們都能夠運作，我想問的是有關於載入順序和效能之類的區別，哪個做法更加好。  
註: 兩個方法的詳細說明如下

1. `<style>` 的方法是載入一個內定義了 `@font-face` 並連結到站台上的 woff2 字型檔。
2. `<link>` 的方法是實作 1 方法的同時，在 html 中加上 `<link as="font" crossorigin="" href="/fonts/Iansui-Regular.woff2" rel="preload" type="font/woff2">` 這種方式來載入該 woff2 字型檔

{% end %}

{% chat(speaker="felo") %}
在網頁效能的極致調校中，對於關鍵字型的載入，結合在 `<head>` 中內嵌 `<style>` 來定義 `@font-face`，並同時使用 `<link rel="preload" as="font">` 來預先載入字型檔案的做法，通常能提供更佳的載入效能。這種組合策略能夠確保瀏覽器盡早發現字型宣告並以高優先級開始下載字型檔案，從而加速首次內容繪製 (FCP) 和最大內容繪製 (LCP)，並有助於減少因字型替換引起的版面配置位移 (CLS)。
{% end %}

## 字型載入機制

在討論兩種方法的差異之前，理解瀏覽器如何處理字型載入至關重要。

### `@font-face` 宣告

`@font-face` CSS 規則用於定義自訂字型的名稱及其來源檔案路徑 [^65][^66]。一個常見的誤解是瀏覽器一遇到 `@font-face` 宣告就會請求字型檔案，但事實並非如此 [^65]。字型檔案僅在頁面中實際有元素使用了該 `font-family` 時才會被下載 [^65][^66]。例如：

```css
@font-face {
  font-family: "MyCustomFont";
  src: url("/fonts/MyCustomFont.woff2") format("woff2");
}

h1 {
  font-family: "MyCustomFont";
}
```

在此範例中，只有當頁面包含 `<h1>` 元素時，`MyCustomFont.woff2` 才會被下載。

### `<link rel="preload" as="font">`

`<link rel="preload" as="font">` 是一種資源提示 (resource hint)，指示瀏覽器提早並以高優先級下載指定的資源，而不阻塞文件的渲染 [^27][^66]。對於字型而言，這意味著瀏覽器可以在解析 CSS 並確定字型是否被實際使用之前就開始下載字型檔案 [^66]。預先載入的字型檔案會更早地可用於頁面渲染。

## 方法一：僅使用 `<style>` 內嵌 `@font-face`

此方法是將 `@font-face` 宣告直接寫在 HTML 文件 `<head>` 內的 `<style>` 標籤中。

### 載入順序與發現時機

- 瀏覽器在解析 HTML `<head>` 時會遇到內嵌的 `<style>` 標籤，從而較早發現 `@font-face` 宣告 [^65][^66]。這比將宣告放在外部 CSS 檔案中能更早被瀏覽器知曉。
- 字型檔案的下載請求僅在瀏覽器解析 CSS 並確認頁面上有元素實際使用此字型後才會發出 [^65]。

### 效能優點

- **早期宣告發現**：相較於外部樣式表，內嵌樣式能讓瀏覽器更快發現字型宣告 [^66]。
- **避免下載未使用字型**：如果頁面最終沒有使用該字型，瀏覽器不會下載字型檔案，從而節省頻寬 [^66]。

### 效能考量

- 儘管字型宣告被早期發現，字型檔案的下載仍需等待 CSSOM (CSS Object Model) 构建完成，且瀏覽器確認該字型被頁面元素所需要之後才會開始。這可能導致字型檔案相對於頁面其他關鍵資源較晚開始下載。

## 方法二：`<style>` 內嵌 `@font-face` 搭配 `<link rel="preload">`

此方法在實施方法一的基礎上，額外在 HTML 的 `<head>` 中加入 `<link rel="preload" as="font" ...>` 來指定相同的 `woff2` 字型檔。使用者提供的範例如下：

```html
<link as="font" crossorigin="" href="/fonts/Iansui-Regular.woff2" rel="preload" type="font/woff2">
```

同時，`<style>` 中定義：

```css
@font-face {
  font-family: 'Iansui'; /* 假設的字型名稱 */
  src: url('/fonts/Iansui-Regular.woff2') format('woff2');
  /* 其他 font-display 等屬性 */
}
```

### 載入順序與發現時機（方法二）

- `@font-face` 宣告同樣透過內嵌 `<style>` 被早期發現。
- `<link rel="preload">` 指令使瀏覽器能夠在頁面載入的早期階段就發現字型資源，並立即以高優先級開始下載，無需等待樣式表下載和解析完成，也不論該字型是否已被頁面實際需要 [^66][^96]。
- 預先載入字型時，必須包含 `crossorigin` 屬性，因為字型被視為CORS資源，即使是同源字型也是如此 [^66]。

### 效能優點（方法二）

- **極早期字型檔案下載**：`preload` 確保關鍵字型檔案的下載請求非常早就被觸發，通常與其他關鍵資源並行下載 [^66]。
- **改善 FCP/LCP**：字型檔案更早可用，可以縮短文字不可見 (FOIT) 或使用後備字型 (FOUT) 的時間，對於依賴此字型的文字內容，能有效提升首次內容繪製 (FCP) 和最大內容繪製 (LCP) 的速度 [^65]。
- **減少 CLS**：如果字型能足夠早地載入，可以避免或減少因字型切換（從後備字型換到網頁字型）導致的版面配置位移 (Cumulative Layout Shift, CLS) [^65]。
- **確保關鍵字型可用性**：對於影響「首屏」(above-the-fold) 內容渲染的關鍵字型，`preload` 尤其有效 [^65]。

### 效能考量（方法二）

- **資源競爭**：`preload` 會指示瀏覽器以高優先級下載資源，這可能會從其他關鍵資源的載入中占用頻寬和處理器時間 [^65]。因此，應僅對確實關鍵的字型使用 `preload`。
- **可能下載未使用字型**：由於 `preload` 不等待確認字型是否實際被使用就開始下載 [^66]，如果預先載入的字型最終未在頁面中使用，會造成頻寬浪費。
- **繞過部分內容協商**：`preload` 會忽略如 `unicode-range` 等 CSS 描述符，它會下載指定的整個字型檔案 [^65]。因此，審慎使用時，應只用於載入單一、確切需要的字型格式 [^65]。

## 比較與建議

兩種方法的核心區別在於**字型檔案下載的觸發時機和優先級**。

- **僅使用 `<style>` 內嵌 `@font-face`**：瀏覽器早早知道有這個字型定義，但只有在頁面實際用到它時才開始下載。
- **`<style>` 內嵌 `@font-face` 搭配 `<link rel="preload">`**：瀏覽器不僅早早知道定義，還被指示立即以高優先級下載字型檔案，使其能更快地應用於頁面渲染。

對於追求極致效能，特別是針對影響初始渲染和核心網頁指標 (Core Web Vitals) 的關鍵字型，**方法二（內嵌 `<style>` + `preload`）通常是更優的選擇**。

- 內嵌 `@font-face` 確保了宣告的早期發現，而 `preload` 則確保了字型檔案本身的早期下載 [^66]。
- 研究指出，預先載入最重要的字型可以非常有效地改善載入效能 [^65]。一項測試顯示，預先載入網頁字型並結合內嵌關鍵 CSS（包含字型宣告）可以顯著提升頁面載入速度指標 [^96]。
- 使用 `preload` 時，應確保僅用於頁面初始視圖渲染所必需的字型，並正確使用 `crossorigin` 屬性。

## 結論

為了最大化網頁字型的載入效能，特別是對於那些對使用者體驗至關重要的關鍵字型（例如用於標題或主要內容的字型），結合在 `<head>` 中內嵌 `<style>` 以定義 `@font-face`，並同時使用 `<link rel="preload" as="font">` 預先載入相應的 `woff2` 字型檔案，是當前推薦的更佳實踐。這種方法透過提早發現字型宣告和以高優先級下載字型檔案，有效地縮短了字型可用時間，從而對 FCP、LCP 和 CLS 等核心網頁指標產生正面影響。然而，`preload` 的使用應具有策略性，僅限於真正關鍵的字型，以避免不必要的資源競爭和頻寬浪費。

[^27]: [預先載入模組| Articles - web.dev](https://web.dev/articles/modulepreload?hl=zh-tw)
[^65]: [Best practices for fonts | Articles | web.dev](https://web.dev/articles/font-best-practices)
[^66]: [Optimize web fonts | web.dev](https://web.dev/learn/performance/optimize-web-fonts)
[^96]: [Benchmarking Web-font Loading | Oliver Roick](https://oliverroick.net/writing/2021/benchmarking-web-font-loading.html)
