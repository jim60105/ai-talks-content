+++
title = "Git 設定檔優先順序全解析：從臨時到系統層級"
description = "深入解析 Git 設定檔的套用優先序，從命令列、環境變數到本地、全域與系統層級。掌握多層級設定，提升開發彈性。還有進階 includeIf 與 CI/CD 實用技巧！"
date = "2025-06-25T01:12:02Z"
updated = "2025-07-28T12:51:13Z"
draft = false

[taxonomies]
tags = [ "Git" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/hXjR58s9DdykQxzsyWJgrN?invite=dOLYGeJyZJqVX>"
+++
{% alert(edit=true) %}
研究這個是由於自從我開始 Vibe Coding 之後 AI 直接用我的身份在活動，我發現我的 GitHub 帳號越來越像 🤖...

為了改善這個問題，我開了另一個帳號 [bot0419](https://github.com/bot0419) 專門給我的 🤖 小夥伴使用。我需要設定它在 commit 的時候使用自己的 username & email，但又不能直接改變專案層級或使用者層級設定，因為我自己也會使用同一個 repo。
{% end %}

{% alert(edit=true) %}
使用了一個月下來，最順暢的設定是[透過提示詞要求它在 commit 時帶上 `--author` 參數](https://github.com/jim60105/copilot-prompt/blob/master/.github/instructions/applyall_git.instructions.md#commit-command-template)；專案層級不設定 Author；使用者層級設定我的個人帳號。

一點心得提供給各位參考 👍
{% end %}

{% chat(speaker="jim") %}
請你列出 git 設定的套用順序，包含 ~/.gitconfig , /etc/gitconfig , [ENV 環境變數](https://git-scm.com/book/be/v2/Git-Internals-Environment-Variables), and [GIT_CONFIG_KEY 環境變數]( https://git-scm.com/docs/git-config#ENVIRONMENT)

如果我有任何遺漏的其它方式也請幫我補上
{% end %}

{% chat(speaker="felo") %}
Git 的組態設定採用一個明確的階層式覆蓋系統，確保了高度的靈活性與客製化能力。設定的優先序由高至低依序為：單一指令的臨時參數、環境變數、本地倉庫設定、使用者全域設定，以及系統層級設定。這個設計讓開發者能針對不同情境（例如單一指令、特定專案、個人化偏好或全系統標準）進行精細的控制，其中最特定、最接近當前操作的設定會覆蓋較通用、範圍較廣的設定。理解此優先序是高效管理開發環境與確保不同專案間設定正確性的關鍵。
{% end %}

## Git 組態的層級與優先序

Git 透過多個來源讀取其組態設定，並依照嚴格的優先順序來決定最終生效的值。當同一個設定鍵（key）出現在多個層級時，擁有較高優先權的層級其設定值將會覆蓋（override）較低優先權的層級 [^2][^6][^33][^42][^64]。以下將由最高優先序至最低優先序，詳細解析各個組態層級。

### 1. 命令列 `-c` 參數 (臨時配置)

這是優先序最高的設定方式，其效力僅限於單次執行的 Git 指令 [^16]。這種方式非常適合用於臨時覆蓋某個特定設定，而不想永久修改任何設定檔。

* **設定方式**: 在執行 `git` 指令時，於子命令前加上 `-c <鍵>=<值>` 參數 [^16]。
* **使用情境**: 當您需要為單一次的提交（commit）使用特定的使用者名稱或信箱，但不希望更動本地或全域設定時。
* **範例**:

    ```bash
    # 僅在這次提交中，臨時將 user.name 設定為 "Temporary Author"
    git -c user.name="Temporary Author" -c user.email="temp@example.com" commit -m "Initial commit with temporary author"
    ```

### 2. 環境變數 (Environment Variables)

環境變數的優先序僅次於 `-c` 參數，高於所有設定檔 [^20][^52]。Git 會讀取特定的環境變數來影響其行為或直接覆蓋組態值。

* **`GIT_CONFIG_KEY_<KEY>`**: 這是 Git 提供的一種強大機制，可以透過環境變數直接設定任何 `git config` 的值。 ~~變數名稱的 `<KEY>` 部分會被轉換為小寫並將底線 `_` 轉換為點 `.`。例如，`GIT_CONFIG_KEY_USER_NAME` 會對應到 `user.name`。~~

  {% alert(edit=true) %}
  這邊 AI 的回答是錯誤的，事實上這功能得像下方這樣子用:

  ```bash
  export GIT_CONFIG_COUNT=2
  export GIT_CONFIG_KEY_0='user.name'
  export GIT_CONFIG_VALUE_0='Your Name'
  export GIT_CONFIG_KEY_1='user.email'
  export GIT_CONFIG_VALUE_1='your-email@example.com'
  ```

  注意 KEY 和 VALUE 後面的數字是 0-based，而 count 是數量。一定要從 0 開始，不能跳號，count 要對上！  
  請參閱 [官方文件](https://git-scm.com/docs/git-config#Documentation/git-config.txt-GITCONFIGCOUNT) 以及 [Stack Overflow](https://stackoverflow.com/a/78064753/8706033)
  {% end %}

* **行為控制變數**: Git 也定義了許多用來控制特定功能的環境變數，它們會覆蓋設定檔中的對應項目 [^4][^8]。
  {% alert(edit=true) %}
  這些並不是全部的 Git 設定值都有，請參考這份官方文件:
  
  <https://git-scm.com/book/be/v2/Git-Internals-Environment-Variables>
  {% end %}
  * `GIT_EDITOR`: 指定當需要編輯文字（如提交訊息）時所啟動的編輯器，其優先權高於 `core.editor` 設定 [^8][^52]。
  * `GIT_PAGER`: 控制多頁輸出時所使用的分頁程式，優先於 `core.pager` [^4][^8]。
  * `GIT_DIR`: 指定 `.git` 目錄的路徑。如果設定了此變數，Git 將不會在當前目錄向上搜尋 `.git` 目錄 [^4][^8]。
* **`GIT_CONFIG_NOSYSTEM`**: 若設定此環境變數，Git 將會完全忽略系統層級的設定檔 (`/etc/gitconfig`) [^4][^8]。

### 3. 本地/倉庫層級 (`--local`)

此層級的設定僅對單一的 Git 倉庫有效，是針對特定專案進行客製化的主要方式 [^6][^16]。

* **設定檔位置**: 位於倉庫根目錄下的 `.git/config` 檔案 [^2][^17][^18][^22][^51]。
* **設定方式**: 在倉庫目錄下執行 `git config` 指令（預設即為 `--local`），或手動編輯 `.git/config` 檔案 [^3][^6]。

    ```bash
    # 僅為當前專案設定 user.email
    git config user.email "project-specific-email@work.com"
    ```

* **優先序**: 其優先權高於全域和系統層級的設定 [^45][^54][^66]。這意味著您可以在全域設定個人信箱，然後在公司專案的本地設定中將其覆蓋為工作信箱 [^20]。

### 4. 全域/使用者層級 (`--global`)

全域設定會影響目前登入使用者所操作的所有 Git 倉庫，除非被本地設定所覆蓋 [^3][^16]。這是設定個人開發環境偏好的最常用層級。

* **設定檔位置**: 通常位於使用者家目錄下的 `~/.gitconfig` 或 `~/.config/git/config` [^2][^17][^22]。
* **設定方式**: 使用 `git config --global` 指令 [^3][^16]。

    ```bash
    # 設定所有倉庫預設使用的使用者名稱與信箱
    git config --global user.name "Your Name"
    git config --global user.email "your-personal-email@example.com"
    
    # 設定一個常用的指令別名
    git config --global alias.st status
    ```

* **優先序**: 高於系統層級設定，但會被本地層級設定覆蓋 [^6][^33][^57]。

### 5. 系統層級 (`--system`)

這是優先序最低的設定層級，其設定會套用到該系統上的所有使用者及其所有倉庫 [^2][^3][^22]。

* **設定檔位置**: 在 Linux 或 macOS 系統上通常是 `/etc/gitconfig` [^2][^28][^34]；在 Windows 上則通常位於 Git 的安裝目錄下，例如 `C:\Program Files\Git\etc\gitconfig` [^35]。
* **設定方式**: 需使用 `git config --system` 指令，且通常需要系統管理員權限才能修改 [^2][^3][^22]。
* **使用情境**: 通常由系統管理員用來設定適用於全公司的 HTTP 代理、憑證或預設行為。

## 優先序總結與套用規則

Git 在執行時會由低至高讀取所有層級的設定檔，並將它們合併。若遇到重複的鍵，高優先序的值會覆蓋低優先序的值 [^18][^57][^64]。

您可以使用 `git config --list --show-origin` 指令來查看所有生效的設定、它們的值以及它們來自哪個設定檔。

下表總結了 Git 組態的優先順序：

| 層級 (Level) | 說明 | 設定方式 / 檔案 | 優先序 |
| :--- | :--- | :--- | :--- |
| **臨時 (Temporary)** | 僅對單一指令有效，擁有絕對最高的優先權 [^16]。 | `git -c key=value` | **1 (最高)** |
| **環境變數 (Environment)** | 影響當前 Shell 工作階段，覆蓋所有設定檔 [^52]。 | `export GIT_...` | **2** |
| **本地 (Local)** | 僅對當前倉庫有效，用於專案特定設定 [^6][^16]。 | `.git/config` | **3** |
| **全域 (Global)** | 對當前使用者所有倉庫有效，用於個人化設定 [^3][^22]。 | `~/.gitconfig` | **4** |
| **系統 (System)** | 對系統上所有使用者有效，用於系統級標準 [^2][^22]。 | `/etc/gitconfig` | **5 (最低)** |

## 進階組態管理技巧

對於專業開發者，除了基本的層級設定外，Git 還提供了一些進階技巧來應對複雜的開發場景。

### 條件式組態 `includeIf`

這是一個非常實用的功能，允許您根據倉庫所在的目錄來條件性地載入不同的設定檔。最常見的用途是管理多個開發身份，例如個人專案和工作專案使用不同的使用者名稱和信箱 [^32]。

* **範例**: 在您的 `~/.gitconfig` 中加入以下設定：

    ```gitconfig
    [user]
        name = Your Personal Name
        email = personal@example.com

    # 如果倉庫路徑在 ~/work/ 底下
    [includeIf "gitdir:~/work/"]
        # 就載入 ~/.gitconfig-work 這個設定檔
        path = ~/.gitconfig-work
    ```

    然後在 `~/.gitconfig-work` 檔案中定義工作專用的設定：

    ```gitconfig
    # ~/.gitconfig-work
    [user]
        name = Your Work Name
        email = work@company.com
    ```

### GUI 與 IDE 整合

許多圖形化介面工具（如 GitKraken）和整合開發環境（如 Visual Studio）提供了方便的 UI 來設定 Git [^2][^12]。這些工具在後端實際上也是透過執行 `git config` 指令來修改對應的設定檔 [^12]。

### CI/CD 環境中的應用

在自動化流程（如 GitLab CI/CD 或 GitHub Actions）中，環境變數是設定 Git 行為最主要且最靈活的方式。例如，在 GitLab CI 中，可以使用 `GIT_STRATEGY` 環境變數來控制 Runner 如何獲取程式碼 (`clone`, `fetch`, 或 `none`)，以優化管線執行效率 [^15]。

[^2]: [初次設定Git](https://git-scm.com/book/zh-tw/v2/%E9%96%8B%E5%A7%8B-%E5%88%9D%E6%AC%A1%E8%A8%AD%E5%AE%9A-Git)
[^3]: [git config_git config 配置多个路径 - CSDN博客](https://blog.csdn.net/qq_33240556/article/details/138949557)
[^4]: [環境変数 - Git](https://git-scm.com/book/ja/v2/Git%E3%81%AE%E5%86%85%E5%81%B4-%E7%92%B0%E5%A2%83%E5%A4%89%E6%95%B0)
[^6]: [git config的配置（system、global、local） - 老潘的博客](https://www.panyanbin.com/article/ff4e1f85.html)
[^8]: [环境变量 - Git](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F)
[^12]: [Visual Studio 中的Git 設定和喜好設定](https://learn.microsoft.com/zh-tw/visualstudio/version-control/git-settings?view=vs-2022)
[^15]: [GIT_STRATEGY 種類對運行效率的影響 - CPHT](https://www.cpht.pro/blog/blog-post-42/)
[^16]: [Git的基本配置 - Zenggyu的博客](https://blog.zenggyu.com/zh/post/2018-08-27/setting-up-a-git-repository/)
[^17]: [【Git】了解git config 設定- 健忘工程師](https://forgetfulengineer.github.io/Other/Git/Understanding-Git-Config-Settings/)
[^18]: [Git 设置全局或者当前项目的用户信息原创 - CSDN博客](https://blog.csdn.net/u013719138/article/details/89888908)
[^20]: [如何为单个Git 仓库设置独立邮箱，不影响全局配置 - CSDN博客](https://blog.csdn.net/weixin_51524504/article/details/148849140)
[^22]: [Git应用详解-1.2 git配置 - 智客工坊](https://www.52interview.com/book/12/114)
[^28]: [配置Git — git-reference 0.1 文档](https://git-reference.readthedocs.io/zh_CN/latest/Customizing-Git/Git-Configuration/)
[^32]: [调整你的Git 配置以适应多个用户ID 的需要| Linux 中国 - 知乎专栏](https://zhuanlan.zhihu.com/p/287081862)
[^33]: [【Git教學】 超輕鬆git config 設定指南 - Max行銷誌](https://www.maxlist.xyz/2022/05/26/git-config-setting/)
[^34]: [git config 配置 - 陈看川的个人网站。](https://kanchuan.com/blog/124-gitconfig)
[^35]: [Git for windows 疑難雜症 - 傑克! 真是太神奇了!](https://magicjackting.pixnet.net/blog/post/222998706)
[^42]: [初次設定Git](https://git-scm.com/book/zh-tw/v2/%E9%96%8B%E5%A7%8B-%E5%88%9D%E6%AC%A1%E8%A8%AD%E5%AE%9A-Git)
[^45]: [【Git】了解git config 設定- 健忘工程師](https://forgetfulengineer.github.io/Other/Git/Understanding-Git-Config-Settings/)
[^51]: [git/git-config.md at master · peterhpchen/git - GitHub](https://github.com/peterhpchen/git/blob/master/git-config.md)
[^52]: [Git 筆記](https://www.hwchiu.com/2014/07/28/git)
[^54]: [[Git] 設定不同路徑或單一Repo 擁有各自的作者資訊](https://weirenxue.github.io/2022/01/22/git_author/)
[^57]: [30 天精通Git 版本控管(14)： Git for Windows 選項設定](https://ithelp.ithome.com.tw/articles/10137473)
[^64]: [Git 介紹＆ 基本指令操作- HackMD](https://hackmd.io/@rgnkud/ryAJG7-DT)
[^66]: [git：config命令原创 - CSDN博客](https://blog.csdn.net/sayyy/article/details/99951469)
