+++
title = "Kotlin 後端專案：為何 Service Layer 是不可或缺的一層？"
description = "你是否曾在 Controller 直接調用 Repository？本文將解析這種做法的隱憂，並說明 Service Layer 帶來的多重好處。最後還提供實用的遷移步驟，助你優化專案架構！"
date = "2025-05-26T14:54:36Z"
updated = "2025-07-28T04:22:18.480Z"
draft = false

[taxonomies]
tags = [ "Prompt Engineering", "Design Pattern" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/5AdtYNriKsZAr497zTJEmR?invite=dOLYGeJyZJqVX>"
+++
{% editor_note() %}
其實在學 Kotlin 的不是我，是另一位認識的網友。  
我幫他 code review 之後發現他沒有做 service layer。

我的問句是一種 *使用 AI 來產生詳細指引* 的提問法，當資深人員在產出指南給新人時非常適合這種提示詞技巧。
{% end %}

{% chat(speaker="jim") %}
我正在學習規劃 kotlin 後端專案架構。請你說明為什麼不應該直接在 Controller 中使用 Repository，Service layer 帶來的好處為何。並且在最後說明從現有的「在 controller 中調用 Repository」的程式遷移到「擁有 service layer」的程式的 TODO 工作步驟。以 bullet point 撰寫工作步驟。
{% end %}

{% chat(speaker="felo") %}
在軟體架構設計中，分層是一種常見且重要的實踐，旨在提高程式碼的可維護性、可測試性和可重用性。
{% end %}

當 Controller 直接調用 Repository 時，雖然在簡單情境下看似便捷，但隨著應用程式複雜度的增加，會導致多種問題，例如業務邏輯分散、關注點混淆以及測試困難。引入 Service Layer 作為 Controller 和 Repository 之間的中介層，能夠有效地封裝業務邏輯、促進關注點分離，並簡化各層的職責，從而構建更健壯、更易於管理的應用程式。本報告將闡述不應直接在 Controller 中使用 Repository 的原因、Service Layer 帶來的好處，以及從現有架構遷移至包含 Service Layer 架構的具體步驟。

## Controller、Service 與 Repository 的職責

在典型的三層式架構中，各層的職責如下：

* **Controller (控制層):** 作為應用程式的入口點，負責接收客戶端的請求（例如 HTTP 請求），解析請求參數，並調用 Service Layer 處理業務邏輯。處理完成後，Controller 會將 Service Layer 返回的結果組裝成適當的回應（例如 JSON、HTML）返回給客戶端 [^3][^19][^101]。Controller 應保持輕量，主要關注請求的接收與轉發，以及回應的呈現 [^8][^11]。

* **Service (服務層):** 包含應用程式的核心業務邏輯 [^13][^19]。它協調對 Repository 的調用以進行數據持久化操作，執行業務規則、數據驗證、交易管理等 [^13][^16][^58]。Service Layer 封裝了複雜的業務流程，並向上層（Controller）提供清晰定義的接口 [^13][^68][^69]。

* **Repository (倉儲層/持久層):** 負責與數據儲存（例如資料庫）進行交互，執行數據的創建 (Create)、讀取 (Read)、更新 (Update) 和刪除 (Delete) 等 CRUD 操作 [^3][^13][^90]。Repository 抽象了數據訪問的細節，使得上層（通常是 Service Layer）無需關心具體的數據庫實現 [^3][^97]。

## 為何不建議在 Controller 中直接使用 Repository

儘管在某些極其簡單的 CRUD 操作中，Controller 直接調用 Repository 看似可以減少程式碼量，但從長遠來看，這種做法會帶來以下問題：

* **違反關注點分離 (Violation of Separation of Concerns):** Controller 的主要職責是處理 HTTP 請求和回應流程，以及基本的輸入驗證 [^56]。如果直接在 Controller 中調用 Repository，會將數據訪問邏輯甚至部分業務邏輯耦合到 Controller 中，導致 Controller 職責不清、變得臃腫，難以理解和維護 [^38][^56]。

* **降低程式碼可重用性 (Reduced Code Reusability):** Repository 通常設計為可被應用程式中多個部分（例如不同的 Service）重用的組件 [^56]。如果將 Repository 直接注入到 Controller 中，其重用性會受到限制 [^38][^56]。業務邏輯如果散落在多個 Controller 中，也難以被重用 [^8]。

* **增加測試複雜度 (Increased Testing Complexity):** 當 Controller 與 Repository 緊密耦合時，對 Controller 進行單元測試會變得更加困難 [^38][^56]。測試 Controller 時，可能需要模擬 Repository 的行為，如果業務邏輯也混雜其中，測試案例會更複雜。

* **業務邏輯分散 (Scattered Business Logic):** 如果沒有 Service Layer，業務邏輯可能會分散在各個 Controller 中，或者甚至部分洩漏到 Repository 層 [^38][^42]。這使得業務規則難以集中管理和修改。

* **交易管理困難 (Difficulty in Transaction Management):** 交易管理通常涉及多個數據庫操作的原子性。Service Layer 是實施交易管理的理想場所 [^13]。如果 Controller 直接操作 Repository，交易邊界的劃分會變得不清晰或需要在 Controller 層處理，這並不符合其職責。

## Service Layer 的優勢

引入 Service Layer 可以帶來諸多好處，有效解決上述問題：

* **封裝業務邏輯 (Encapsulation of Business Logic):** Service Layer 是應用程式核心業務邏輯的集中地 [^8][^13][^58]。所有業務規則、計算、流程協調都在這一層完成，使得業務邏輯更易於管理和修改 [^58][^68]。

* **提升程式碼可重用性 (Improved Code Reusability):** 封裝在 Service 中的業務邏輯可以被多個 Controller 或其他 Service 重用，減少了程式碼重複 [^8][^56][^58]。

* **簡化 Controller (Simplified Controllers):** Controller 的職責回歸到處理 HTTP 請求和回應，以及調用 Service [^8][^11]。Controller 變得更輕量、更專注，易於理解和維護 [^8][^74]。

* **易於測試 (Easier Testing):** Service Layer 可以獨立於 Controller 和 Repository 進行單元測試 [^8][^56]。由於業務邏輯集中在 Service 中，可以針對性地編寫測試案例，驗證業務規則的正確性 [^56][^61]。

* **明確的交易管理 (Clear Transaction Management):** Service Layer 是實施交易管理的理想位置 [^13]。可以確保涉及多個數據庫操作的業務功能在一個事務中完成，保證數據的一致性 [^13]。

* **促進關注點分離 (Promotion of Separation of Concerns):** Service Layer 作為 Controller 和 Repository 之間的中介，清晰地劃分了表現邏輯、業務邏輯和數據訪問邏輯，使得整體架構更加清晰和模組化 [^8][^56][^58]。

* **提高可維護性和可擴展性 (Enhanced Maintainability and Scalability):** 清晰的分層和職責劃分使得系統更容易維護 [^8][^58]。當需求變更或新增功能時，可以更容易地定位和修改相關程式碼，同時也方便系統的擴展 [^58][^67]。

## 從「Controller 中調用 Repository」遷移至「擁有 Service Layer」的 TODO 工作步驟

將現有的直接在 Controller 中調用 Repository 的 Kotlin 後端專案遷移到包含 Service Layer 的架構，可以遵循以下步驟：

* **分析現有 Controller：**
  * 識別 Controller 中直接調用 Repository 的部分。
  * 找出 Controller 中包含的任何業務邏輯（例如數據轉換、條件判斷、多個 Repository 操作的協調等）。

* **定義 Service 接口 (Interface)：**
  * 根據識別出的業務邏輯和 Controller 的需求，為每個主要的業務功能或實體定義 Service 接口。
  * 接口應描述業務操作，而不是數據庫操作。

* **實作 Service 類別 (Implementation Class)：**
  * 創建 Service 接口的實作類別。
  * 將 Repository 作為依賴注入到 Service 實作類別中 [^4][^102]。
  * 將原先在 Controller 中的業務邏輯遷移到相應的 Service 方法中 [^4][^71]。
  * Service 方法內部調用 Repository 進行數據持久化操作。

* **重構 Controller：**
  * 將 Service 作為依賴注入到 Controller 中 [^4][^11][^74]。
  * 修改 Controller 中的方法，使其不再直接調用 Repository，而是調用注入的 Service 的方法 [^4][^71][^74]。
  * 確保 Controller 只負責接收請求、調用 Service 並返回回應。

* **處理數據傳輸對象 (DTOs)：**
  * 考慮在各層之間使用 DTOs 進行數據傳輸，避免直接暴露數據庫實體 (Entities) 到 Controller 或客戶端 [^46][^53]。
  * Service Layer 可以負責實體與 DTO 之間的轉換。

* **配置依賴注入 (Dependency Injection)：**
  * 確保 Service 和 Repository 都被正確配置為依賴注入容器（例如 Spring、Koin、Hilt 等）中的組件，以便能夠自動注入 [^4][^5]。

* **編寫單元測試：**
  * 為新的 Service Layer 編寫單元測試，驗證其業務邏輯的正確性。
  * 更新或編寫 Controller 的單元測試，模擬 Service 的行為。

* **進行整合測試：**
  * 執行整合測試，確保 Controller、Service 和 Repository 之間的交互按預期工作。

* **程式碼審查與重構：**
  * 對遷移後的程式碼進行審查，確保其符合設計原則和最佳實踐。
  * 進行必要的重構以提高程式碼品質。

* **逐步遷移 (Optional but Recommended for Large Projects)：**
  * 對於大型專案，可以考慮逐步遷移，例如先選擇一個或幾個模組進行改造，驗證可行性後再擴展到整個專案。

[^3]: [控制层（Controller） 服务层（Service） 持久层（Repository ...](https://blog.csdn.net/m0_59281987/article/details/137890739)
[^4]: [SpringBoot的Controller,Service,Repository层的使用 - CSDN博客](https://blog.csdn.net/u013385925/article/details/87969180)
[^5]: [使用Hilt 插入依附元件| App architecture - Android Developers](https://developer.android.com/training/dependency-injection/hilt-android?hl=zh-tw)
[^8]: [Laravel中服务层（Service Layer）使用详解原创 - CSDN博客](https://blog.csdn.net/yang45310/article/details/130881513)
[^11]: [【Spring Boot】第4課－實作三層式架構的Service 與Repository](https://chikuwacode.github.io/articles/spring-boot-three-tier-architecture/)
[^13]: [简单地聊一聊Spring Boot的构架 - 腾讯云](https://cloud.tencent.com/developer/article/2358428)
[^16]: [在Service层中进行验证(还是翻的) 转载 - CSDN博客](https://blog.csdn.net/weixin_34221332/article/details/94603254)
[^19]: [Spring 練功場#2筆記-知識量爆發 - YungHsin](https://yunghsincc.medium.com/spring-%E7%B7%B4%E5%8A%9F%E5%A0%B4-2%E7%AD%86%E8%A8%98-%E7%9F%A5%E8%AD%98%E9%87%8F%E7%88%86%E7%99%BC-b944f5a3a0ec)
[^38]: [Best Practices in Spring Boot: Avoid Injecting Repositories ...](https://medium.com/nerd-for-tech/best-practices-in-spring-boot-avoid-injecting-repositories-directly-into-controllers-f3867ff61ec3)
[^42]: [Should I always use service, or can I use the repositories ...](https://stackoverflow.com/questions/8943667/should-i-always-use-service-or-can-i-use-the-repositories-directly)
[^46]: [Is it a bad practice that controller uses repository directly?](https://www.reddit.com/r/learnjava/comments/1bltng9/is_it_a_bad_practice_that_controller_uses/)
[^53]: [控制器直接用repository 是不是不太好？ : r/learnjava - Reddit](https://www.reddit.com/r/learnjava/comments/1bltng9/is_it_a_bad_practice_that_controller_uses/?tl=zh-hans)
[^56]: [Best Practices in Spring Boot: Avoid Injecting Repositories Directly into Controllers | by Vidushika Dasanayaka | Nerd For Tech | Medium](https://medium.com/nerd-for-tech/best-practices-in-spring-boot-avoid-injecting-repositories-directly-into-controllers-f3867ff61ec3)
[^58]: [Understanding the Service Layer in Software Architecture](https://medium.com/@LadlaAqeel/understanding-the-service-layer-in-software-architecture-1d2019fb58c9)
[^61]: [Test Automation in Service Layer | Top 5 Benefits](https://www.travancoreanalytics.com/insight/test-automation-in-service-layer/)
[^67]: [Do I need a service layer? - Coderanch](https://coderanch.com/t/100875/engineering/service-layer)
[^68]: [Understanding the Service Layer in Software Architecture | by LadlaAqeel | Medium](https://medium.com/@LadlaAqeel/understanding-the-service-layer-in-software-architecture-1d2019fb58c9)
[^69]: [Service Layer Pattern in Java: Enhancing Application Architecture with Robust Service Layers | Java Design Patterns](https://java-design-patterns.com/patterns/service-layer/)
[^71]: [SpringBoot的Controller,Service,Repository层的使用 - CSDN博客](https://blog.csdn.net/li_dabo/article/details/137279185)
[^74]: [【Spring Boot】第4課－實作三層式架構的Service 與Repository](https://chikuwacode.github.io/articles/spring-boot-three-tier-architecture/)
[^90]: [控制层（Controller） 服务层（Service） 持久层（Repository ...](https://blog.csdn.net/m0_59281987/article/details/137890739)
[^97]: [简单理解Repository层、Service层、Controller层 - CSDN博客](https://blog.csdn.net/weixin_37575140/article/details/130694553)
[^101]: [Controller、@Service和@Repository注解详解 - CSDN博客](https://blog.csdn.net/Mr_VK/article/details/122762743)
[^102]: [SpringBoot的Controller,Service,Repository层的使用-CSDN博客](https://blog.csdn.net/li_dabo/article/details/137279185)
