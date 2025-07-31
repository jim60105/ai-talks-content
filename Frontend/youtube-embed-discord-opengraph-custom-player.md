+++
title = "YouTube 影片嵌入技術與 Discord 平台限制全解析"
description = "深入解析 YouTube 影片嵌入原理及 Discord 嵌入限制，揭示自訂播放器在 Discord 的可行性。探索 Open Graph 與 Webhook 等替代方案。協助工程師打造最佳影片分享體驗。"
date = "2025-07-20T19:55:34Z"
updated = "2025-07-29T02:45:47Z"
draft = false

[taxonomies]
tags = [ "YouTube", "Discord" ]
providers = [ "Felo Search", "Claude" ]

[extra]
withAI = "<https://felo.ai/search/TMqEDuVTNAudpeT9uUj6pL?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
我注意到 Youtube 在其它網站上的預覧使用它自有的播放器。請告訴我它們使用的技術是什麼，我身為一個網頁工程師，是否也能實作我自己的預覧播放器? 我的終極目標是在其它的 app (例如 Discord) 內實現呈現我自己實作的特別的影音預覧播放器。
{% end %}

{% chat(speaker="felo") %}
本報告深度解析了在第三方應用程式（特別是 Discord）中嵌入客製化影音播放器的技術可行性。研究發現，雖然 YouTube 利用 `<iframe>` 技術和強大的 IFrame Player API，讓開發者能在自有網站上實現高度客製化的影片嵌入體驗，但要將一個完全自訂的播放器介面嵌入到如 Discord 這類的封閉平台中，**基本上是不可行的。**
{% end %}

主要原因在於 Discord 出於安全考量，不支援渲染來自任意網站的 `<iframe>` 或執行外部 JavaScript 程式碼。Discord 的影片嵌入機制嚴格依賴 Open Graph (OG) 協定，並要求 `og:video` 標籤必須直接指向一個可公開存取的影片檔案（如 MP4、WebM）。當偵測到有效的影片連結時，Discord 會使用其內建的原生播放器來播放內容，而非載入來源網站的播放器。

因此，開發者雖然可以在自己的網站上完全掌控影音播放體驗，但在 Discord 上的呈現方式則受到平台嚴格的限制。YouTube 能夠在 Discord 中顯示其自有播放器，是基於 Discord 對其進行的特殊白名單整合，這並非一項對所有開發者開放的通用功能。對於希望在 Discord 中提供豐富影片體驗的開發者，最佳策略是遵循 Discord 的規範，利用 Open Graph 實現原生播放，並可透過 Webhooks 或 Discord Bots 創建引人注目的「Rich Embeds」訊息，以引導使用者到開發者自己的網站上體驗完整的客製化播放器。

## 第一部分：YouTube 影片嵌入技術深度解析

YouTube 作為全球最大的影音平台，提供了一套成熟且功能強大的嵌入機制，讓網站開發者可以輕鬆地將影片整合到自己的網頁中。

### 核心技術：`<iframe>` 與 oEmbed

YouTube 嵌入功能的核心是 HTML 的 `<iframe>` 元素 [^52]。當您在 YouTube 影片頁面點擊「分享」然後選擇「嵌入」時，平台會產生一段 `<iframe>` 的 HTML 程式碼 [^35][^37][^58]。這段程式碼會在您的網頁中創建一個獨立的瀏覽上下文，並在其中載入 YouTube 的影片播放器。

```html
<!-- YouTube 提供的標準嵌入程式碼範例 -->
<iframe width="560" height="315" 
    src="https://www.youtube.com/embed/VIDEO_ID" 
    frameborder="0" 
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
    allowfullscreen>
</iframe>
```

此外，許多內容管理系統（如 WordPress）利用 oEmbed 技術來簡化嵌入流程 [^53]。開發者只需貼上 YouTube 影片的 URL，系統便會自動查詢 oEmbed 端點，並將其轉換為對應的 `<iframe>` 嵌入程式碼，無需手動複製貼上 [^53]。

### 使用 IFrame Player API 進行客製化

對於需要更進階控制的開發者，YouTube 提供了 IFrame Player API [^51][^54][^59]。這是一個 JavaScript API，允許開發者透過程式碼與嵌入的播放器進行互動。其主要功能包括：

* **播放控制**：播放、暫停、停止影片，以及跳轉到指定時間點 [^59]。
* **音量控制**：設定或靜音播放器的音量。
* **影片佇列**：以程式化方式載入或預備（cue）影片、播放清單 [^59]。
* **事件監聽**：可以監聽播放器狀態的變化（如播放、暫停、結束），並在事件發生時觸發自訂的函式 [^59]。
* **資訊擷取**：獲取目前播放影片的相關資料，如影片時長、目前播放時間等。

以下是一個使用 IFrame Player API 的基本範例，它會在播放器準備就緒後自動播放影片，並在播放 6 秒後停止 [^59]。

```javascript
// 此程式碼會非同步載入 IFrame Player API
var tag = document.createElement('script');
tag.src = "https://www.youtube.com/iframe_api";
var firstScriptTag = document.getElementsByTagName('script')[^0];
firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);

var player;
function onYouTubeIframeAPIReady() {
  player = new YT.Player('player', {
    height: '390',
    width: '640',
    videoId: 'M7lc1UVf-VE',
    events: {
      'onReady': onPlayerReady,
      'onStateChange': onPlayerStateChange
    }
  });
}

function onPlayerReady(event) {
  event.target.playVideo();
}

var done = false;
function onPlayerStateChange(event) {
  if (event.data == YT.PlayerState.PLAYING && !done) {
    setTimeout(stopVideo, 6000);
    done = true;
  }
}
function stopVideo() {
  player.stopVideo();
}
```

### 客製化參數與隱私選項

開發者可以透過在 `<iframe>` 的 `src` URL 中添加參數來自訂播放器的行為 [^58]。常見的參數包括：

* `autoplay=1`：自動播放影片（多數現代瀏覽器已限制此功能）。
* `start=90`：從第 90 秒開始播放影片。
* `cc_load_policy=1`：預設開啟字幕。

為了保護使用者隱私，YouTube 提供了「隱私權保護增強模式」。透過將嵌入網域從 `https://www.youtube.com` 更改為 `https://www.youtube-nocookie.com`，YouTube 將不會在使用者點擊播放前，利用嵌入影片的觀看數據來影響其在 YouTube 上的瀏覽體驗或投放個人化廣告 [^58]。

## 第二部分：在 Discord 中嵌入影片的機制與限制

將影片嵌入 Discord 的運作方式與在一般網站上截然不同。Discord 是一個封閉的生態系統，對外部內容的呈現有著嚴格的控制。

### Discord 的嵌入原理：Open Graph 協定

當使用者在 Discord 頻道中貼上一個連結時，Discord 的伺服器會抓取該連結的網頁內容，並解析其 HTML 中的 Open Graph (OG) 元標籤（meta tags）來產生預覽 [^6][^94]。要讓影片能夠在 Discord 中直接播放，網頁中必須包含特定的 OG 標籤 [^6][^11][^94]。

一個最基本的影片嵌入 OG 標籤設定如下：

```html
<meta property="og:type" content="video.other">
<meta property="og:title" content="影片標題">
<meta property="og:image" content="https://example.com/thumbnail.png">
<meta property="og:video" content="https://example.com/video.mp4">
<meta property="og:video:secure_url" content="https://example.com/video.mp4">
<meta property="og:video:type" content="video/mp4">
<meta property="og:video:width" content="1280">
<meta property="og:video:height" content="720">
```

### 關鍵限制：不支援任意 `<iframe>`

這是實現客製化播放器最根本的障礙。出於安全考量，防止惡意網站透過 `<iframe>` 進行釣魚或執行惡意腳本，Discord 不會渲染來自任意第三方網站的 `<iframe>` 內容 [^11]。因此，您無法透過 OG 標籤指向一個包含您自訂 JavaScript 播放器的網頁，並期望它在 Discord 中被渲染。Discord 只會尋找 `og:video` 標籤中指向影片檔案的直接連結 [^11][^90]。

### 影片直接連結的要求

為了讓影片在 Discord 內嵌播放，`og:video` 和 `og:video:secure_url` 標籤的 `content` 屬性**必須**指向一個影片檔案的直接 URL，例如 `https://.../my_clip.mp4` [^6][^94]。如果該 URL 指向一個 HTML 頁面或一個需要經過重定向才能取得影片檔案的連結，嵌入將會失敗，通常只會顯示一個普通的連結預覽方塊 [^6][^94]。

### 檔案格式、編碼與大小限制

在 Discord 中嵌入影片還需遵守其技術限制 [^15]。

| 限制類型 | 規格 | 來源 |
| :--- | :--- | :--- |
| **支援格式** | MP4, WebM, MOV | [^6][^22][^76][^94] |
| **支援編碼** | H.264 (AVC), VP8, HEVC (H.265) | [^5][^20][^64][^92] |
| **檔案上傳限制 (免費)** | 25MB (部分舊資料為 8MB) | [^22][^32][^91] |
| **檔案上傳限制 (Nitro)** | 50MB (Basic), 500MB (Nitro) | [^22] |
| **連結嵌入大小限制** | 約 100MB，但有使用者回報低於此值 (如 30-50MB) | [^2][^17][^33][^61][^71] |

### YouTube 的特例

使用者可能會觀察到 YouTube 連結在 Discord 中似乎是使用了 YouTube 的播放器。這並非因為 Discord 支援 YouTube 的 `<iframe>`，而是 Discord 平台本身對 YouTube、Twitter 等大型主流服務進行了特殊的白名單整合 [^11][^16][^80]。Discord 內部有專門的程式碼來處理這些特定來源的連結，從而提供更豐富的整合體驗，例如「Watch Together」功能 [^9]。這項能力並未對一般開發者開放。

## 第三部分：在 Discord 中實現豐富影片體驗的替代方案

儘管無法嵌入完全客製化的播放器，開發者仍有多種方法可以在 Discord 中提供優質的影片分享體驗。

### 方案一：使用 Open Graph 呈現原生播放

這是最直接且符合 Discord 規範的方法。您需要：

1. 將您的影片檔案（如 `video.mp4`）上傳到一個可公開存取的伺服器或 CDN。
2. 建立一個網頁，並在 `<head>` 中設定完整的 Open Graph 標籤，確保 `og:video` 指向您影片檔案的直接 URL [^6][^94]。
3. 當您將此網頁的連結分享到 Discord 時，Discord 將抓取 OG 資訊，並顯示一個內嵌的原生播放器，使用者可以直接在聊天視窗中觀看影片。

### 方案二：利用 Webhooks 與 Rich Embeds

Discord 的 Webhooks 是一個強大的工具，可以讓外部應用程式向 Discord 頻道發送訊息 [^7][^93]。與普通訊息不同，透過 Webhooks 可以發送結構化的「Rich Embeds」訊息 [^66]。

一個 Rich Embed 可以包含：

* 標題、描述、作者、註腳 [^7][^12]
* 左側的彩色邊條 [^93]
* 縮圖和大型圖片 [^93]
* 多個並排的欄位 (Fields) [^7]

雖然 Rich Embed 本身不能播放影片，但您可以設計一個非常精美的預覽卡片，包含影片的縮圖、標題、簡介，並將標題或圖片連結到您自己網站上託管的客製化播放器頁面。這能提供比單純連結更吸引人的視覺效果。您可以使用像 Discohook 這類的線上工具來輕鬆產生 Rich Embed 所需的 JSON 結構 [^93]。

### 方案三：開發 Discord Bot

若需要更高的互動性，開發一個 Discord Bot 是最佳選擇。機器人可以：

* 監聽特定指令，然後回覆帶有影片連結的 Rich Embed 訊息 [^21][^70]。
* 使用按鈕（Buttons）和下拉選單（Select Menus）等互動元件，讓使用者選擇影片或進行其他操作。
* 實現更複雜的功能，例如將 YouTube 或其他來源的影片音訊串流到語音頻道中 [^13][^72]。

### 方案四：整合第三方自動化服務

對於不想自行編寫程式碼的使用者，可以利用 IFTTT 或 Zapier 等自動化平台 [^9][^14][^63][^93]。這些服務可以作為橋樑，將您的應用程式（例如，當您在某個平台上傳新影片時）與 Discord 連接起來。您可以設定一個自動化流程，當新影片發布時，自動透過 Webhook 在指定的 Discord 頻道中發布一條帶有影片連結的訊息 [^14][^68]。

## 結論

身為網頁工程師，在您完全控制的網站環境中，利用 HTML5 和 JavaScript 打造一個功能獨特、介面精美的客製化影音播放器是完全可行的。然而，將此客製化體驗「平移」到 Discord 這樣的第三方應用程式中則會遇到根本性的平台限制。

Discord 的安全模型不允許嵌入任意的 `<iframe>` 或外部腳本，其影片播放功能完全依賴其內建播放器，並透過 Open Graph 協定中的直接影片檔案連結來觸發。YouTube 等服務在 Discord 中的特殊呈現方式是平台級別的整合，而非一項通用技術。

因此，您的終極目標需要調整策略。最佳實踐路徑是：

1. **遵循平台規範**：在您的網頁上正確設定 Open Graph 元標籤，並提供影片檔案的直接連結，以確保在 Discord 中能實現基本的內嵌播放。
2. **增強視覺呈現**：利用 Webhooks 或 Discord Bot 發送精心設計的 Rich Embed 訊息，以一個引人注目的卡片形式來介紹您的影片。
3. **引導至完整體驗**：在 Rich Embed 中嵌入連結，將使用者導向您自己的網站，讓他們在那裡體驗您所打造的、功能完整的客製化播放器。

透過這種方式，您既能遵守 Discord 的生態規則，又能為使用者提供通往您獨特影音體驗的入口，實現兩全其美的效果。

[^2]: [Why does embedding video only work for a set of whitelisted ...](https://www.reddit.com/r/discordapp/comments/v9ltnz/why_does_embedding_video_only_work_for_a_set_of/)
[^5]: [What's with the custom video background upload limit? - Reddit](https://www.reddit.com/r/discordapp/comments/1as1hjs/whats_with_the_custom_video_background_upload/)
[^6]: [How to embed videos in Discord - Stack Overflow](https://stackoverflow.com/questions/74394473/how-to-embed-videos-in-discord)
[^7]: [Using Webhooks and Embeds - Discord](https://discord.com/safety/using-webhooks-and-embeds)
[^9]: [How to show your watching Youtube on Discord - IFTTT](https://ifttt.com/explore/how-to-show-youtube-on-discord)
[^11]: [Video embeds in Discord · Issue #5040 · Chocobozzz/PeerTube](https://github.com/Chocobozzz/PeerTube/issues/5040)
[^12]: [Embeds | discord.js Guide](https://discordjs.guide/popular-topics/embeds)
[^13]: [Video streaming bot for Discord - aixxe](https://aixxe.net/2021/04/discord-video-bot)
[^14]: [api.video Discord Integration - Quick Connect - Zapier](https://zapier.com/apps/apivideo/integrations/discord)
[^15]: [Need A Video Compressor for Discord? A Complete Guide](https://repairit.wondershare.com/video-repair/discord-video-compressor.html)
[^16]: [How does Discord embed YouTube-Playable Videos?](https://stackoverflow.com/questions/64190002/how-does-discord-embed-youtube-playable-videos)
[^17]: [Video embed size limit - Discord Support](https://support.discord.com/hc/en-us/community/posts/9164583446807-Video-embed-size-limit)
[^20]: [How It All Goes Live: An Overview of Discord's Streaming ...](https://discord.com/blog/how-it-all-goes-live-an-overview-of-discords-streaming-technology)
[^21]: [How to Embed Reddit Video in Discord - Hollyland](https://www.hollyland.com/blog/topics/how-to-embed-reddit-video-in-discord)
[^22]: [Complete Guide to Send a Playable Video on Discord - AnyMP4](https://www.anymp4.com/repair-video/send-playable-videos-on-discord.html)
[^32]: [What Is Best Video Compressor for Discord as of 2025](https://www.videoproc.com/video-editor/video-compressor-for-discord.htm)
[^33]: [Embed videos larger than 100MB in size - Discord Support](https://support.discord.com/hc/en-us/community/posts/10859755412759-Embed-videos-larger-than-100MB-in-size)
[^35]: [嵌入视频和播放列表- YouTube帮助](https://support.google.com/youtube/answer/171780?hl=zh-Hans)
[^37]: [Embed videos & playlists - YouTube Help](https://support.google.com/youtube/answer/171780?hl=en)
[^51]: [YouTube Player API Reference for iframe Embeds](https://developers.google.com/youtube/iframe_api_reference)
[^52]: [网站嵌入YouTube视频原创 - CSDN博客](https://blog.csdn.net/zhongjie19/article/details/121820608)
[^53]: [Embedding YouTube Videos in WordPress: Best Practices](https://getdevdone.com/blog/embedding-youtube-videos-in-wordpress-best-practices.html)
[^54]: [YouTube Player API Reference for iframe Embeds](https://developers.google.com/youtube/iframe_api_reference?hl=zh-tw)
[^58]: [嵌入视频和播放列表 - YouTube帮助](https://support.google.com/youtube/answer/171780?hl=zh-Hans)
[^59]: [YouTube Player API Reference for iframe Embeds | YouTube IFrame Player API | Google for Developers](https://developers.google.com/youtube/iframe_api_reference?hl=zh-tw)
[^61]: [Why does embedding video only work for a set of whitelisted ...](https://www.reddit.com/r/discordapp/comments/v9ltnz/why_does_embedding_video_only_work_for_a_set_of/)
[^63]: [api.video Discord Integration - Quick Connect - Zapier](https://zapier.com/apps/apivideo/integrations/discord)
[^64]: [What's with the custom video background upload limit? - Reddit](https://www.reddit.com/r/discordapp/comments/1as1hjs/whats_with_the_custom_video_background_upload/)
[^66]: [Using Webhooks and Embeds - Discord](https://discord.com/safety/using-webhooks-and-embeds)
[^68]: [How to show your watching Youtube on Discord - IFTTT](https://ifttt.com/explore/how-to-show-youtube-on-discord)
[^70]: [How to Embed Reddit Video in Discord - Hollyland](https://www.hollyland.com/blog/topics/how-to-embed-reddit-video-in-discord)
[^71]: [Video embed size limit - Discord Support](https://support.discord.com/hc/en-us/community/posts/9164583446807-Video-embed-size-limit)
[^72]: [Video streaming bot for Discord - aixxe](https://aixxe.net/2021/04/discord-video-bot)
[^76]: [Complete Guide to Send a Playable Video on Discord - AnyMP4](https://www.anymp4.com/repair-video/send-playable-videos-on-discord.html)
[^80]: [How does Discord embed YouTube-Playable Videos?](https://stackoverflow.com/questions/64190002/how-does-discord-embed-youtube-playable-videos)
[^90]: [Video embeds in Discord · Issue #5040 · Chocobozzz/PeerTube](https://github.com/Chocobozzz/PeerTube/issues/5040)
[^91]: [What Is Best Video Compressor for Discord as of 2025](https://www.videoproc.com/video-editor/video-compressor-for-discord.htm)
[^92]: [Discord Patch Notes: July 1, 2024](https://discord.com/blog/discord-patch-notes-july-1-2024)
[^93]: [Using Webhooks and Embeds](https://discord.com/safety/using-webhooks-and-embeds)
[^94]: [facebook opengraph - How to embed videos in Discord - Stack Overflow](https://stackoverflow.com/questions/74394473/how-to-embed-videos-in-discord)

{% chat(speaker="jim") %}
解釋更多有關於 oEmbed 的背景和實作範例
{% end %}

{% chat(speaker="claude") %}
oEmbed 是一個在 2008 年推出的開放標準，由 Cal Henderson（Flickr）和 Mike Malone（SimpleGeo）共同設計。它解決了一個核心問題：如何讓網站能夠安全且標準化地嵌入來自其他平台的豐富媒體內容。
{% end %}

## oEmbed 背景

### 設計理念

* **消費者導向**：重點是讓嵌入方（consumer）能夠輕鬆獲取嵌入代碼
* **提供者友善**：讓內容提供者（provider）能控制自己內容的呈現方式
* **安全性**：避免直接嵌入未知的第三方代碼

### 支援的媒體類型

* `video`：影片內容
* `photo`：圖片
* `link`：一般連結預覽
* `rich`：豐富的 HTML 內容

## oEmbed 工作流程

1. **發現階段**：消費者檢查網頁的 `<link>` 標籤或使用已知的 oEmbed 端點
2. **請求階段**：向提供者的 oEmbed API 發送請求
3. **回應階段**：提供者返回 JSON/XML 格式的嵌入資訊
4. **渲染階段**：消費者使用回應資料渲染內容

## 實作範例

### 1. 提供者端實作

```js
// Express.js 實作 oEmbed 提供者
const express = require('express');
const app = express();

// 模擬的影片資料庫
const videos = {
  'abc123': {
    title: '範例影片標題',
    description: '這是一個範例影片的描述',
    thumbnail: 'https://example.com/thumb/abc123.jpg',
    duration: 180,
    width: 1280,
    height: 720,
    embedUrl: 'https://example.com/embed/abc123'
  }
};

// oEmbed 端點
app.get('/oembed', (req, res) => {
  const { url, format = 'json', maxwidth, maxheight } = req.query;
  
  // 解析 URL 獲取影片 ID
  const videoId = extractVideoId(url);
  const video = videos[videoId];
  
  if (!video) {
    return res.status(404).json({ error: '找不到影片' });
  }
  
  // 計算最適合的尺寸
  let { width, height } = calculateDimensions(
    video.width, 
    video.height, 
    maxwidth, 
    maxheight
  );
  
  const oembedResponse = {
    version: '1.0',
    type: 'video',
    title: video.title,
    author_name: '你的網站名稱',
    author_url: 'https://example.com',
    provider_name: '你的播放器',
    provider_url: 'https://example.com',
    thumbnail_url: video.thumbnail,
    thumbnail_width: 480,
    thumbnail_height: 360,
    width: width,
    height: height,
    html: `<iframe src="${video.embedUrl}" width="${width}" height="${height}" frameborder="0" allowfullscreen></iframe>`
  };
  
  if (format === 'xml') {
    res.set('Content-Type', 'application/xml');
    res.send(jsonToXml(oembedResponse));
  } else {
    res.set('Content-Type', 'application/json');
    res.json(oembedResponse);
  }
});

function extractVideoId(url) {
  // 假設 URL 格式：https://example.com/watch?v=abc123
  const match = url.match(/[?&]v=([^&]+)/);
  return match ? match[1] : null;
}

function calculateDimensions(originalWidth, originalHeight, maxWidth, maxHeight) {
  let width = originalWidth;
  let height = originalHeight;
  
  if (maxWidth && width > maxWidth) {
    height = height * (maxWidth / width);
    width = maxWidth;
  }
  
  if (maxHeight && height > maxHeight) {
    width = width * (maxHeight / height);
    height = maxHeight;
  }
  
  return { width: Math.round(width), height: Math.round(height) };
}

// 在 HTML 頁面中加入發現標籤
app.get('/watch', (req, res) => {
  const videoId = req.query.v;
  const video = videos[videoId];
  
  if (!video) {
    return res.status(404).send('影片不存在');
  }
  
  const html = `
    <!DOCTYPE html>
    <html>
    <head>
      <title>${video.title}</title>
      
      <!-- oEmbed 發現標籤 -->
      <link rel="alternate" type="application/json+oembed" 
            href="https://example.com/oembed?url=${encodeURIComponent(req.originalUrl)}&format=json" 
            title="${video.title}">
      
      <!-- Open Graph 標籤 -->
      <meta property="og:title" content="${video.title}">
      <meta property="og:description" content="${video.description}">
      <meta property="og:image" content="${video.thumbnail}">
      <meta property="og:video" content="${video.embedUrl}">
      <meta property="og:video:width" content="${video.width}">
      <meta property="og:video:height" content="${video.height}">
      <meta property="og:type" content="video.other">
    </head>
    <body>
      <h1>${video.title}</h1>
      <iframe src="${video.embedUrl}" width="800" height="450" frameborder="0" allowfullscreen></iframe>
      <p>${video.description}</p>
    </body>
    </html>
  `;
  
  res.send(html);
});

app.listen(3000, () => {
  console.log('oEmbed 提供者運行在 port 3000');
});
```

### 2. 消費者端實作

```js
// oEmbed 消費者實作
class OEmbedConsumer {
  constructor() {
    // 已知的 oEmbed 提供者端點
    this.providers = {
      'youtube.com': 'https://www.youtube.com/oembed',
      'vimeo.com': 'https://vimeo.com/api/oembed.json',
      'example.com': 'https://example.com/oembed' // 你的自訂提供者
    };
  }

  async embedUrl(url, options = {}) {
    try {
      // 步驟 1: 嘗試從已知提供者獲取
      const knownProvider = this.findKnownProvider(url);
      if (knownProvider) {
        return await this.fetchFromProvider(knownProvider, url, options);
      }

      // 步驟 2: 嘗試自動發現
      const discoveredEndpoint = await this.discoverOEmbedEndpoint(url);
      if (discoveredEndpoint) {
        return await this.fetchFromProvider(discoveredEndpoint, url, options);
      }

      throw new Error('找不到 oEmbed 提供者');
    } catch (error) {
      console.error('oEmbed 嵌入失敗:', error);
      return null;
    }
  }

  findKnownProvider(url) {
    const hostname = new URL(url).hostname;
    const domain = hostname.replace('www.', '');
    return this.providers[domain];
  }

  async fetchFromProvider(providerUrl, url, options) {
    const params = new URLSearchParams({
      url: url,
      format: 'json'
    });

    // 加入可選參數
    if (options.maxwidth) params.append('maxwidth', options.maxwidth);
    if (options.maxheight) params.append('maxheight', options.maxheight);

    const response = await fetch(`${providerUrl}?${params}`);
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    return await response.json();
  }

  async discoverOEmbedEndpoint(url) {
    try {
      const response = await fetch(url);
      const html = await response.text();
      
      // 解析 HTML 尋找 oEmbed 連結
      const parser = new DOMParser();
      const doc = parser.parseFromString(html, 'text/html');
      
      const oembedLink = doc.querySelector(
        'link[type="application/json+oembed"]'
      );
      
      return oembedLink ? oembedLink.href : null;
    } catch (error) {
      console.error('自動發現失敗:', error);
      return null;
    }
  }

  async renderEmbed(url, containerId, options = {}) {
    const embedData = await this.embedUrl(url, options);
    const container = document.getElementById(containerId);
    
    if (!embedData || !container) {
      container.innerHTML = '<p>無法載入嵌入內容</p>';
      return;
    }

    // 根據類型渲染不同內容
    switch (embedData.type) {
      case 'video':
      case 'rich':
        container.innerHTML = embedData.html;
        break;
      
      case 'photo':
        container.innerHTML = `
          <img src="${embedData.url}" 
               alt="${embedData.title || ''}"
               width="${embedData.width || 'auto'}"
               height="${embedData.height || 'auto'}">
        `;
        break;
      
      case 'link':
        container.innerHTML = `
          <div class="link-preview">
            ${embedData.thumbnail_url ? 
              `<img src="${embedData.thumbnail_url}" alt="預覽圖">` : ''}
            <h3>${embedData.title}</h3>
            <p>${embedData.description || ''}</p>
            <a href="${url}" target="_blank">查看原始連結</a>
          </div>
        `;
        break;
    }
  }
}

// 使用範例
async function demonstrateOEmbed() {
  const consumer = new OEmbedConsumer();
  
  // 範例 1: 嵌入 YouTube 影片
  try {
    const youtubeData = await consumer.embedUrl(
      'https://www.youtube.com/watch?v=dQw4w9WgXcQ',
      { maxwidth: 560, maxheight: 315 }
    );
    console.log('YouTube oEmbed 資料:', youtubeData);
  } catch (error) {
    console.error('YouTube 嵌入失敗:', error);
  }
  
  // 範例 2: 嵌入自訂影片
  try {
    const customData = await consumer.embedUrl(
      'https://example.com/watch?v=abc123',
      { maxwidth: 800 }
    );
    console.log('自訂影片 oEmbed 資料:', customData);
  } catch (error) {
    console.error('自訂影片嵌入失敗:', error);
  }
  
  // 範例 3: 直接渲染到 DOM
  await consumer.renderEmbed(
    'https://example.com/watch?v=abc123',
    'video-container',
    { maxwidth: 640, maxheight: 360 }
  );
}

// Discord Bot 整合範例
class DiscordOEmbedBot {
  constructor(consumer) {
    this.oembed = consumer;
  }

  async handleMessage(message) {
    const urlRegex = /(https?:\/\/[^\s]+)/g;
    const urls = message.content.match(urlRegex);
    
    if (!urls) return;

    for (const url of urls) {
      try {
        const embedData = await this.oembed.embedUrl(url, {
          maxwidth: 400,
          maxheight: 300
        });

        if (embedData) {
          // 建立 Discord 嵌入訊息
          const discordEmbed = {
            title: embedData.title,
            description: embedData.description,
            url: url,
            thumbnail: embedData.thumbnail_url ? {
              url: embedData.thumbnail_url
            } : undefined,
            author: {
              name: embedData.author_name || embedData.provider_name,
              url: embedData.author_url || embedData.provider_url
            }
          };

          // 發送嵌入訊息（這裡需要 Discord.js 的實際實作）
          console.log('Discord 嵌入預覽:', discordEmbed);
        }
      } catch (error) {
        console.error(`處理 URL ${url} 失敗:`, error);
      }
    }
  }
}

// 初始化
const oembedConsumer = new OEmbedConsumer();
const discordBot = new DiscordOEmbedBot(oembedConsumer);

// 匯出供使用
if (typeof module !== 'undefined' && module.exports) {
  module.exports = { OEmbedConsumer, DiscordOEmbedBot };
}
```

## 重要的實作細節

### 安全性考量

* 驗證來源 URL 避免 SSRF 攻擊
* 限制回應大小防止 DoS
* 過濾 HTML 內容避免 XSS
* 實作速率限制

### 效能最佳化

* 快取 oEmbed 回應減少重複請求
* 設置合理的超時時間
* 非同步處理避免阻塞

### 相容性

* 同時支援 JSON 和 XML 格式
* 實作自動發現機制
* 優雅降級處理失敗情況

這個實作提供了完整的 oEmbed 生態系統，你可以根據需求調整和擴展。需要我解釋任何特定的部分嗎？
