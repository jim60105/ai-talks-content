+++
title = "HTTP GET 與 POST 傳輸機敏資料的安全迷思與 RESTful 設計抉擇"
description = "你是否認為用 POST 就能安全傳輸機敏資料？本文深入解析 GET 與 POST 的真正差異，並揭示 RESTful 設計與資安的最佳平衡策略。"
date = "2025-08-15T06:30:01Z"
updated = "2025-09-07T16:09:32Z"
draft = false

[taxonomies]
tags = [ "Backend", "Design Pattern", "Security" ]
providers = [ "Felo Search" ]

[extra]
withAI = "<https://felo.ai/search/ATa8WPUDvmREgtonjRv3eW?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
我們公司的開發安全指引中說，「不應使用 HTTP GET 傳輸機敏資料」，並且「應使用 HTTP POST 方法」。 請你思考並解說這個規則是否正確。使用 POST 是否能帶來足夠的提升資安的效果，讓我應該放棄部份的 RESTful 設計並這麼做? 考量在確實是 GET 語境時，我是否會因為使用 GET 而非 POST 傳送機敏資料而曝露在風險之中?
{% end %}

{% chat(speaker="felo") %}
開發安全指引中「不應使用 HTTP GET 傳輸機敏資料，應改用 POST」的規則，其核心精神是為了防止機敏資訊因 GET 的特性而暴露在 URL 中，這在方向上是正確的，但此規則本身是一個過於簡化的論述，可能導致開發者產生「POST 本身就是安全的」此一危險的誤解。
{% end %}

將資料擷取（GET 語境）的操作改用 POST 方法，雖然能避免資料出現在 URL、瀏覽器歷史紀錄與伺服器日誌中，從而降低了*資料暴露*的風險，但這並非真正的*傳輸安全*。在未使用 HTTPS 的情況下，無論是 GET 還是 POST，其傳輸的資料均為明文，可被輕易攔截[^38][^49]。真正的傳輸安全必須依賴 HTTPS 加密[^52][^77]。

此外，為了這項有限的「隱私」提升而放棄 RESTful 設計原則，會帶來顯著的工程代價，包括：破壞 API 的語意化、喪失 GET 的冪等性（Idempotency）與可快取性（Cacheability），從而增加系統的複雜性與維護難度，並可能引發非預期的副作用[^5][^76]。

因此，更精確且安全的實踐應是：

1. **強制使用 HTTPS**：這是保障傳輸安全的根本，所有 API 端點都應使用 HTTPS[^12][^24]。
2. **遵循 RESTful 原則**：在 HTTPS 的基礎上，繼續使用語意化的 HTTP 方法（GET 用於讀取，POST 用於建立）[^4][^9]。
3. **避免在 URL 中放置機敏資料**：這是對原始規則的精煉。即使在 HTTPS 下，也應避免將密碼、Token 等高度敏感資訊置於 URL 參數中，以防其在伺服器日誌、瀏覽器歷史等環節中留下紀錄[^42][^106]。

結論是，不應盲目地將所有 GET 操作替換為 POST。應優先確保 HTTPS 的全面實施，然後在遵循 RESTful 設計的同時，審慎評估並最小化 URL 中參數的敏感性。

---

## HTTP GET 與 POST 在資料傳輸上的根本差異

要理解其安全性議題，首先必須釐清 GET 與 POST 在 HTTP 協定中的設計與資料承載方式。這兩者最直觀的區別常被比喻為「明信片」與「信封」[^6][^86]。

### GET：明信片模型

GET 請求旨在從伺服器*擷取*資源[^17][^32]。其所有參數都附加在 URL 的查詢字串（Query String）中，對外完全可見[^86]。

* **範例：** `https://api.example.com/users?id=123`
* **特性：**
  * **可見性：** 參數直接暴露在 URL 中，任何能看到 URL 的環節（如瀏覽器網址列、網路代理、伺服器日誌）都能看到參數內容[^41][^48][^66]。
  * **長度限制：** URL 長度受限於瀏覽器和伺服器，通常最大約為 2048 個字元[^6][^87]。
  * **可快取性：** GET 請求的結果可以被瀏覽器或代理伺服器快取，以提升後續存取效能[^6][^32]。
  * **冪等性（Idempotency）：** 多次發送相同的 GET 請求，對伺服器資源狀態的影響應完全相同，即不會產生副作用[^5][^126]。

### POST：信封模型

POST 請求旨在向伺服器*提交*資料，通常用於*建立*新資源或觸發需要附帶資料的處理程序[^17][^26][^37]。其參數被放置在請求的主體（Request Body）中，不會顯示在 URL 上[^39][^86]。

* **範例：**

    ```txt,name=http
    POST /users HTTP/1.1
    Host: api.example.com
    Content-Type: application/json

    {"name": "Judy", "role": "admin"}
    ```

* **特性：**
  * **隱蔽性：** 參數位於請求主體內，不會出現在 URL 中，提供了表面的隱私[^6][^73]。
  * **無長度限制：** 理論上可以傳輸大量資料[^6][^78]。
  * **不可快取性：** POST 請求的結果預設不可被快取[^6][^87]。
  * **非冪等性：** 多次發送相同的 POST 請求可能會建立多個資源，產生不同的結果[^5][^92]。

## 安全性分析：「POST 比 GET 安全」的迷思與真相

公司的安全指引根植於一個普遍的觀念：POST 比 GET 更安全。這個說法在特定情境下部分成立，但若不深入理解其前提與限制，將會產生嚴重的安全誤區。

### GET 的風險：暴露而非洩漏

使用 GET 傳輸機敏資料的主要風險在於其*可見性*，而非傳輸過程本身的不安全。具體風險點包括：

1. **URL 可見性與肩窺攻擊（Shoulder Surfing）：** 使用者螢幕上的瀏覽器網址列會直接顯示包含機敏資料的 URL[^48]。
2. **瀏覽器歷史紀錄：** 包含參數的完整 URL 會被儲存在瀏覽器歷史紀錄中，任何能存取該電腦的人都可能看到[^6][^47][^60]。
3. **伺服器與代理日誌（Server/Proxy Logs）：** 網站伺服器、防火牆、負載平衡器等網路節點通常會記錄完整的請求 URL，這會導致機敏資料以明文形式儲存在日誌檔案中[^40][^47][^65]。
4. **書籤與分享：** 使用者可能會將包含機敏資料的 URL 加入書籤或不經意地分享給他人[^47][^87]。
5. **Referrer 標頭：** 當使用者從一個包含機敏資料的頁面點擊連結到另一個網站時，完整的 URL 可能會透過 HTTP `Referer` 標頭傳送給目標網站。

這些風險的核心是**資料在端點（客戶端、伺服器）的靜態儲存與暴露**。

### POST 的「安全」假象及其侷限性

將請求從 GET 改為 POST，確實可以有效規避上述所有風險，因為資料被移入了請求主體[^6][^86]。然而，這僅僅是將資料從「明信片」變成了「未加密的信封」。

### 關鍵限制：HTTP 本質是明文傳輸

無論是 GET 還是 POST，只要是透過標準的 HTTP 協定傳輸，所有資料（包括 URL、標頭和主體）都是未加密的明文[^38][^49][^95]。攻擊者若處於網路中間位置（Man-in-the-Middle），可以輕易地使用封包分析工具（如 Wireshark）攔截並讀取所有傳輸內容[^38][^43][^93]。

{{ cr(body="因此，在 HTTP 下，POST 與 GET 一樣不安全。") }}認為 POST 能提供傳輸層級的安全性是一個極其危險的誤解。

### 真正的解決方案：**HTTPS** (HTTP over TLS)

傳輸安全的基石是 HTTPS。HTTPS 會在 HTTP 協定外層包裹一層 TLS/SSL 加密。這層加密保護了整個 HTTP 請求，包括：

* 請求行（包含 URL 和查詢參數）
* 請求標頭（Headers）
* 請求主體（Body）

當使用 HTTPS 時，即使是 GET 請求中的 URL 參數，在傳輸過程中也是被加密的，中間人無法窺視其內容[^52][^56][^77]。這從根本上解決了傳輸過程中的資料洩漏風險。

## 違背 RESTful 設計的工程代價

在已經實施 HTTPS 的前提下，若仍堅持將 GET 操作改為 POST，雖然能解決日誌記錄等端點暴露問題，但會引入一系列嚴重的軟體工程問題，這些代價往往超過其帶來的有限好處。

RESTful API 的核心理念是使用統一的介面（HTTP 方法）來操作定義清晰的資源（URL），以達到系統的鬆散耦合與平台獨立性[^2][^125]。

### 破壞語意化與可讀性

RESTful 設計風格將 HTTP 方法對應到資料的 CRUD（建立、讀取、更新、刪除）操作[^4][^91]。

* `GET`: 讀取 (Read)
* `POST`: 建立 (Create)
* `PUT`/`PATCH`: 更新 (Update)
* `DELETE`: 刪除 (Delete)

這種對應關係使得 API 非常直觀且易於理解[^7][^25]。當開發者看到一個 `GET /api/users/123` 的請求時，能立刻明白其意圖是「獲取 ID 為 123 的使用者資訊」。如果將其改為 `POST /api/users/123`，語意就變得模糊不清，是「建立」還是「獲取」？這會極大地增加開發者的溝通成本與學習曲線，違背了 RESTful 設計的初衷[^28][^94]。

### 喪失冪等性（Idempotency）

冪等性是指一個操作無論執行一次或多次，其結果都相同[^5][^126]。GET、PUT、DELETE 都是冪等的，而 POST 不是[^5][^92]。

* **GET /users/123**：執行 100 次，你得到的都是同一個使用者的資料，伺服器狀態不變。
* **POST /users**：執行 100 次，你可能會建立 100 個新使用者。

將讀取操作用 POST 實現，會讓這個本應安全的讀取操作失去冪等性。這在網路不穩定、需要重試的場景下尤其危險。客戶端或代理在請求超時後自動重試一個 POST 請求，可能會引發非預期的後果。

### 喪失可快取性（Cacheability）

HTTP 協定允許瀏覽器和中間代理（如 CDN）快取 GET 請求的回應[^6][^32]。這對於不常變動的資料能極大提升效能、降低伺服器負載。而 POST 請求的回應在預設情況下是不可被快取的[^6][^87]。將讀取操作改為 POST，等於放棄了整個 HTTP 快取機制帶來的效能優勢。

### 例外情況：何時可考慮用 POST 進行查詢

儘管應盡力避免，但在極少數情況下，使用 POST 進行資料查詢是可以接受的。最常見的場景是當查詢條件過於複雜或龐大，導致 GET 請求的 URL 長度超過了瀏覽器或伺服器的限制（約 2KB）[^99]。例如，一個需要傳入數百個 ID 進行批次查詢的 API，將這些 ID 放在 POST 的請求主體中是更務實的作法。

## 結論與最佳實踐建議

綜合以上分析，公司安全指引中的規則是一個善意但過於簡化的建議。直接遵循它可能會導致對安全性的錯誤認知，並犧牲掉 RESTful 設計帶來的工程優勢。

一個更成熟、更安全的 API 設計策略應遵循以下層次：

1. **第一道防線：強制使用 HTTPS**  
    這是最重要且不可妥協的基礎。確保所有 API 端點都透過 HTTPS 提供服務，從根本上保障資料在*傳輸過程*中的機密性與完整性[^12][^24][^63]。

2. **第二道防線：遵循 RESTful 語意化設計**  
    在 HTTPS 的保護下，應大膽地使用符合語意的 HTTP 方法。使用 `GET` 進行資料讀取，以利用其冪等性、可快取性和清晰的語意[^4][^9][^107]。

3. **第三道防線：精煉化 URL 參數管理**  
    這是對原始規則精神的正確詮釋。即使在 HTTPS 下，也應避免將高度機敏的資料（如密碼、Session Token、API 金鑰）直接放在 URL 參數中。這是為了防範*端點暴露*的風險，例如伺服器日誌未經處理、開發者不慎分享 URL 等[^42][^106]。
    * **可接受的 GET 參數：** 非敏感的識別碼（如 UUID）、分頁參數（`limit`, `offset`）、篩選條件等[^2][^12]。
    * **應避免的 GET 參數：** 密碼、個人身份證號、高權限的存取權杖。這些更適合透過 HTTP 標頭（如 `Authorization`）或在 POST/PUT 的請求主體中傳遞。

4. **伺服器端強化：日誌過濾與清理**  
    作為最後的補充，應在伺服器端實施日誌過濾機制，自動移除或遮蔽日誌中可能出現的敏感參數，確保即使發生配置錯誤，機敏資料也不會被永久記錄下來[^65]。

總結來說，您不應該僅僅因為安全考量而放棄 RESTful 設計。正確的做法是建立一個多層次的防禦體系：**以 HTTPS 為基礎，以 RESTful 為設計框架，以謹慎的參數管理為細節優化**。這樣才能在確保安全的同時，兼顧系統的健壯性、可維護性與效能。

[^2]: [RESTful Web API 设计的最佳做法 - Microsoft Learn](https://learn.microsoft.com/zh-cn/azure/architecture/best-practices/api-design)
[^4]: [RESTful API 設計指南，3 個必備條件缺一不可！](https://kucw.io/blog/restful-api/)
[^5]: [五分鐘理解RESTful API設計](https://jamesshieh.medium.com/%E4%BA%94%E5%88%86%E9%90%98%E7%90%86%E8%A7%A3restful-api%E8%A8%AD%E8%A8%88-14f413b031f7)
[^6]: [Http 中的GET 和POST 的差別在哪裡？ - 古古的後端筆記](https://kucw.io/blog/http-get-post/)
[^7]: [26. [BE] API 設計拿資料要透過POST，會有什麼問題嗎？](https://ithelp.ithome.com.tw/articles/10227433)
[^9]: [【科技友瘋狂】RESTful API：全端工程師必懂的API 設計標準](https://www.wpgdadatong.com/blog/detail/77027)
[^12]: [RESTful API 设计指南- 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2014/05/restful_api.html)
[^17]: [API 的請求方法：GET 或POST ?| Yuanlin Lin 林沅霖](https://yual.in/posts/should-i-use-get-or-post)
[^24]: [REST API 安全最佳做法 - Akamai](https://www.akamai.com/zh/blog/security/rest-api-security-best-practices)
[^25]: [什麼是RESTful API?路由語義化設計教學 - ALPHA Camp](https://tw.alphacamp.co/blog/rest-restful-api)
[^26]: [重新認識HTTP請求方法 - iThome](https://www.ithome.com.tw/voice/80062)
[^28]: [就是為了「簡化工程師之間的溝通成本」。 所以RESTful API](https://www.facebook.com/kucw.io/posts/restful-api-%E6%98%AF%E4%BB%80%E9%BA%BC%E4%B8%80%E6%96%87%E6%90%9E%E6%87%82%E8%A8%AD%E8%A8%88-restful-api-%E5%BF%85%E5%82%99%E7%9A%84-3-%E5%80%8B%E6%A2%9D%E4%BB%B6%E5%93%88%E5%9B%89%E6%88%91%E6%98%AF%E5%8F%A4%E5%8F%A4%E5%8F%88%E5%88%B0%E4%BA%86%E6%AF%8F%E9%80%B1%E4%BA%8C%E7%9A%84%E5%BE%8C%E7%AB%AF%E9%9B%BB%E5%AD%90%E5%A0%B1%E4%BA%86%E9%80%99%E9%80%B1%E8%A6%81%E8%A8%8E%E8%AB%96%E7%9A%84%E4%B8%BB%E9%A1%8C%E6%98%AFrestful-a/503460005403697/)
[^32]: [解析OWASP API Security Top 10 - 叡揚資訊](https://www.gss.com.tw/eis/214-eis-99/2647-eis99-8)
[^37]: [HTTP方法| RESTful API 中文网 - 什么是REST](https://restful.p2hp.com/resources/http-methods)
[^38]: [HTTP 的POST 方法比GET 方法安全性比较原创 - CSDN博客](https://blog.csdn.net/qq_49641239/article/details/118399236)
[^39]: [HTTP： GET vs POST 原创 - CSDN博客](https://blog.csdn.net/apple_64847327/article/details/143061165)
[^40]: [HTTP POST vs GET: Is One More Secure For Use In REST ...](https://medium.com/@robert.broeckelmann/http-post-vs-get-is-one-more-secure-for-use-in-rest-apis-2469753121b0)
[^41]: [What Is the Difference Between GET and POST Methods?](https://www.baeldung.com/cs/http-get-vs-post)
[^42]: [[Web] RESTful 敏感性GET 參數 - YuCheng's Site](https://blog.tigernaxo.com/posts/web/security-get/)
[^43]: [Http 中的GET 和POST 的差別在哪裡？ - 古古的後端筆記](https://kucw.io/blog/http-get-post/)
[^47]: [GET vs POST, which is more secure? [duplicate]](https://security.stackexchange.com/questions/33837/get-vs-post-which-is-more-secure)
[^48]: [Verifying I understand GET vs. POST : r/AskNetsec - Reddit](https://www.reddit.com/r/AskNetsec/comments/2x9slv/verifying_i_understand_get_vs_post/)
[^49]: [為什麼HTTP 不安全 - Cloudflare](https://www.cloudflare.com/zh-tw/learning/ssl/why-is-http-not-secure/)
[^52]: [使用HTTPS 和HTTP 嚴格傳輸安全性，讓惡意中間人無法得逞](https://web.dev/articles/transport-layer-security?hl=zh-tw)
[^56]: [HTTP请求方法的安全性- get、put - CSDN博客](https://blog.csdn.net/gybshen/article/details/123945099)
[^60]: [HTTP 方法：GET 对比POST - 菜鸟教程](http://www.runoob.com/tags/html-httpmethods.html)
[^63]: [HTTP是什麼？HTTP 協定：從請求流程到DDoS攻擊手法與防禦](https://www.skycloud.com.tw/news/72)
[^65]: [REST API 潔癖症患者：幹嘛對GET 跟POST 這麼堅持？ - Reddit](https://www.reddit.com/r/learnprogramming/comments/1hoyhmq/rest_api_purists_why_so_stubborn_about_get_vs_post/?tl=zh-hant)
[^66]: [面试官：说一下GET 和POST 的区别？ #145 - GitHub](https://github.com/linwu-hi/code-interview/issues/145)
[^73]: [什么是HTTP POST 请求？初学者指南与示范原创 - CSDN博客](https://blog.csdn.net/m0_71808387/article/details/140204629)
[^76]: [前端应该懂的HTTP和HTTPS知识GET和POST的请求的区别 ...](https://juejin.cn/post/7196124982233382971)
[^77]: [HTTP 與HTTPS 的比較- 傳輸協定之間的差異 - AWS](https://aws.amazon.com/tw/compare/the-difference-between-https-and-http/)
[^78]: [HTTP请求的GET与POST方式的区别- 爱跳舞的程序员 - 博客园](https://www.cnblogs.com/rdchen/p/14048683.html)
[^86]: [Http 中的 GET 和 POST 的差別在哪裡？](https://kucw.io/blog/http-get-post/)
[^87]: [HTTP POST vs GET: Is One More Secure For Use In REST APIs? - APILayer Blog - All About APIs: AI, ML, Finance, &amp; More APIs](https://blog.apilayer.com/http-post-vs-get-is-one-more-secure-for-use-in-rest-apis/)
[^91]: [RESTful API 設計指南，3 個必備條件缺一不可！](https://kucw.io/blog/restful-api/)
[^92]: [五分鐘理解RESTful API設計](https://jamesshieh.medium.com/%E4%BA%94%E5%88%86%E9%90%98%E7%90%86%E8%A7%A3restful-api%E8%A8%AD%E8%A8%88-14f413b031f7)
[^93]: [Http 中的GET 和POST 的差別在哪裡？ - 古古的後端筆記](https://kucw.io/blog/http-get-post/)
[^94]: [26. [BE] API 設計拿資料要透過POST，會有什麼問題嗎？](https://ithelp.ithome.com.tw/articles/10227433)
[^95]: [RESTful API - Choson資安大小事](https://choson.lifenet.com.tw/?p=1161)
[^99]: [RESTful API 设计指南- 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2014/05/restful_api.html)
[^106]: [REST API 安全最佳做法 - Akamai](https://www.akamai.com/zh/blog/security/rest-api-security-best-practices)
[^107]: [實務上，RESTful API 的Best Practice - Medium](https://medium.com/johnliu-%E7%9A%84%E8%BB%9F%E9%AB%94%E5%B7%A5%E7%A8%8B%E6%80%9D%E7%B6%AD/%E5%AF%A6%E5%8B%99%E4%B8%8A-restful-api-%E7%9A%84-best-practice-564cf5fdd87e)
[^125]: [Web API 设计最佳做法 - Azure Architecture Center | Microsoft Learn](https://learn.microsoft.com/zh-cn/azure/architecture/best-practices/api-design)
[^126]: [HTTP方法 | RESTful API 中文网](https://restful.p2hp.com/resources/http-methods)
