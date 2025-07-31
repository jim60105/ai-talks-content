+++
title = "Flatpak 如何讓應用讀取系統程序？TL;DR: 不行。"
description = "Flatpak 沙盒機制下，應用預設無法讀取系統程序列表。即使透過權限設定（如 --filesystem=host 或 Flatseal）開放 /proc 存取，仍可能因沙盒限制無法達成需求。本文說明原因、權限調整方式及其侷限，並建議如需此功能可考慮非 Flatpak 版本。"
date = "2025-03-28T13:28:44Z"
updated = "2025-07-27T13:12:23.739Z"
draft = false

[taxonomies]
tags = [ "Flatpak", "Linux" ]
providers = [ "Felo Search" ]


[extra]
withAI = "<https://felo.ai/search/FL3rE5r6spxkdLhC9wGsZg?invite=dOLYGeJyZJqVX>"

  [extra.comments]
  id = "114924536011258063"
+++
{% alert(edit=true) %}
問這個問題是因為我想要讓 Discord Flatpak 版能抓到運行中的遊戲，然而並沒有弄成功。  
若有人成功了希望能在下方回覆區跟我分享一下，我非常想知道。
{% end %}

{% chat(speaker="jim") %}
flatpak 我要開放使應用能「讀取已開啟的程式」的權限，請問是哪個權限，並給我指令
{% end %}

{% chat(speaker="felo") %}
Flatpak 應用程式預設在沙盒環境中執行，無法讀取系統中正在執行的程序列表。根據搜尋結果，這是 Flatpak 的安全設計，目的是隔離應用程式以提高系統安全性。
{% end %}

## 關於讀取系統程序的權限

根據搜尋結果[^18]，Flatpak 應用程式無法直接讀取系統程序列表（即 `/proc` 目錄中的資訊）。這是因為：

1. Flatpak 應用程式在沙盒中執行，預設無法存取沙盒外的程序[^8]
2. 這是 Flatpak 的安全設計之一，目的是限制應用程式的系統調用能力[^8]

## 可能的解決方案

雖然 Flatpak 沒有直接提供「讀取系統程序列表」的單一權限選項，但您可以嘗試以下方法：

### 1. 使用 `--filesystem=host` 權限

這會給予應用程式對整個主機檔案系統的存取權限，包括 `/proc` 目錄：

```
flatpak override --filesystem=host 應用程式ID
```

請注意，這會給予應用程式非常廣泛的權限，可能會降低安全性。

### 2. 使用 Flatseal 圖形化工具

Flatseal 是一個圖形化工具，可以更方便地管理 Flatpak 應用程式的權限[^1][^6][^10]：

1. 安裝 Flatseal：`flatpak install flathub com.github.tchx84.Flatseal`
2. 開啟 Flatseal 並選擇您要修改的應用程式
3. 在「檔案系統」區域中，啟用「全部檔案系統」或「主機檔案系統」選項

### 3. 使用 `flatpak override` 指令

您可以使用以下指令來開放特定目錄的存取權限[^6][^16]：

```
flatpak override --filesystem=/proc 應用程式ID
```

## 注意事項

1. 即使開放了 `/proc` 的存取權限，某些應用程式可能仍然無法正常讀取系統程序，因為 Flatpak 的沙盒機制還包含其他限制[^8][^18]

2. 根據搜尋結果[^18]，Flatpak 開發者認為讓應用程式讀取系統程序列表可能存在安全風險，因此不太可能在未來添加這項專門的權限

3. 如果您需要這項功能，可能需要考慮使用非 Flatpak 版本的應用程式，或使用其他方式安裝該應用程式

4. 開放過多權限會降低 Flatpak 的安全性優勢，請謹慎評估是否真的需要此權限[^21]

如果您能分享您具體需要哪個應用程式讀取系統程序列表，可能會有更具體的解決方案。

{% alert(edit=true) %}
請閱讀[這篇 issue](https://github.com/flatpak/flatpak/issues/3922) 了解更多。
{% end %}

[^1]: [使用Flatpak安裝Linux軟體，最常遇到的就是權限問題了 ...](https://www.threads.net/@ivonhuang_ig/post/DC8dksYT0xR/%E4%BD%BF%E7%94%A8flatpak%E5%AE%89%E8%A3%9Dlinux%E8%BB%9F%E9%AB%94%E6%9C%80%E5%B8%B8%E9%81%87%E5%88%B0%E7%9A%84%E5%B0%B1%E6%98%AF%E6%AC%8A%E9%99%90%E5%95%8F%E9%A1%8C%E4%BA%86flatpak%E9%A0%90%E8%A8%AD%E6%98%AF%E6%B2%99%E7%9B%92%E9%9A%94%E9%9B%A2%E7%9A%84%E5%81%87%E8%A8%ADflatpak%E8%BB%9F%E9%AB%94%E6%B2%92%E6%9C%89%E8%80%83%E6%85%AE%E5%A5%BD%E6%AC%8A%E9%99%90%E8%A8%AD%E8%A8%88%E9%82%A3%E5%B0%B1%E6%9C%83%E6%9C%89%E7%89%B9%E5%AE%9A%E7%9B%AE%E9%8C%84%E7%9A%84%E6%AA%94%E6%A1%88%E7%84%A1%E6%B3%95%E8%AE%80%E5%8F%96)
[^6]: [無法開啟檔案？用Flatseal調整Linux Flatpak應用程式權限 ...](https://ivonblog.com/posts/flatseal-flatpak-permission/)
[^8]: [沙箱权限- Flatpak 文档](https://docs.flatpak.org/zh-cn/latest/sandbox-permissions.html)
[^10]: [使用Flatseal 管理Flatpak 的权限 - LinuxStory](https://linuxstory.org/manage-flatpak-permissions-with-flatseal/)
[^16]: [How to give a Flatpak app access to a directory - Ask Ubuntu](https://askubuntu.com/questions/1086529/how-to-give-a-flatpak-app-access-to-a-directory)
[^18]: [Permission to read system processes? Will it ever be added?](https://github.com/flatpak/flatpak/issues/3922)
[^21]: [Flatpak: "home" access allows trivial privilege escalation, what ...](https://discussion.fedoraproject.org/t/flatpak-home-access-allows-trivial-privilege-escalation-what-to-do-instead/112651)
