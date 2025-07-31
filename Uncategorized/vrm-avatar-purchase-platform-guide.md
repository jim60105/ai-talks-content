+++
title = "VRM 格式人物模型購買與流通平台全攻略"
description = "想入手 VRM 格式的人物模型嗎？本文介紹 VRM 格式的特色、流通平台與購買管道，並帶你了解初學者常用的創作工具與注意事項。掌握這些資訊，輕鬆踏入虛擬化身世界！"
date = "2025-05-06T04:49:18Z"
updated = "2025-07-27T16:18:37.516Z"
draft = false

[taxonomies]
tags = [ "Prompt Engineering" ]
providers = [ "Felo Search" ]

[extra]
withAI = "<https://felo.ai/search/W7EqYyGuRoMFoQsTN6uS2U?invite=dOLYGeJyZJqVX>"
featured = true
+++
{% alert(edit=true) %}
在接觸一個全新的領域時，透過 AI 可以很快速的掌握關鍵入門知識。

可以注意我的提問實施了 CO-STAR 提示詞技巧。  
(其中的 Style 和 Tone 由於 Felo Search Pro 的研究模式本身已經有足夠的設計，可以不需要額外指定。)
{% end %}

{% alert(edit=true) %}
對了，我後來並沒有怎麼玩 VRChat，我社恐 😶‍🌫️
{% end %}

{% chat(speaker="jim") %}
我剛開始接觸 VRChat，希望在網路商城購買一個人物模型。請你給我一個初學者介紹，包含從在哪裡購買，常見的檔案格式，使用什麼工具轉檔匯出，匯入 VRChat 等等的詳細流程以及基礎概念知識。
{% end %}

{% chat(speaker="felo") %}
根據搜尋結果，我將詳細分析相關問題。
{% end %}

## Overview

本報告為 VRChat 初學者提供一份詳細指南，說明如何從網路商城購買人物模型 (Avatar) 並將其匯入 VRChat 使用。內容涵蓋了購買模型的管道、常見檔案格式、必要的軟體工具、詳細的匯入與上傳步驟，以及相關的基礎概念知識，旨在幫助新手順利擁有並使用自訂的虛擬化身。核心流程包括：註冊 VRChat 帳號並達到上傳權限、安裝 VRChat Creator Companion (VCC) 與指定版本的 Unity、透過 VCC 建立專案、依序匯入所需插件 (如動態骨骼、著色器) 與模型檔案、使用 VRChat SDK 控制面板設定並上傳模型。

## Detailed Report

### 基礎概念 (Basic Concepts)

在開始之前，理解一些 VRChat 開發的基本術語會很有幫助：

* **Avatar (虛擬化身):** 您在 VRChat 中使用的角色模型 [^114]。
* **World (世界):** VRChat 中的虛擬環境或地圖 [^31][^60]。
* **SDK (Software Development Kit):** VRChat 提供的軟體開發工具包，用於在 Unity 中創建 Avatars 和 Worlds [^1][^26][^46][^56][^70]。目前 Avatar 主要使用 SDK3 [^5][^6][^12][^27][^58][^76][^77]。
* **VCC (VRChat Creator Companion):** 官方推薦的管理工具，用於安裝和管理 Unity 版本、VRChat SDK 以及創建專案，能大幅簡化設定流程 [^11][^19][^53][^64][^67][^101][^102][^111][^112][^119][^121]。
* **Unity:** 一款強大的遊戲引擎，是創建和上傳 VRChat 內容的主要平台 [^11][^19][^100][^101][^111]。VRChat 需要使用特定版本的 Unity [^11][^100][^103][^104][^105]。
* **Shader (著色器):** 決定模型外觀顯示效果的程式碼，例如卡通風格或寫實風格 [^8][^13][^32][^48][^49][^51][^63][^75][^98][^100]。常見的 VRChat Shader 有 lilToon [^13][^100]、UTS2 [^6][^49][^99]、Arktoon Shaders [^32][^51][^98] 等。
* **Prefab (預製件):** 在 Unity 中預先配置好的物件，VRChat 模型通常會提供 Prefab 檔案，方便使用者直接載入和上傳 [^6][^32][^45][^49][^51][^79][^89][^98][^99][^100]。
* **.unitypackage:** Unity 用來打包資源 (模型、貼圖、腳本等) 的壓縮檔案格式，便於分發和匯入 [^1][^6][^26][^47][^49][^51][^56][^68][^98][^99]。
* **FBX:** 常見的 3D 模型檔案格式，被 Unity 廣泛支援 [^23][^29][^35][^57][^100]。
* **VRM:** 一種標準化的人形 3D 虛擬化身檔案格式，常由 VRoid Studio 等軟體產生。匯入 VRChat 需要透過 Unity 插件 (如 UniVRM) 進行轉換 [^2][^17][^27][^39][^54][^58][^85][^95]。
* **Dynamic Bone (動態骨骼):** 一款常見的付費 Unity 插件，用於模擬頭髮、尾巴、衣物等的物理擺動效果 [^6][^8][^32][^49][^51][^63][^89][^98][^99][^100]。許多販售的模型會需要此插件。

### 購買模型 (Purchasing Models)

有多種管道可以獲取 VRChat 模型：

* **Booth.pm:** 日本最大的創作者市場之一，提供大量高品質的 VRChat Avatar 和相關配件，尤其以日系動漫風格為主 [^5][^8][^63][^98][^100]。模型通常以 `.unitypackage` 格式提供，購買前需注意作者指定的 Unity 版本、必要的 Shader 和是否需要 Dynamic Bone [^98][^100]。務必確認並遵守作者的使用條款 (Terms of Use/利用規約)。
* **Unity Asset Store:** Unity 官方的資源商店，也提供一些 VRChat 可用的模型和資源 [^100]。
* **淘寶/閒魚:** 中國的電商平台，可以找到一些 VRChat 模型，價格可能較低。但需注意模型可能非最新版本，且部分賣家可能要求提供 VRChat 帳號密碼代為上傳，這存在安全風險 [^100]。建議優先選擇提供模型檔案自行上傳的賣家。
* **遊戲內複製:** VRChat 允許玩家在遊戲中直接複製他人設定為「公開 (Public)」的 Avatar。這是最簡單的入門方式，但無法進行修改 [^11][^100]。

### 準備工作 (Preparation)

在購買模型並準備上傳前，需要完成以下準備：

**帳號與權限 (Account and Permissions):**

1. **註冊 VRChat 帳號:** 必須擁有 VRChat 官方網站註冊的帳號，而非僅使用 Steam 帳號登入 [^1][^26][^56][^100]。
2. **提升帳號等級:** 上傳 Avatar 需要至少達到「New User」等級 (帳號名稱顯示為藍色) [^11][^100][^114]。提升等級的方式包括：累積遊戲時間、增加好友數量、與好友互動、探索不同世界等 [^11][^27][^58][^99]。

**安裝必要軟體 (Installing Necessary Software):**

1. **安裝 VRChat Creator Companion (VCC):** 前往 VRChat 官網下載並安裝 VCC [^11][^111][^112]。這是官方推薦的工具，能協助管理後續步驟。
2. **安裝 Unity Hub:** 前往 Unity 官網下載並安裝 Unity Hub，用於管理不同版本的 Unity 編輯器 [^11][^19][^41][^66][^99][^101][^102]。
3. **安裝指定 Unity 版本:** 開啟 VCC，它會提示您安裝 VRChat 當前支援的 Unity 版本 (截至 2025 年 5 月，通常是 2022.3.x 的某個版本，具體版本請以 VCC 或 VRChat 官方文件為準 [^104][^111])。VCC 會引導您透過 Unity Hub 下載並安裝正確版本 [^11][^19][^101][^102]。請勿自行安裝最新版本或其他版本的 Unity，除非 VRChat 官方明確支援 [^87][^95]。
4. **了解 VRChat SDK:** VCC 在建立專案時會自動導入對應的 VRChat SDK (目前為 SDK3 for Avatars) [^11][^64][^101][^111]。

### 匯入與設定流程 (Import and Setup Process)

**建立 Unity 專案 (Creating a Unity Project):**

1. 開啟 VCC。
2. 點擊 "Create New Project"。
3. 選擇 "Avatars" 模板 [^11][^100]。
4. 設定專案名稱和儲存路徑，點擊 "Create Project" [^11][^100]。VCC 會自動設定好 Unity 版本並導入基礎的 VRChat SDK。

**匯入資源 (Importing Assets):**

在 Unity 專案中，需要按特定順序匯入購買模型所需的插件和模型檔案，錯誤的順序可能導致問題 [^6][^49][^99]。

1. **(若需要) 匯入 Dynamic Bone:** 如果購買的模型需要 Dynamic Bone，請先將 Dynamic Bone 的 `.unitypackage` 檔案拖入 Unity 視窗下方的 Assets 區域，或透過上方選單 `Assets` > `Import Package` > `Custom Package...` 來匯入 [^6][^8][^32][^49][^51][^63][^89][^98][^99]。**必須在匯入模型前完成此步驟** [^8][^32][^51][^63][^89][^98][^99]。
2. **匯入必要 Shader:** 將模型指定的 Shader (例如 lilToon) 的 `.unitypackage` 檔案匯入 Unity [^6][^8][^32][^49][^51][^63][^98][^99][^100]。部分模型可能已將 Shader 包裝在模型檔案內 [^32][^51][^98]。
3. **匯入模型檔案:** 最後，將購買的模型主體 `.unitypackage` 檔案匯入 Unity [^6][^32][^47][^48][^49][^51][^68][^75][^98][^99]。

**載入模型 (Loading the Model):**

1. 在 Unity 的 Assets 視窗中，找到匯入的模型資料夾。
2. 尋找模型的主要 Prefab 檔案 (圖示通常是一個藍色方塊，檔案名稱可能包含模型名，結尾為 `.prefab`) 或 Scene 檔案 (圖示類似 Unity logo，結尾為 `.unity`) [^6][^32][^45][^49][^51][^98][^99]。
3. 將 Prefab 檔案拖曳到左側的 Hierarchy 視窗中，或雙擊 Scene 檔案來載入 [^6][^32][^45][^49][^51][^98][^99][^100]。模型應該會出現在中間的 Scene 視圖中。

**(可選) VRM 模型轉換 ((Optional) VRM Model Conversion):**

如果您購買或製作的是 VRM 格式的模型，您需要先在 Unity 專案中安裝 VRM 轉換插件 (例如 UniVRM)，然後透過該插件將 VRM 檔案匯入，轉換成 VRChat 可用的格式，再進行後續設定與上傳 [^2][^17][^27][^58][^95]。

### 上傳至 VRChat (Uploading to VRChat)

**開啟 SDK 控制面板 (Opening SDK Control Panel):**

1. 在 Unity 上方選單列，點擊 `VRChat SDK` > `Show Control Panel` [^1][^6][^11][^26][^31][^32][^45][^49][^51][^56][^60][^98][^99][^100]。

**檢查與修復 (Checking and Fixing):**

1. **登入:** 在控制面板中輸入您的 VRChat 帳號和密碼登入 [^1][^6][^11][^26][^31][^32][^45][^49][^51][^56][^60][^98][^99][^100]。面板會顯示您的帳號是否有上傳權限 [^11][^100]。
2. **切換到 Builder 標籤:** 點擊控制面板上方的 "Builder" 標籤 [^6][^32][^45][^49][^51][^98][^99][^100]。
3. **自動修復:** 面板可能會顯示一些警告或錯誤訊息 (通常是紅色或黃色的圖示)。如果看到旁邊有 "Auto Fix" 按鈕，請點擊它讓 SDK 自動嘗試修復問題 [^49][^51][^98][^100]。重複此步驟直到沒有 Auto Fix 按鈕出現。

**設定模型資訊 (Configuring Avatar Information):**

1. **名稱與描述:** 填寫 "Avatar Name" (必填) 和 "Description" (選填) [^6][^32][^51][^98][^99][^100]。
2. **縮圖:** 點擊 "Thumbnail" 旁邊的相機圖示或按鈕，可以從 Scene 視圖截取或選擇一張圖片作為模型的預覽圖 [^13][^59][^100]。
3. **可見性:** 在 "Sharing" 或 "Visibility" 選項中選擇 "Private" (私有，只有自己能用，他人無法複製) 或 "Public" (公開，他人可在遊戲中複製使用) [^32][^51][^98][^99][^100]。新手建議先設為 Private。

**建置與發布 (Building and Publishing):**

1. **確認同意:** 勾選上傳按鈕上方的確認框，表示您已閱讀並同意相關條款 [^32][^51][^98][^99][^100]。
2. **點擊上傳:** 點擊 "Build & Publish for Windows" 按鈕 [^6][^32][^45][^49][^51][^98][^99][^100]。Unity 會開始編譯模型並將其上傳到 VRChat 伺服器。這個過程可能需要幾分鐘或更長時間，取決於模型複雜度和網路速度。
3. **完成:** 上傳成功後會顯示 "Upload successful" 的訊息 [^51][^98]。之後您就可以在 VRChat 遊戲內的 Avatar 選單中找到並使用您剛上傳的模型了。

### 常見問題與疑難排解 (Common Issues and Troubleshooting)

* **無法上傳 (按鈕灰色或錯誤):**
  * 檢查 Unity 版本是否為 VRChat 指定版本 [^87][^95]。
  * 檢查 VRChat 帳號等級是否達到 New User [^11][^100]。
  * 檢查是否遺漏必要的 Shader 或 Dynamic Bone [^89][^98][^100]。
  * 檢查資源匯入順序是否正確 [^6][^49][^99]。
  * 點擊 SDK 控制面板 Builder 標籤中的 "Auto Fix" 按鈕 [^49][^51][^98][^100]。
  * 嘗試刪除藍圖 ID (Pipeline Manager -> Detach) [^88][^99]。
  * 檢查網路連線，上傳過程可能需要穩定的網路，部分地區可能需要 VPN/加速器 [^99][^100]。
* **模型顯示為粉紅色:** 通常表示缺少對應的 Shader 或 Shader 未正確設定 [^98][^100]。請確認已匯入模型所需的 Shader，並在 Unity 的材質 (Material) 設定中選擇正確的 Shader。
* **動態效果 (頭髮/裙子) 不動:** 確認已購買並正確匯入 Dynamic Bone 插件 [^98][^100]。檢查模型 Prefab 上是否有 Missing Script (腳本遺失) 的錯誤，若有則可能需要移除該腳本或確保 Dynamic Bone 已安裝 [^32][^98]。
* **VCC/Unity 錯誤:** 嘗試重新啟動 VCC 和 Unity。若問題持續，可嘗試刪除專案中的 `Library` 資料夾讓 Unity 重建。作為最後手段，可考慮重新安裝 VCC 或 Unity [^99]。

建議新手在遇到問題時，多查閱教學影片 [^7][^9][^18][^21][^50][^52][^55][^97]、官方文件 [^10][^36][^64][^69][^70][^71][^103][^104][^105][^111][^114][^117][^120] 或相關社群尋求幫助。

{% chat(speaker="jim") %}
再重點介紹一下 VRM 格式，自己這個格式在哪裡流通，在哪裡購買這個格式的人物。
{% end %}

{% chat(speaker="felo") %}
根據搜尋結果，我將詳細分析相關問題。
{% end %}

## Overview

VRM 是一種專為 3D 人形虛擬化身 (Avatar) 設計的標準化、跨平台檔案格式，基於 glTF 2.0 技術建構 [^1004][^1010][^1038]。其核心目標是解決不同應用程式和平台間 3D 模型不兼容、資訊缺乏的問題，讓使用者能在各種 VR/AR 應用、VTuber 直播軟體和元宇宙平台中無縫使用同一個化身 [^1001][^1010][^1013][^1037][^1038]。VRM 格式不僅包含模型的幾何、貼圖、骨骼等基本資訊，還標準化了表情控制 (BlendShapes)、視線追蹤、物理擺動效果 (如頭髮) 以及重要的元數據，特別是化身的使用權限和許可證資訊 [^1001][^1004][^1028][^1037][^1039]。該格式由 VRM Consortium 推動發展，目前有 VRM 0.x 和較新的 VRM 1.0 版本 [^1004][^1008][^1039]。VRM 模型廣泛流通於 VTuber 社群、VR 平台 (如 VRChat，但需轉換) 和 3D 內容創作工具鏈中 [^1003][^1009][^1013][^1038]。使用者可以透過 Booth.pm、VRoid Hub 等平台購買或下載 VRM 模型，也可以使用 VRoid Studio 等工具自行創作，或委託創作者訂製 [^1002][^1005][^1009][^1054]。

## Detailed Report

### VRM 格式詳解 (VRM Format Explained)

VRM (Virtual Reality Model) 是一種開放原始碼、免版稅的 3D 虛擬化身檔案格式，專為處理 VR 應用中的人形 3D 模型而設計 [^1004][^1010][^1013][^1016][^1038]。它基於業界標準的 glTF 2.0 格式進行擴展，旨在提供一個平台無關的解決方案，讓同一個化身模型能在不同的軟體和服務間流通使用 [^1001][^1004][^1010][^1013][^1027][^1037][^1038]。

**核心特性:**

* **可攜性 (Portability):** VRM 將模型所需的所有數據，包括網格 (Mesh)、貼圖 (Textures)、材質 (Materials)、骨骼 (Bones) 等，打包在一個單一的 `.vrm` 檔案中 (其底層結構兼容 `.glb`)，方便在不同應用程式中載入和使用 [^1004][^1019][^1028][^1037][^1038][^1039]。
* **標準化 (Standardization):**
  * *座標系統與單位:* 統一採用右手座標系、Y 軸向上 (Y-UP)、公制單位 (1.0 = 1 公尺)，並面向 Z+ 方向 (VRM 1.0)，解決了不同建模工具可能產生的差異 [^1001][^1004][^1028][^1037][^1039]。
  * *骨架與姿勢:* 採用 T-Pose 作為標準初始姿勢，骨架結構需符合 Unity Humanoid 標準，確保動作捕捉和動畫的兼容性 [^1004][^1028][^1037][^1038][^1039]。
  * *模型操作:* 標準化了模型的姿勢控制、表情控制 (Facial Expressions) 和視線追蹤 (Gaze Control) [^1001][^1028][^1037]。
* **表情與口型同步 (Expressions & Lip Sync):** VRM 定義了一套標準的表情混合形狀 (BlendShapes)，包括喜(Joy)、怒(Anger)、哀(Sorrow)、樂(Fun) 等基本情緒 (VRM 1.0 新增 Surprise)，以及用於口型同步的「A」「I」「U」「E」「O」元音發音嘴型和眨眼 (Blink) [^1001][^1014][^1028][^1037]。這使得應用程式可以透過標準 API 統一控制不同模型的表情和實現語音驅動的口型動畫 [^1003][^1014][^1028][^1037]。
* **視線控制 (Gaze Control):** 支援透過骨骼 (Bone)、混合形狀 (BlendShape) 或貼圖 UV (TextureUV) 三種方式來控制化身的眼睛視線方向 [^1001][^1028][^1037]。
* **材質系統 (Materials):** 除了支援 glTF 標準的 PBR (物理基礎渲染) 和 Unlit (無光照) 材質外，VRM 還定義了 MToon 材質，這是一種專為實現日系動畫風格 (Cel Shading / Toon Shading) 而設計的非真實感渲染 (NPR) 材質 [^1001][^1004][^1028][^1037][^1038][^1039]。
* **物理模擬 (Physics Simulation):** VRM 內建了標準的「搖晃物 (Spring Bone)」設定，可以模擬頭髮、尾巴、裙襬等物件的物理擺動效果，而無需依賴外部物理引擎 [^1001][^1004][^1028][^1037][^1038][^1039]。VRM 1.0 中新增了約束 (Constraint) 功能。
* **VR 特化設定 (VR Setup):** 包含用於 VR 環境下第一人稱視角重現的相關資訊，例如定義頭部位置以及在第一人稱視角下需要隱藏的模型部分 [^1001][^1004][^1028][^1037][^1038][^1039]。
* **元數據與授權 (Metadata & Licensing):** VRM 檔案可以嵌入模型的標題、作者、縮圖等元數據 [^1001][^1004][^1028][^1037][^1039]。更重要的是，它可以包含針對虛擬化身時代特別設計的授權資訊，例如：是否允許暴力或性描寫、是否允許營利使用、是否允許修改與再分發 (可對應創用 CC 等) 等，甚至定義了化身的「人格權」相關許可 [^1001][^1004][^1013][^1028][^1037][^1038][^1039]。這是 VRM 相較於一般 3D 模型格式的顯著特點。VRM Consortium 也定義了 VRM 公共授權文件 (VRM Public License) [^1004][^1013][^1038]。
* **版本演進:** 主要有 VRM 0.x 和 VRM 1.0 兩個版本 [^1004][^1008][^1039]。VRM 1.0 於 2022 年 9 月正式發布，是目前的官方標準，修正和改進了 0.x 版本的一些規格 (如座標朝向)，並提供了更好的長期兼容性潛力 [^1004][^1008][^1013][^1038][^1048]。雖然目前仍有許多應用支援 VRM 0.x，但建議優先使用或導出為 VRM 1.0 格式 [^1008][^1048]。
* **開發與推廣:** VRM 格式由日本多家企業聯合成立的 VRM Consortium 負責制定規範與推廣 [^1004][^1031][^1038][^1039]。該協會也提供 C# 語言的 Unity 標準實現 (UniVRM) 用於讀寫 VRM 檔案 [^1004][^1010][^1013][^1038]。近期 VRM Consortium 已與 Khronos Group (glTF 標準的制定者) 合作，推動 VRM 的國際標準化 [^1004][^1039]。

### VRM 的流通與應用 (Circulation and Applications)

VRM 格式因其標準化和針對虛擬化身的特性，在以下領域廣泛流通和應用：

* **VTuber (虛擬直播主) 領域:** VRM 是日本 VTuber 社群中興起並廣泛採用的格式 [^1003]。許多 VTuber 直播軟體原生支援 VRM，使用者可以直接匯入 VRM 模型進行面部捕捉和直播，例如：
  * VSeeFace [^1003]
  * LuppetX [^1009]
  * Vtuber Animator [^1011]
  * 以及其他眾多支援 OSC/VMC 協議的直播工具。
* **VR/AR 與元宇宙平台:** 許多虛擬實境和元宇宙平台支援使用者匯入自己的 VRM 化身，實現跨平台的身分一致性：
  * VRChat: 雖然 VRChat 本身主要使用 Unity 進行模型上傳，但可以透過 UniVRM 等工具將 VRM 模型轉換為 VRChat 可用的格式 [^1013][^1038]。
  * VIVERSE [^1032][^1043][^1061]
  * The Seed Online [^1013][^1038]
  * VRoid Hub (本身也是 VRM 模型分享平台) [^1009][^1013][^1038][^1061]
  * Webaverse [^1013][^1038]
  * Monaverse [^1013][^1038]
  * Somnium Space [^1013][^1038]
  * 其他支援 VRM 的遊戲或社交應用 [^1012][^1013][^1038]。
* **3D 內容創作與遊戲開發:**
  * *VRoid Studio:* Pixiv 開發的免費 3D 角色創建工具，專門用於製作 VRM 格式的模型，極大降低了 VRM 模型的創作門檻 [^1002][^1009][^1026][^1034][^1061]。
  * *Unity:* 透過官方提供的 UniVRM 插件，可以在 Unity 中匯入、編輯和匯出 VRM 模型，是 VRM 生態的核心工具之一 [^1003][^1004][^1010][^1013][^1038]。
  * *Blender:* 社群開發了 VRM 插件 (VRM Add-on for Blender)，允許在 Blender 中直接匯入和匯出 VRM 檔案，並設置 MToon 材質等 [^1013][^1016][^1036][^1038]。
  * *Unreal Engine (UE):* 也有開發者研究如何在 UE5 中使用 VRM 模型進行虛擬直播等應用 [^1002][^1058]。
  * *CLIP STUDIO PAINT:* 從 Ver.2.2 開始支援讀取 VRM 檔案作為 3D 人物素材，Ver.3.2 更支援讀取 VRM 內建的表情 [^1014]。
  * *其他工具:* RPG Developer Bakin [^1012]、RPG Maker Unite [^1013][^1038] 等遊戲開發工具也增加了對 VRM 的支援。

### 獲取 VRM 模型 (Acquiring VRM Models)

獲取 VRM 模型主要有以下幾種途徑：

* **線上模型商店/平台 (Marketplaces/Platforms):**
  * **Booth.pm:** 日本最大的創作者作品販售平台之一，有大量高品質的 3D 模型，其中許多是 VRM 格式或提供 VRM 檔案，特別是日系動漫風格 [^1005][^1009][^1058]。購買時需注意作者標示的模型格式 (有些可能主推 VRChat 格式，但也提供 VRM) 以及使用條款 [^1009][^1040][^1058][^1060]。
  * **VRoid Hub:** 由 Pixiv 營運，專門用於展示和分享使用 VRoid Studio 或其他工具創建的 VRM 模型的平台。使用者可以在此找到大量免費或標有特定授權的 VRM 模型供下載使用 [^1009][^1013][^1038][^1061]。
  * **Vket Store:** 主要販售 VRChat 相關商品，但也有部分模型會明確標示提供 VRM 格式檔案 [^1040][^1060]。
  * **Sketchfab:** 綜合性的 3D 模型平台，可以搜尋到一些標記為 VRM 格式的模型，部分可免費下載或付費購買 [^1041][^1042][^1047]。
  * **Etsy:** 部分創作者會在 Etsy 上販售自己製作的 VRoid 模型或 VRM 角色 [^1044][^1046]。
  * **淘寶 (Taobao) / 愛發電 (Afdian) 等:** 中國大陸的電商或創作平台也有販售 VRM 模型的賣家或創作者 [^1005][^1020][^1045][^1051][^1057][^1059]。購買時需留意模型來源、品質以及授權的清晰度。
  * **VIVERSE Avatar Market:** 提供免費 VRM 模型下載，並可匯入/匯出 VRM [^1032][^1043][^1061]。
* **自行創作 (Creation):**
  * **VRoid Studio:** 對初學者最友善的方式，透過類似「捏人」的直觀介面即可創建出完整的 VRM 模型 [^1002][^1009][^1026][^1034][^1061]。
  * **專業 3D 軟體 + Unity:** 使用 Blender、Maya 等專業 3D 建模軟體從頭創建模型，然後匯入 Unity，使用 UniVRM 插件進行骨骼綁定、表情設定、材質調整等，最終匯出為 VRM 格式 [^1002][^1009]。
* **委託訂製 (Commissions):**
  * 尋找 3D 模型師或專門製作 VTuber 模型的創作者進行付費訂製 [^1023][^1049][^1052][^1054][^1056]。可以在 Twitter、Clibo 同人委託平台 [^1049]、米畫師 [^1023][^1052]、Fiverr [^1054] 或其他自由工作者平台尋找合適的創作者。訂製費用會依模型複雜度和創作者經驗而有很大差異 [^1049][^1056]。

**注意事項:** 無論是購買、下載免費模型還是委託訂製，務必仔細閱讀並遵守模型作者設定的使用條款 (Terms of Use / 利用規約)，特別是關於是否允許修改、再分發、商業利用以及特定用途 (如暴力、成人內容) 的限制 [^1001][^1004][^1040][^1055][^1060][^1062]。VRM 格式內建的授權資訊欄位就是為了方便使用者確認這些規範 [^1001][^1004][^1013][^1028][^1037][^1039]。

[^1]: [VRChat简易教程1-开发环境准备（SDK） - 布兰姥爷- 博客园](https://www.cnblogs.com/cation/p/10311702.html)
[^2]: [【vrchat】五分钟教你把vrm模型导入vrchat！ - 哔哩哔哩](https://www.bilibili.com/video/BV1vL411371e/)
[^5]: [#vrchat #digitalart #3dmodeling #characterdesign #maplevra ...](https://www.linkedin.com/posts/romain-peiffer-792890b0_vrchat-digitalart-3dmodeling-activity-7218831160662007808-sX_2)
[^6]: [VRChat成品模型上传的小白说明书](https://www.bilibili.com/read/cv21531240/)
[^7]: [【VRCHAT】4分钟让你学会怎么上传模型](https://www.bilibili.com/video/av972364306/)
[^8]: [上传从Booth上购买来的模型- VRChat](https://steamcommunity.com/sharedfiles/filedetails/?l=vietnamese&id=2827364181&searchtext=Cerca+tra+le+guide+di+VRChat)
[^9]: [【VRCHAT】4分钟让你学会怎么上传模型](https://www.bilibili.com/video/BV1Pp4y1876t/)
[^10]: [欢迎！ - VRChat 汉化文档中心](https://docs.vrczh.org/creators.vrchat.com/getting-started)
[^11]: [VRChat日记：超简单获得角色模型/上传教程- 梦雨玲音](https://www.rainlain.com/index.php/2024/06/07/2277/)
[^12]: [【VRCHAT教程】VRC 3.0 Avatar模型全面详细功能解析专为萌 ...](https://www.bilibili.com/video/BV1R5411w7Ee/)
[^13]: [VRCHAT模型教程（上传改模）](https://www.bilibili.com/video/BV1Ve411g7ob/)
[^17]: [非常个人向的VRM转VRC作业流程Part1:基础准备【VRChat ...](https://www.bilibili.com/read/cv29343793)
[^18]: [VRChat模型上传教程超详细完全新手向](https://www.bilibili.com/video/BV13T41157GW/)
[^19]: [【VRChat 改模】开发环境搭建：VCC - CSDN博客](https://blog.csdn.net/qq_43768851/article/details/144017378)
[^21]: [【VRchat教程】只要几分钟你就可以学会做模型](https://www.bilibili.com/video/BV1Yt411o787/)
[^23]: [使用Clip Studio Modeler 時出現無回應錯誤使用Clip ... - clip studio ask](https://ask.clip-studio.com/zh-tw/detail?id=110530)
[^26]: [VRChat简易教程1-开发环境准备（SDK） 转载 - CSDN博客](https://blog.csdn.net/weixin_33913332/article/details/93531799)
[^27]: [【vrchat】如何将vrm模型导入vrchat？_哔哩哔哩](https://www.bilibili.com/video/BV1tS4y1A7jU/)
[^29]: [我想將3D 物件匯入Unity 並在VRchat 中使用它們 - clip studio ask](https://ask.clip-studio.com/zh-tw/detail?id=101577)
[^31]: [VRChat简易教程2-创建一个最基本的世界（world） - 博客园](https://www.cnblogs.com/cation/p/10311823.html)
[^32]: [【已过时】轻松简单自己上传VRChat的Avatar - 热心的博客](https://sunnyrx.com/2019/12/28/Easy-to-DIY-VRChat-Avatar/)
[^35]: [Search "FBX" - Plurk](https://testing.plurk.com/search?q=FBX)
[^36]: [角色 - VRChat Wiki](https://wiki.vrchat.com/wiki/Avatars/zh-hant)
[^39]: [【教學】VRM 介紹與教學(概觀)，面向VR應用的3D模型新格式](https://home.gamer.com.tw/creationDetail.php?sn=4075972)
[^41]: [VRChat 入门包 - VRCD 文档库](https://docs.vrcd.org.cn/books/vrchat)
[^45]: [VRChat成品模型的上传及Unity汉化](https://vrc.yexca.net/archives/19)
[^46]: [Public SDK API - VRChat 汉化文档中心](https://docs.vrczh.org/creators.vrchat.com/sdk/public-sdk-api)
[^47]: [2. 导入模型到Unity - VRCD 文档库](https://docs.vrcd.org.cn/books/vrchat/page/2-unity)
[^48]: [【VRChat · 改模】Unity工程导入人物模型；并添加着色器教程](https://blog.csdn.net/qq_43768851/article/details/145235492)
[^49]: [VRChat成品模型的上传及Unity汉化](https://vrc.yexca.net/archives/19)
[^50]: [【VRCHAT】4分钟让你学会怎么上传模型](https://www.bilibili.com/video/av972364306/)
[^51]: [【已过时】轻松简单自己上传VRChat的Avatar - 热心的博客](https://sunnyrx.com/2019/12/28/Easy-to-DIY-VRChat-Avatar/)
[^52]: [VRChat模型上传教程超详细完全新手向](https://www.bilibili.com/video/BV13T41157GW/)
[^53]: [VRChat日记：超简单获得角色模型/上传教程- 梦雨玲音](https://www.rainlain.com/index.php/2024/06/07/2277/)
[^54]: [【vrchat】五分钟教你把vrm模型导入vrchat！ - 哔哩哔哩](https://www.bilibili.com/video/BV1vL411371e/)
[^55]: [【VRchat教程】只要几分钟你就可以学会做模型](https://www.bilibili.com/video/BV1Yt411o787/)
[^56]: [VRChat简易教程1-开发环境准备（SDK） - 布兰姥爷- 博客园](https://www.cnblogs.com/cation/p/10311702.html)
[^57]: [我想將3D 物件匯入Unity 並在VRchat 中使用它們 - clip studio ask](https://ask.clip-studio.com/zh-tw/detail?id=101577)
[^58]: [【vrchat】如何将vrm模型导入vrchat？_哔哩哔哩](https://www.bilibili.com/video/BV1tS4y1A7jU/)
[^59]: [VRCHAT模型教程（上传改模）](https://www.bilibili.com/video/BV1Ve411g7ob/)
[^60]: [VRChat简易教程2-创建一个最基本的世界（world） - 博客园](https://www.cnblogs.com/cation/p/10311823.html)
[^63]: [上传从Booth上购买来的模型- VRChat](https://steamcommunity.com/sharedfiles/filedetails/?l=vietnamese&id=2827364181&searchtext=Cerca+tra+le+guide+di+VRChat)
[^64]: [欢迎！ - VRChat 汉化文档中心](https://docs.vrczh.org/creators.vrchat.com/getting-started)
[^66]: [VRChat 入门包 - VRCD 文档库](https://docs.vrcd.org.cn/books/vrchat)
[^67]: [【VRChat 改模】开发环境搭建：VCC - CSDN博客](https://blog.csdn.net/qq_43768851/article/details/144017378)
[^68]: [2. 导入模型到Unity - VRCD 文档库](https://docs.vrcd.org.cn/books/vrchat/page/2-unity)
[^69]: [角色 - VRChat Wiki](https://wiki.vrchat.com/wiki/Avatars/zh-hant)
[^70]: [Public SDK API - VRChat 汉化文档中心](https://docs.vrczh.org/creators.vrchat.com/sdk/public-sdk-api)
[^71]: [自此開始 - VRChat Wiki](https://wiki.vrchat.com/wiki/Getting_Started/zh-hant)
[^75]: [【VRChat · 改模】Unity工程导入人物模型；并添加着色器教程](https://blog.csdn.net/qq_43768851/article/details/145235492)
[^76]: [【VRCHAT教程】VRC 3.0 Avatar模型全面详细功能解析专为萌 ...](https://www.bilibili.com/video/BV1R5411w7Ee/)
[^77]: [【心得】Avatar 3.0 教學圖文(2021年3月12完工)](https://forum.gamer.com.tw/C.php?bsn=33654&snA=587)
[^79]: [VRChat日记：模型改衣/衣柜设计教程（基础篇） - 梦雨玲音](https://www.rainlain.com/index.php/2024/06/14/2330/)
[^85]: [【教學】VRM 介紹與教學(概觀)，面向VR應用的3D模型新格式](https://home.gamer.com.tw/creationDetail.php?sn=4075972)
[^87]: [8【VRC改模】上传模型报错100%解决方案！【VRChat】](https://www.bilibili.com/video/BV1Pj421S7WH/)
[^88]: [VRchat上传角色error无法保存蓝图解决原创 - CSDN博客](https://blog.csdn.net/weixin_42666952/article/details/127357321)
[^89]: [【問題】新人上傳模組問題@VRChat 哈啦板 - 巴哈姆特](https://forum.gamer.com.tw/C.php?bsn=33654&snA=584)
[^95]: [非常个人向的VRM转VRC作业流程Part1:基础准备【VRChat ...](https://www.bilibili.com/read/cv29343793)
[^97]: [【VRCHAT】4分钟让你学会怎么上传模型](https://www.bilibili.com/video/BV1Pp4y1876t/)
[^98]: [【已过时】轻松简单自己上传VRChat的Avatar - 热心的博客 | Weekend](https://sunnyrx.com/2019/12/28/Easy-to-DIY-VRChat-Avatar/)
[^99]: [VRChat成品模型上传的小白说明书 - 哔哩哔哩](https://www.bilibili.com/read/cv21531240/)
[^100]: [VRChat日记：超简单获得角色模型/上传教程 - 梦雨玲音](https://www.rainlain.com/index.php/2024/06/07/2277/)
[^101]: [【VRChat 改模】开发环境搭建：VCC - CSDN博客](https://blog.csdn.net/qq_43768851/article/details/144017378)
[^102]: [將專案升級至Unity 2022 - VRChat 中文技術文件](https://vrcdocs.kamiya.app/getting-started/managing-unity/upgrading-projects-to-2022)
[^103]: [当前支持的Unity 版本 - VRChat 汉化文档中心](https://docs.vrczh.org/creators.vrchat.com/sdk/upgrade/current-unity-version)
[^104]: [Current Unity Version - VRChat Creation](https://creators.vrchat.com/sdk/upgrade/current-unity-version)
[^105]: [目前支援的Unity 版本 - VRChat 中文技術文件](https://vrcdocs.kamiya.app/getting-started/managing-unity/current-unity-version)
[^111]: [Getting Started - VRChat Creation](https://creators.vrchat.com/sdk/)
[^112]: [VRChat Creator Companion](https://vcc.docs.vrchat.com/)
[^114]: [虚拟形象 - VRChat Wiki](https://wiki.vrchat.com/wiki/Avatars/zh-hans)
[^117]: [Updating the SDK - VRChat Creation](https://creators.vrchat.com/sdk/updating-the-sdk/)
[^119]: [How to upload a VRChat Avatar in 2024 tutorial [PC and ...](https://ask.vrchat.com/t/how-to-upload-a-vrchat-avatar-in-2024-tutorial-pc-and-quest-beginners/22309)
[^120]: [Creating Your First Avatar - VRChat Creation](https://creators.vrchat.com/avatars/creating-your-first-avatar/)
[^121]: [How to upload avatar to VRChat in 2025 (Creator Companion ...](https://ask.vrchat.com/t/how-to-upload-avatar-to-vrchat-in-2025-creator-companion-pc-quest-mobile-and-ios-unity-tutorial/41446)
[^1001]: [Features and contents of VRM](https://vrm.dev/en/vrm/vrm_features/)
[^1002]: [③ 用UE5打造VTuber虚拟直播——准备人物模型（VRM模型）](http://www.bilibili.com/read/cv27193738/)
[^1003]: [Converting a VRChat Avatar to VRM Format for VTubing](https://xeiaso.net/blog/vrchat-avatar-to-vrm-vtubing-2022-01-02/)
[^1004]: [VRM Consortium](https://vrm-consortium.org/en/)
[^1005]: [Top 100件vrm模型- 2025年3月更新 - 淘寶](https://world.taobao.com/product/vrm%E6%A8%A1%E5%9E%8B.htm)
[^1008]: [不同的VRM 版本都有什么区别？浅谈VRM0.0 和VRM1.0﻿ - 模之屋](https://www.aplaybox.com/article/details/276874423)
[^1009]: [Preparing a 3D Model | LuppetX Document](https://doc.luppet.jp/en/basics/character)
[^1010]: [What is VRM ? What can VRM do ? | VRM](https://vrm.dev/en/vrm/vrm_about/)
[^1011]: [Steam 上的Vtuber Animator](https://store.steampowered.com/app/2810100/Vtuber_Animator/?l=schinese)
[^1012]: [Guide: RPG Developer Bakin & VRM Models - Meringue Rouge](https://meringue-rouge.itch.io/guide-rpg-developer-bakin-vrm-models-animations-rigging-and-more)
[^1013]: [VRM - Library of Congress](https://www.loc.gov/preservation/digital/formats/fdd/fdd000564.shtml)
[^1014]: [使用表情豐富的VRM格式3D人物-Ver3.2 - Clip Studio TIPS](https://tips.clip-studio.com/zh-tw/articles/10118)
[^1016]: [[Blender] How to Import VRM Files to Blender - STYLY](https://styly.cc/en/tips/blender-modeling-vrm/)
[^1019]: [Understanding VRM Files: Everything You Need to Know](https://www.modelo.io/damf/article/2024/07/03/2309/understanding-vrm-files--everything-you-need-to-know)
[^1020]: [开始使用爱发电了！丨Yueou丨爱发电](https://afdian.com/p/4151f35440dc11ed8e6d52540025c377)
[^1023]: [空猫Nilcat 的橱窗豪华版3D直播VRM像素风模型- 米画师](https://www.mihuashi.com/stalls/677822)
[^1026]: [VRoid Studio - imuslab](https://imuslab.com/wordpress/tag/vroid-studio)
[^1027]: [NFT 加密貨幣- 新手小白的區塊鏈遊記 - 方格子](https://vocus.cc/article/61e5a241fd89780001702266#!)
[^1028]: [Features and contents of VRM](https://vrm.dev/en/vrm/vrm_features/)
[^1031]: [VRM Consortium](https://vrm-consortium.org/en/)
[^1032]: [VRM Avatar Maker, Download Free VRM Models - VIVERSE](https://avatar.viverse.com/avatar/what-is-vrm)
[^1034]: [Let's Play Passpartout 2 Posting Campaign! - VRoid](https://vroid.com/en/news/32mPOek7ugFnbLcf6zT07c)
[^1036]: [VRM format - Blender Extensions](https://extensions.blender.org/add-ons/vrm/)
[^1037]: [Features and contents of VRM | VRM](https://vrm.dev/en/vrm/vrm_features/)
[^1038]: [VRM](https://www.loc.gov/preservation/digital/formats/fdd/fdd000564.shtml)
[^1039]: [VRM Consortium](https://vrm-consortium.org/en/)
[^1040]: [Alore [VRC & VRM 3D model] - Vket Store](https://store.vket.com/en/items/5447)
[^1041]: [vrm 3D 模型- 下载3D vrm Available formats: c4d, max, obj, fbx ...](https://ch.3dexport.com/3d-models/vrm)
[^1042]: [vrm - A 3D model collection by jeremy-coleman - Sketchfab](https://sketchfab.com/jeremy-coleman/collections/vrm-de8e545cce0e416e87effba63eedb123)
[^1043]: [VRM Avatar Maker, Download Free VRM Models - VIVERSE](https://avatar.viverse.com/avatar/what-is-vrm)
[^1044]: [Vrm - Etsy](https://www.etsy.com/market/vrm)
[^1045]: [vrm模型3d最新正品好评推荐- 2025年4月| 淘宝马来西亚](https://my.world.taobao.com/product/vrm%E6%A8%A1%E5%9E%8B3d.htm)
[^1046]: [3d Vtuber Model Vrm - Etsy](https://www.etsy.com/market/3d_vtuber_model_vrm?ref=lp_queries_external_bottom-23)
[^1047]: [Vrm 3D models - Sketchfab](https://sketchfab.com/tags/vrm)
[^1048]: [不同的VRM 版本都有什么区别？浅谈VRM0.0 和VRM1.0 - 模之屋](https://www.aplaybox.com/article/details/276874423)
[^1049]: [VRM 3D模型委託- 玥萊恩YoRenN | Clibo 同人委託平台](https://clibo.tw/commissions/E6Z9Gk)
[^1051]: [Top 100件vrm模型- 2025年3月更新 - 淘寶](https://world.taobao.com/product/vrm%E6%A8%A1%E5%9E%8B.htm)
[^1052]: [空猫Nilcat 的橱窗豪华版3D直播VRM像素风模型- 米画师](https://www.mihuashi.com/stalls/677822)
[^1054]: [24 Best Vrm Services To Buy Online | Fiverr](https://www.fiverr.com/gigs/vrm)
[^1055]: [Pastel Girl \[VRM model for streaming and VrChat ... - BOOTH](https://booth.pm/en/items/4236643)
[^1056]: [I'm looking for someone that can make me a custom VRM ...](https://www.reddit.com/r/VRoid/comments/1duoq5t/im_looking_for_someone_that_can_make_me_a_custom/)
[^1057]: [【VRChat/VRM】原创3D人物模型《佩露拉 - 爱发电](https://afdian.com/item/f3da27e0e11111ee95a252540025c377)
[^1058]: [③ 用UE5打造VTuber虚拟直播——准备人物模型（VRM模型）](http://www.bilibili.com/read/cv27193738/)
[^1059]: [原创VRchat/VRM 3D模型【伊芙IFu】(liltoon) - 爱发电](https://afdian.com/item/a15f36e46ba511efa8eb5254001e7c00)
[^1060]: [Alore [VRC & VRM 3D model] | Vket Store](https://store.vket.com/en/items/5447)
[^1061]: [VRM Avatar Maker, Download Free VRM Models | VIVERSE](https://avatar.viverse.com/avatar/what-is-vrm)
[^1062]: [[Vtuber Model] avatar 3D (vrm archive) - ELI Shop - BOOTH](https://booth.pm/en/items/3420476)
