+++
title = "以 USB 和 Git 裸儲存庫：離線程式碼同步的終極指南"
description = "本指南詳解如何使用 USB 隨身碟搭配 Git 裸儲存庫，在無網路環境中安全同步程式碼。解釋避免直接在 USB 上開發的原因，並逐步指導從建立裸儲存庫、複製到本地開發、推送變更，到使用 PowerShell 腳本自動推送到遠端的完整流程。"
date = "2025-09-05T18:26:11Z"
updated = "2025-09-07T07:20:12Z"
draft = false

[taxonomies]
tags = [ "DevOps", "Git", "PowerShell", "Prompt Engineering" ]
providers = [ "Felo Search" ]

[extra]
withAI = "<https://felo.ai/search/mfEhvfZoWsee7ZGX7E7fXd?invite=dOLYGeJyZJqVX>"
hot = true
+++
{% alert(edit=true) %}
我的情境在提示詞中描述得不夠清楚，導致 Felo 的回答和實際不太相符，但內容是合用的。它把內外網搞反了 😅

## 內網環境：B 電腦操作 USB 與 Git Remote 同步

<figure>
{{ image(url="internal-network-flow.svg", alt="內網環境：B 電腦操作 USB 與 Git Remote 同步流程圖", no_hover=true) }}
<figcaption>內網環境：B 電腦操作 USB 與 Git Remote 同步流程圖</figcaption>
</figure>

## 外網環境：A 電腦與 USB 同步

<figure>
{{ image(url="external-network-flow.svg", alt="外網環境：A 電腦與 USB 同步流程圖", no_hover=true) }}
<figcaption>外網環境：A 電腦與 USB 同步流程圖</figcaption>
</figure>

實際情境是 Git Remote 在內網環境，B 電腦連接內網做為中介，A 電腦是開發主機禁止連接內網，只能透過 USB 隨身碟帶回 B 電腦再推送到內網的 Git Remote。當然，在外網同步程式碼被嚴格禁止，程式碼不能儲存在 A、B、USB、Git Remote 之外的地方。

環境會長這樣當然是因為公司的資安政策，以及政策下的 workaround。  
我個人是覺得用 USB 插來插去很髒，但我太菜無法改變這個事實。😶‍🌫️  
開發者會需要裝些奇奇怪怪的東西，劃定 A 開發機較危險是合理的，但若能切個獨立的 DMZ 開發網段給 A 和 Git Remote 會直觀很多。
{% end %}

{% alert(edit=true) %}
從以下提示詞能看出我了解整個流程，我的目的是要產生一份詳細的指南，讓團隊的其他人也能照著做。

這是讓 AI 照我的意思寫指南的提示詞用法。
{% end %}

{% chat(speaker="jim") %}
我有兩個任務：

1. 從 A 電腦透過 USB 將程式碼同步到 B 電腦
2. 從 B 電腦將程式碼推送到遠端

Only the B computer can access the remote.  
Do a brief explain about Git bare repo at the beginning.  
Explain don't create normal Git repo in USB and develop there because of the flash drive design.

Please guide me create a Git bare repo in the USB, by cloning the repo from remote at very first-time.  
Guide me clone from USB bare repo to A computer.  
Guide me push the commits to USB bare repo after doing some development tasks on A computer.  
Guide me push the commits from USB bare repo to remote with a pwsh script on B computer.
{% end %}

{% chat(speaker="felo") %}
本報告闡述了一種利用 Git 裸儲存庫 (Bare Repository) 和 USB 隨身碟，在可存取遠端的電腦 (B 電腦) 與離線電腦 (A 電腦) 之間安全同步程式碼的完整工作流程。此方法的核心是將 USB 隨身碟作為一個可攜式的中介遠端儲存庫，讓開發者能在離線環境中進行版本控制，然後再透過另一台電腦將變更推送到中央伺服器。此流程不僅解決了網路隔離環境下的開發協作問題，也透過使用裸儲存庫避免了直接在 USB 隨身碟上進行高頻率讀寫操作而可能導致的損壞風險。整個流程包含在 USB 上建立裸儲存庫、從裸儲存庫複製 (clone) 到本機進行開發、將本地變更推送回 USB，最終透過 PowerShell 指令碼將 USB 上的變更同步至遠端儲存庫。
{% end %}

## Git 裸儲存庫 (Bare Repository) 詳解

Git 裸儲存庫是一種特殊的儲存庫，它不包含工作目錄 (working directory)，也就是你看不到專案的實際檔案 [^27]。它僅包含 Git 版本控制的 Metadata，即一般儲存庫中 `.git` 資料夾內的所有內容 [^27][^45]。這種設計使其成為一個理想的中央集散地，用於接收來自開發者的推送 (push) 和提供複製 (clone) 來源，但不能直接在其中編輯檔案或進行提交 (commit) [^14][^16][^27]。

## 裸儲存庫 vs. 標準儲存庫

| 特性 | 裸儲存庫 (Bare Repository) | 標準儲存庫 (Non-Bare Repository) |
| --- | --- | --- |
| **工作目錄** | 無，無法直接編輯檔案 [^27] | 有，包含專案檔案的實際工作副本 [^27] |
| **`.git` 資料夾** | 內容直接儲存在儲存庫根目錄 | 儲存在專案根目錄下的 `.git` 隱藏資料夾中 |
| **主要用途** | 作為中央伺服器或中介儲存庫，用於分享與協作 [^14][^23][^45] | 用於本地開發、編輯程式碼與提交變更 [^4] |
| **常用指令** | `git init --bare`, `git clone --bare` [^15][^53] | `git init`, `git clone` [^6][^19] |
| **推送 (Push) 操作** | 設計用來接收來自其他儲存庫的推送 [^24] | 通常不建議直接推送到一個已簽出 (checked-out) 分支的標準儲存庫 |

## 為何不應在 USB 隨身碟上直接開發

USB 隨身碟的快閃記憶體 (flash memory) 在設計上有其讀寫次數的壽命限制。若在 USB 隨身碟上建立一個標準的 Git 儲存庫並直接進行開發，將會產生大量且頻繁的讀寫操作。例如，執行 `git status`、`git add`、`git commit` 等指令都會讀取或修改 `.git` 資料夾內的索引檔、物件檔和日誌檔 [^21]。這些高頻率的操作會加速快閃記憶體的耗損，可能導致隨身碟提早損壞 [^21][^23]。

相對地，將 USB 隨身碟作為一個裸儲存庫，僅在同步時執行 `git push` 操作，這是一個相對低頻率的批次寫入動作 [^25]。這種模式大幅減少了對隨身碟的零碎讀寫，能有效延長其使用壽命，使其成為一個更可靠的程式碼中介載體 [^13][^23]。

{% alert(edit=true) %}
無論如何，USB 仍是隨時會壞掉的東西，務必隨時和遠端儲存庫保持同步。
{% end %}

## 步驟一：在 B 電腦建立 USB 裸儲存庫 (首次設定)

此步驟的目標是將遠端伺服器 (如 GitHub、GitLab) 上的現有儲存庫，以裸儲存庫的形式完整複製到 USB 隨身碟上。此操作在具有網路存取權限的 B 電腦上執行。

1. **準備工作**:
    * 將 USB 隨身碟連接至 B 電腦。假設其磁碟機代號為 `F:`。
    * 開啟 PowerShell 或其他命令列工具。

2. **複製遠端儲存庫為裸儲存庫**:
    使用 `git clone` 指令搭配 `--bare` 參數。這個指令會下載遠端儲存庫的所有資料，但不建立工作目錄 [^51][^53]。依照慣例，裸儲存庫的資料夾名稱通常以 `.git` 結尾 [^14][^47]。

    ```powershell
    # 將 <remote-repository-url> 替換為你的遠端儲存庫 URL
    git clone --bare <remote-repository-url> F:\your-project.git
    ```

    執行後，`F:\your-project.git` 資料夾內將包含所有 Git 的版本歷史、分支和標籤，作為一個可攜式的遠端儲存庫 [^51]。

## 步驟二：在 A 電腦從 USB 複製儲存庫並進行開發

現在，將 USB 隨身碟從 B 電腦拔除，並連接至沒有網路存取權限的 A 電腦。你將從這個 USB 裸儲存庫複製一份標準的開發用儲存庫到 A 電腦的本機磁碟。

1. **複製 USB 裸儲存庫至本機**:
    開啟命令列工具，使用標準的 `git clone` 指令，來源指向 USB 隨身碟上的裸儲存庫路徑 [^3][^11][^19]。

    ```powershell
    # Git 會在目前路徑下建立一個名為 "your-project" 的資料夾
    git clone F:\your-project.git
    ```

    複製完成後，Git 會自動將 USB 上的裸儲存庫設定為名為 `origin` 的遠端來源 [^33]。這意味著後續的 `push` 和 `pull` 操作預設都會與 USB 隨身碟互動。

2. **進行開發與提交**:
    進入新建立的專案資料夾，你可以像平常一樣進行開發工作。

    ```powershell
    # 進入專案目錄
    cd your-project

    # 進行程式碼修改...
    # 例如，建立一個新檔案
    New-Item -ItemType File "feature-A.txt"
    Write-Output "Details for feature A" | Out-File "feature-A.txt"

    # 將變更加入暫存區並提交
    git add .
    git commit -m "Add feature A on offline machine"
    ```

3. **將變更推送至 USB 隨身碟**:
    開發完成後，將本地的提交推送到 USB 隨身碟上的裸儲存庫 [^12][^17][^48]。

    ```powershell
    # 假設你在 main 分支上開發
    git push origin main
    ```

    此時，你在 A 電腦上的所有新提交都已安全地儲存到 USB 裸儲存庫中。

## 步驟三：在 B 電腦透過 PowerShell 將 USB 變更推送到遠端

最後一步是將攜帶最新程式碼的 USB 隨身碟帶回 B 電腦，並將其上的變更同步到真正的遠端伺服器。為了簡化這個重複性任務，我們將建立一個 PowerShell 指令碼來自動化此流程。

{% alert(edit=true) %}
這裡應該要先 `git fetch` 再 `git push`，並且僅推送你的開發分支，以避免衝突。
{% end %}

1. **建立 PowerShell 指令碼**:
    在 B 電腦上，建立一個名為 `Sync-To-Remote.ps1` 的檔案，並貼上以下內容。這個指令碼會接收 USB 裸儲存庫的路徑作為參數，進入該目錄，然後執行 `git push` 指令 [^22]。

    ```powershell
    # Sync-To-Remote.ps1

    # 定義指令碼參數，接收 USB 裸儲存庫的路徑
    param (
        [Parameter(Mandatory=$true)]
        [string]$UsbRepoPath
    )

    # 檢查提供的路徑是否存在
    if (-not (Test-Path $UsbRepoPath)) {
        Write-Error "錯誤：找不到指定的 USB 儲存庫路徑: $UsbRepoPath"
        return
    }

    try {
        Write-Host "正在進入 USB 裸儲存庫目錄: $UsbRepoPath"
        
        # 進入裸儲存庫目錄
        Push-Location -Path $UsbRepoPath

        Write-Host "正在將變更從 USB 推送到遠端 'origin'..."
        
        # 執行 git push 指令。
        # 裸儲存庫在建立時已從遠端複製，因此 'origin' 已指向原始的遠端伺服器。
        # 我們將所有本地分支 (refs/heads/*) 推送到遠端的對應分支。
        git push origin refs/heads/*:refs/heads/*

        Write-Host "推送成功！"

    }
    catch {
        # 捕捉並顯示任何錯誤
        Write-Error "發生錯誤: $_"
    }
    finally {
        # 無論成功或失敗，都返回原始目錄
        Pop-Location
        Write-Host "已返回原始目錄。"
    }
    ```

2. **執行指令碼以同步變更**:
    將 USB 隨身碟連接至 B 電腦。開啟 PowerShell，執行剛剛建立的指令碼，並傳入 USB 裸儲存庫的路徑 [^4][^6]。

    ```powershell
    # 執行指令碼，將 F:\your-project.git 的內容推送到遠端
    .\Sync-To-Remote.ps1 -UsbRepoPath "F:\your-project.git"
    ```

    指令碼會自動完成所有操作，將你在 A 電腦上的工作成果安全地發布到遠端伺服器。

## Summary

此報告詳細介紹了一個利用 USB 隨身碟作為中介 Git 裸儲存庫的離線開發工作流程。該方法有效地結合了 Git 的分散式特性與實體媒介的便攜性，為在網路受限環境下進行版本控制提供了可靠的解決方案 [^3][^20][^31]。透過在 USB 上使用裸儲存庫而非標準儲存庫，避免了因頻繁讀寫而導致的硬體耗損問題，確保了資料的完整性與媒介的耐用性 [^23][^25]。從初次設定、離線開發到最終的遠端同步，每個步驟都提供了清晰的指令和解釋，並附上自動化 PowerShell 指令碼，使整個流程高效且不易出錯。此工作流程不僅適用於個人在多台裝置間同步工作，也為需要在隔離網路中進行協作的團隊提供了一個實用的參考模型 [^3][^15]。

[^3]: [How can I use git bare repository on usb drive to collaborate ...](https://stackoverflow.com/questions/76229384/how-can-i-use-git-bare-repository-on-usb-drive-to-collaborate-with-other-develop)
[^4]: [Examples to show Git from Powershell - GitHub Pages](https://papercutsoftware.github.io/git-from-powershell/)
[^6]: [PowerShell下git提交代码仓库原创 - CSDN博客](https://blog.csdn.net/weixin_54338498/article/details/125688144)
[^11]: [Clone a Git repository to your local computer - GitLab Docs](https://docs.gitlab.com/topics/git/clone/)
[^12]: [2.5 Git 基礎- 與遠端協同工作](https://git-scm.com/book/zh-tw/v2/Git-%E5%9F%BA%E7%A4%8E-%E8%88%87%E9%81%A0%E7%AB%AF%E5%8D%94%E5%90%8C%E5%B7%A5%E4%BD%9C)
[^13]: [PC-Phone USB Sync - Google Play 上的应用](https://play.google.com/store/apps/details?id=com.quixotely.usbsync&hl=zh)
[^14]: [Push your Git repositories to a central server, in a bare ...](https://www.jeffgeerling.com/blogs/jeff-geerling/push-your-git-repositories)
[^15]: [用U盘作为远程git仓库管理代码原创 - CSDN博客](https://blog.csdn.net/JAYSITE/article/details/102264476)
[^16]: [git bare 的使用--- 推送代码到本地仓库（与同事协作开发必备 ...](https://blog.csdn.net/qq_24047235/article/details/111042273)
[^17]: [Pushing commits to a remote repository - GitHub Docs](https://docs.github.com/en/get-started/using-git/pushing-commits-to-a-remote-repository)
[^19]: [Cloning a repository - GitHub Docs](https://docs.github.com/articles/cloning-a-repository)
[^20]: [Using Git across multiple systems without network access](https://superuser.com/questions/1367571/using-git-across-multiple-systems-without-network-access)
[^21]: [游戏开发工作丢了两年... : r/gamedev - Reddit](https://www.reddit.com/r/gamedev/comments/v0d3u0/lost_2_years_of_game_dev_work/?tl=zh-hans)
[^22]: [将清单提交到存储库 - Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/package-manager/package/repository)
[^23]: [【转】把Git Repository建到U盘上去- Caersi - 博客园](https://www.cnblogs.com/Caersi/p/6367271.html)
[^24]: [git push to bare remote - GitHub Gist](https://gist.github.com/joahking/780877)
[^25]: [Git Push to USB - Blog 1](https://vincenttam.github.io/blog/2015/01/01/git-push-to-usb)
[^27]: [Git clone bare and git clone mirror - Graphite](https://graphite.dev/guides/git-clone-bare-mirror)
[^31]: [Git 同步多台设备？没有中心仓库也能玩得转！ - 博客园](https://www.cnblogs.com/ghj1976/p/18599000/git-tong-bu-duo-tai-she-bei-mei-you-zhong-xin-cang)
[^33]: [git clone | Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials/setting-up-a-repository/git-clone)
[^45]: [Getting Git on a Server](https://git-scm.com/book/en/v2/Git-on-the-Server-Getting-Git-on-a-Server)
[^47]: [[Git] 用Git在本地電腦做備份](https://mailtojacklai.medium.com/git-%E7%94%A8git%E5%9C%A8%E6%9C%AC%E5%9C%B0%E9%9B%BB%E8%85%A6%E5%81%9A%E5%82%99%E4%BB%BD-22d727af479f)
[^48]: [Pushing Changes to a Git Repository - GeeksforGeeks](https://www.geeksforgeeks.org/git/pushing-changes-to-a-git-repository/)
[^51]: [复制仓库- GitHub 文档](https://docs.github.com/zh/repositories/creating-and-managing-repositories/duplicating-a-repository)
[^53]: [Quickly create a Git bare repo with init or clone example](https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/Quickly-create-a-git-bare-repo-with-init-or-clone)
