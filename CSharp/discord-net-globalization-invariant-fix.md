+++
title = "解決 Discord.NET 全球化不變模式錯誤的 5 個步驟"
description = "Discord.NET 在 Docker 或 Linux 容器裡報『Only the invariant culture is supported』？這篇教你 5 個簡單步驟，徹底解決全球化不變模式的文化錯誤，讓機器人跨平台穩定執行，不再被文化設定卡住！"
date = "2025-04-10T04:27:38Z"
updated = "2025-07-27T13:19:47.390Z"
draft = false

[taxonomies]
tags = [ "C#", "Container", "Discord" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/2ZyXMkyjxbGPawwxV2Dvd4?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
Discord.NET: Only the invariant culture is supported in globalization-invariant mode.

I'm using globalization-invariant mode, how to let discord .net work without 特定的文化
{% end %}

{% chat(speaker="felo") %}
在使用 Discord.NET 時，如果你遇到「只有不變文化在全球化不變模式下受到支持」的錯誤，這通常是因為你的應用程式在全球化不變模式下運行，而 Discord.NET 需要特定的文化設定來正常運作。以下是一些解決方案，幫助你在不使用特定文化的情況下讓 Discord.NET 正常運行。
{% end %}

{% editor_note() %}
它的回覆和我問的是相反的，那是因為我的初始要求其實無法達成。

這個報錯訊息有點不容易理解:  
`Discord.NET: Only the invariant culture is supported in globalization-invariant mode.`

乍看之下像是 Discord.NET 在要求以 Invariant Culture 運行，其實正好相反，它的意思是 **此環境只有 Invariant Culture 使得它無法運行。** 正確的解答是在系統中安裝 Culture，並且 {{ cr(body="不要") }} 以 InvariantGlobalization 運行程式。
{% end %}

## **解決方案**

1. **修改 csproj 檔案**：
   在你的 `.csproj` 檔案中，添加以下設定以禁用全球化不變模式：

   ```xml
   <PropertyGroup>
       <InvariantGlobalization>false</InvariantGlobalization>
   </PropertyGroup>
   ```

2. **安裝 ICU 庫**：
   如果你在 Docker 環境中運行，確保安裝了 ICU 庫。對於基於 Alpine 的映像，你可以使用以下命令安裝：

   ```bash
   RUN apk add --no-cache icu-libs
   ```

   如果需要完整的 ICU 數據，可以安裝 `icu-data-full`：

   ```bash
   RUN apk add icu-libs icu-data-full
   ```

3. **設定環境變數**：
   確保在運行應用程式時，環境變數 `DOTNET_SYSTEM_GLOBALIZATION_INVARIANT` 被設置為 `false`，這樣可以啟用全球化支持。

4. **檢查文化設定**：
   在你的代碼中，檢查是否有任何地方使用了特定的文化設定。如果有，考慮將其改為使用不變文化（Invariant Culture），例如：

   ```csharp
   var cultureInfo = CultureInfo.InvariantCulture;
   ```

5. **捕獲例外**：
   如果你的應用程式仍然需要在不變模式下運行，確保在代碼中適當捕獲 `CultureNotFoundException`，以避免應用程式崩潰。

## **結論**

透過上述步驟，你應該能夠在不使用特定文化的情況下，使 Discord.NET 正常運行。這些設定不僅能解決文化相關的錯誤，還能確保你的應用程式在不同環境中保持穩定性。若仍有問題，建議檢查 Discord.NET 的文檔或社群論壇以獲取更多幫助[^1][^2][^4]。

[^1]: [Requiring cultures does not seem necessary #2704 - GitHub](https://github.com/discord-net/Discord.Net/issues/2704)
[^2]: [Globalization config settings - .NET | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/globalization)
[^4]: [globalization-invariant-mode.md - dotnet/runtime - GitHub](https://github.com/dotnet/runtime/blob/main/docs/design/features/globalization-invariant-mode.md)
