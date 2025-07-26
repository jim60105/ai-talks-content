+++
title = "Windows 與 Linux 檔案名稱與網址長度限制詳解：全型中文字計算方式"
description = "深入解析 Windows 與 Linux 系統下檔案名稱與網址長度限制，並說明全型中文字在不同情境下的計算方式與程式設計建議。"
date = "2025-02-01T00:00:00Z"
updated = "2025-07-26T21:08:35.386Z"
draft = false

[taxonomies]
tags = [ "Linux" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/KEr5tsZwrixxPkB3oVDZwy>"
+++
{% chat(speaker="jim") %}
以正體中文詳細解說「檔案名稱」以及「網址」的長度限制在 Windows 和 Linux 各是多長，程式設計時以什麼長度為標準為適當，並說明全型中文字的情況時，在各個情況下字數限制如何計算。
{% end %}

{% chat(speaker="felo") %}
根據搜尋結果，我將詳細分析相關問題。
{% end %}

## 檔案名稱長度限制與程式設計標準

### **Windows 系統規範**

#### 檔案名稱限制

- **傳統路徑限制**：MAX_PATH 限制為 260 字元（含磁碟機代號、冒號、反斜線及終止空字元），例如 `D:\...<NUL>` 總長度不可超過此值[^1][^4]。
- **單一元件限制**：路徑中的目錄或檔案名稱單一元件最長 255 字元（例如 `folderile.txt` 的 `file.txt` 部分）[^1]。
- **長路徑支援**：Windows 10 1607 後可啟用超過 260 字元的路徑，需同時滿足兩條件：
  - 登錄檔 `LongPathsEnabled` 設為 1
  - 應用程式資訊清單加入 `<longPathAware>` 標籤[^1]。
- **特殊前綴**：使用 `\?\` 前綴可繞過 MAX_PATH 限制，並支援最長 32,767 字元的擴充路徑（需使用 Unicode API）[^1][^7]。

#### 全型中文字處理

- **字元計算**：全型字元（如中文）在路徑長度限制中計為「1 個字元」，與半型字元相同[^4]。
- **位元組佔用**：儲存時 UTF-16 編碼下，每個全型字元佔 2 位元組；若程式需處理位元組層級（如網路傳輸），需注意實際儲存長度。

---

### **Linux 系統規範**

#### 檔案名稱限制

- **單一檔案名**：主流檔案系統（如 ext4）限制為 **255 位元組**，UTF-8 編碼下全型中文字可能佔 3 位元組，因此實際可輸入字元數較少[^3][^7]。
- **完整路徑**：PATH_MAX 限制為 4,096 位元組（含終止空字元），可透過 `getconf PATH_MAX` 查詢[^3][^7]。
- **檔案系統差異**：
  - XFS：預設 255 字元，可擴充
  - Btrfs：依配置支援更長名稱
  - ReiserFS：支援至 4,096 字元[^7]。

#### 全型中文字處理

- **位元組優先**：限制基於位元組而非字元，例如 255 位元組上限在 UTF-8 編碼下，全型中文字最多可輸入約 85 字元（255 ÷ 3）[^3][^7]。
- **編碼影響**：使用 `TextEncoder` 計算字串位元組數時，需注意不同語言字元的編碼差異[^10]。

---

## 網址長度限制與設計建議

### **跨系統共通限制**

- **HTTP 協定**：未明定 URL 長度上限，但實務受限於瀏覽器與伺服器[^6][^9]。
- **瀏覽器限制**：
  - IE/Edge：2,083 字元（含路徑 2,048 字元）
  - Chrome：32,779 字元
  - Firefox：64,000+ 字元[^6][^9]。
- **伺服器限制**：
  - IIS：預設 4,096 位元組，可調整至 16,384 字元
  - Apache：8,177 字元
  - Nginx：4,096 字元（預設）[^6][^9]。

### **全型字元處理**

- **URL 編碼**：全型字元經百分比編碼（Percent-Encoding）後，單一字元可能擴充至 9 位元組（如 UTF-8 中文轉為 `%E4%B8%AD`）[^6][^9]。
- **設計建議**：
  - 保守值：保持在 2,083 字元內以相容所有瀏覽器
  - 替代方案：過長資料改用 POST 傳輸或 URL fragment 儲存前端狀態[^6][^9]。

---

## 程式設計標準與最佳實踐

### **檔案名稱處理**

1. **跨平台設計**：
   - Windows：單一元件 ≤255 字元，完整路徑 ≤260 字元（未啟用長路徑時）
   - Linux：單一檔案名 ≤255 位元組，路徑 ≤4,096 位元組[^1][^3][^7]。
2. **全型字元計算**：
   - 顯示層級：以字元數為基準（1 全型字元 = 1 字元）
   - 儲存層級：UTF-8 編碼需預留 3 倍位元組空間[^4][^10]。
3. **路徑建構工具**：
   - 使用 `pathlib`（Python）或 `Path`（C#）自動處理系統差異
   - 即時驗證：在輸入階段攔截超長名稱（如限制輸入框字數）[^10]。

### **網址處理**

1. **資料傳輸**：
   - GET 參數：總長度 ≤2,000 字元（含編碼擴充）
   - 檔案下載：避免將檔案路徑直接暴露於 URL[^6][^9]。
2. **編碼轉換**：
   - 統一使用 `encodeURIComponent`（JavaScript）或 `UrlEncode`（C#）處理特殊字元
   - 中文字符預留 9 倍位元組長度[^6][^9]。

### **範例程式碼（JavaScript）**

```javascript
// 計算 UTF-8 位元組數
function countBytes(text) {
  return new TextEncoder().encode(text).length;
}

// 裁切超長字串（保留完整字元）
function truncateByBytes(text, maxBytes) {
  const encoder = new TextEncoder();
  const decoder = new TextDecoder();
  const bytes = encoder.encode(text).slice(0, maxBytes);
  return decoder.decode(bytes).replace(/\uFFFD/g, '');
}
```

此方案可同時處理中英文混雜情境，確保裁切後不產生亂碼[^10]。

{% editor_note() %}
一言以蔽之：≤255 bytes  
若是單純以字元數來計算，由於中文字元可以佔三個 bytes  
保險起見總路徑長度不要超過 85 字元。
{% end %}

[^1]: [Count length of value within a cell with full-width characters](https://stackoverflow.com/questions/68412744/count-length-of-value-within-a-cell-with-full-width-characters)
[^3]: ["Linux has a 255 bytes limitation for file names (this translates ...](https://news.ycombinator.com/item?id=19242579)
[^4]: [為Dropbox 檔案與資料夾命名](https://help.dropbox.com/zh-tw/organize/file-names)
[^6]: [整理：各个浏览器及web服务器对URL（get）长度的限制](https://www.cnblogs.com/chinasoft/p/17417694.html)
[^7]: [OneFS and Long Filenames | Dell Technologies Info Hub](https://infohub.delltechnologies.com/p/onefs-and-long-filenames/)
[^9]: [URL Length: how long can a URL be? - SISTRIX](https://www.sistrix.com/ask-sistrix/technical-seo/site-structure/url-length-how-long-can-a-url-be/)
[^10]: [前端Javascript 限制input 中英文字數 - Medium](https://medium.com/codememo/%E5%89%8D%E7%AB%AF-javascript-%E9%99%90%E5%88%B6-input-%E4%B8%AD%E8%8B%B1%E6%96%87%E5%AD%97%E6%95%B8-7908a61a712b)
