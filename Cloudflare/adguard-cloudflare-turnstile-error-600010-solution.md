+++
title = "AdGuard 導致 Cloudflare Turnstile 600010 錯誤原因與解法"
description = "你是否遇過 Cloudflare Turnstile 出現 600010 錯誤？本文解析 AdGuard 攔截機制如何影響驗證流程，並提供實用的排解步驟。讓你輕鬆解決驗證失敗的困擾！"
date = "2025-06-01T00:00:00Z"
updated = "2025-07-28T10:26:11Z"
draft = false

[taxonomies]
tags = [ "AdGuard", "Cloudflare" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/ajoH65hjJRax5TYkp4AASg?invite=dOLYGeJyZJqVX>"
+++
{% editor_note() %}
TL;DR: 把這條加到你的 AdGuard 過濾器條件

```
@@||challenges.cloudflare.com^$important
```

{% end %}

{% chat(speaker="jim") %}
adguard cause [Cloudflare Turnstile] Error: 600010.
{% end %}

{% chat(speaker="felo") %}
600010 屬於 Turnstile 的「客戶端錯誤」，代表使用者未能完成或觸發驗證流程，通常因為驗證腳本或 Cookie 被攔截，導致挑戰無法正常執行。
{% end %}

## **AdGuard 如何導致此錯誤？**

當啟用 AdGuard 的廣告或「煩惱過濾」規則時，可能會攔截或阻擋 Cloudflare Turnstile 所需的：

- JavaScript 檔案（如 challenges.cloudflare.com 上的 API 腳本）  
- 第三方 Cookie 或本地儲存機制  
- 必要的 onload 回呼函式註冊  

這些阻擋行為會讓 Turnstile 在載入時無法找到指定的回呼函式，最終觸發 600010 錯誤，甚至出現「Unable to find onload callback '\_turnstileCb'」的錯誤訊息[^8]。

## **解決方案**

- 在 AdGuard（或任何廣告攔截擴充套件）中將以下網域加入白名單：  
  - challenges.cloudflare.com  
  - 其他可能承載 Turnstile 腳本的子網域
- 停用與 CAPTCHA 或第三方腳本相關的過濾規則（如「Annoyances filter」、「EasyList」中的特定條目），以免將 Turnstile 腳本當成廣告攔截[^8]。  
- 清除瀏覽器快取與 Cookie，排除殘留資料影響驗證流程[^11]。  
- 如仍有問題，可切換至無痕模式或臨時停用所有瀏覽器擴充套件，確認是否為其他工具衝突所致。

[^8]: [Endless challenge at Cloudflare Verification #1765 - GitHub](https://github.com/AdguardTeam/CoreLibs/issues/1765)
[^11]: [Error code 600010 - General - Cloudflare Community](https://community.cloudflare.com/t/error-code-600010/660849)
