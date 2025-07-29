+++
title = "Linux 下 Git 換行符一致性設定全攻略"
description = "跨平台開發常遇換行符混亂問題，本文教你如何在 Linux 下設定 Git，確保 checkout 與 commit 都維持 LF 格式。透過 .gitattributes 及本地組態，讓團隊協作更順暢。立即掌握正確設定流程！"
date = "2025-05-04T18:38:29Z"
updated = "2025-07-27T15:40:21Z"
draft = false

[taxonomies]
tags = [ "Git", "Linux" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/eixgEFxP8UP8qVwAstF7Ai?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
我使用 Fedora 系統，和其它使用 Windows 的開源社群的人一起寫專案。我沒有辦法規範其它的人，我們的 git repo 中有 CRLF 也有 LF。我希望我在 git checkout 時全部轉換為 LF，並且在我 git commit 時按照現有的狀況(或者轉為 LF 加入)。我要如何設定我的 git config 設定?
{% end %}

{% chat(speaker="felo") %}
根據搜尋結果，我將詳細分析相關問題。
{% end %}

## Overview

本報告旨在提供在 Fedora (Linux) 系統上設定 Git 的詳細指南，以確保在與使用 Windows 的開發者協作時，處理混雜 CRLF 與 LF 換行符的儲存庫（repository）能夠滿足特定需求：於 `git checkout` 時將所有文字檔案轉換為 LF 換行符，並在 `git commit` 時，將工作目錄中的 LF 換行符提交至儲存庫，或確保儲存庫中的檔案最終為 LF 換行符。核心解決方案是透過專案根目錄下的 `.gitattributes` 檔案來強制執行換行符規則，輔以適當的本地 Git 全域設定。

## Detailed Report

### 換行符問題背景

不同的作業系統使用不同的字元來標記文字檔案的換行：

* **Windows**: 使用 CRLF (`\r\n`) [^2][^4][^63]。
* **Linux/Unix/macOS (較新版本)**: 使用 LF (`\n`) [^2][^4][^39]。

當不同作業系統的使用者在同一個 Git 儲存庫上協作時，若未妥善處理，換行符的差異會導致 Git 將整個檔案標記為已修改，即使內容本身沒有變更，這會干擾程式碼比對 (diff) 和審查 (review) [^2][^6][^46][^54]。

### Git 處理換行符的機制

Git 提供數種機制來處理換行符轉換：

**1. `core.autocrlf` 全域或本地設定**
此設定決定 Git 在提交 (commit) 和檢出 (checkout) 時是否自動轉換換行符 [^3][^68]。

* `true`: (Windows 建議) 檢出時將 LF 轉為 CRLF，提交時將 CRLF 轉為 LF [^1][^14][^17][^38]。
* `input`: (Linux/macOS 建議) 檢出時不轉換，提交時將 CRLF 轉為 LF [^3][^14][^17][^38]。
* `false`: 檢出與提交時皆不進行任何轉換 [^2][^5][^13][^38]。

**2. `.gitattributes` 檔案**
這是在儲存庫層級定義檔案屬性的方法，其設定會覆寫使用者的 `core.autocrlf` 設定，是確保團隊一致性的最可靠方式 [^3][^9][^15][^20][^50][^85]。

* `text=auto`: Git 自動偵測檔案是否為文字檔。若是，則根據 `core.autocrlf` 或 `core.eol` 進行轉換，預設行為是提交時轉換為 LF [^3][^9][^70]。
* `text eol=lf`: 強制 Git 在檢出時將文字檔案的換行符轉換為 LF，無論在哪個作業系統 [^3][^47][^68][^72]。
* `text eol=crlf`: 強制 Git 在檢出時將文字檔案的換行符轉換為 CRLF [^3][^32][^68][^72]。
* `binary`: 將檔案標記為二進位檔，Git 不會對其進行換行符轉換 [^3][^68]。
* `* text=auto eol=lf`: 結合 `text=auto` 和 `eol=lf`，讓 Git 自動偵測文字檔，並在檢出時強制轉換為 LF [^71][^73]。

**3. `core.eol` 設定**
指定工作目錄中文字檔案應使用的換行符，選項有 `lf`, `crlf`, `native` (作業系統預設) [^5][^38][^71]。通常不建議直接修改此設定，除非有特殊需求 [^15]。

**4. `core.safecrlf` 設定**
用於檢查提交的檔案是否包含混合換行符或可能導致不可逆轉換 [^13]。

* `true`: 若檢查到問題，則拒絕提交 [^2][^13]。
* `warn`: 若檢查到問題，則顯示警告但允許提交 [^13][^92]。
* `false`: 不進行檢查，允許提交 [^13][^25][^90]。

### 滿足使用者需求的設定方案

根據使用者需求（Fedora 環境下，checkout 時強制為 LF，commit 時提交 LF），最佳實踐是使用 `.gitattributes` 檔案來統一儲存庫的行為。

**步驟 1: 設定 `.gitattributes` 檔案**
在專案的根目錄下建立或修改 `.gitattributes` 檔案，加入以下內容：

```gitattributes
# 自動偵測文字檔，並在 checkout 時強制使用 LF 換行符
* text=auto eol=lf
```

* `*`: 此規則適用於所有檔案。
* `text=auto`: Git 會自動判斷哪些是文字檔，並對其應用換行符規則，而二進位檔則不受影響 [^3][^9][^15]。
* `eol=lf`: 強制 Git 在 `git checkout` 時，將所有被識別為文字檔的檔案轉換成 LF 換行符 [^3][^47][^72]。這確保了無論儲存庫中原始的換行符是什麼，使用者在 Fedora 上的工作目錄中看到的都是 LF。
* **提交行為**: 當檔案被標記為 `text` (透過 `text=auto` 或明確指定 `text`) 時，Git 在提交（`git commit`）時會預設將其換行符正規化 (normalize) 為 LF 儲存到儲存庫中 [^5][^77][^81]。結合 `eol=lf` 確保工作目錄為 LF，這能保證提交的也是 LF。

**步驟 2: 設定使用者的本地 Git 全域組態 (Fedora/Linux 建議)**
雖然 `.gitattributes` 優先級更高 [^3][^15][^20]，但設定好本地全域組態仍是良好習慣。

```bash
# 設定提交時將 CRLF 轉換為 LF，checkout 時不轉換 (Linux/macOS 推薦值)
git config --global core.autocrlf input

# 建議開啟：拒絕提交包含混合換行符的檔案
git config --global core.safecrlf true
```

* `core.autocrlf input`: 這是 Linux 系統的標準建議值 [^3][^17][^34]。即使 `.gitattributes` 未涵蓋某些檔案，此設定也能確保在提交時嘗試將 CRLF 轉換為 LF。
* `core.safecrlf true`: 增加一層保護，防止因意外情況（如工具錯誤地引入 CRLF）而提交了包含混合換行符的檔案 [^2][^13]。

**步驟 3: 對現有儲存庫進行正規化**
在加入或修改 `.gitattributes` 檔案後，需要執行一次正規化操作，讓 Git 根據新規則重新處理儲存庫中的所有檔案 [^3][^68][^71]。

```bash
# (可選) 先提交當前的工作變更
# git add . -u
# git commit -m "儲存進度，準備正規化換行符"

# 讓 Git 根據 .gitattributes 重新處理索引中的所有檔案
git add --renormalize .

# 查看哪些檔案的換行符被修改了 (可選)
# git status

# 提交正規化後的結果
git commit -m "正規化所有檔案的換行符"
```

此操作會更新 Git 的索引 (index)，確保之後的提交和檢出都符合 `.gitattributes` 的規範。執行 `git add --renormalize .` 後，`git status` 可能會顯示大量檔案被修改，這是正常的，因為 Git 正在統一換行符。

### 結論

透過在專案根目錄配置 `.gitattributes` 文件，設定 `* text=auto eol=lf`，可以強制 Git 在 `checkout` 時將所有文字檔轉換為 LF，並在 `commit` 時將文字檔以 LF 格式儲存於儲存庫中。同時，建議將 Fedora 使用者的本地全域 Git 設定 `core.autocrlf` 為 `input`，並將 `core.safecrlf` 設為 `true`，以提供額外的保障。最後，務必執行 `git add --renormalize .` 來更新現有儲存庫，使新設定生效。這種方法不需要規範 Windows 使用者的設定，即可在 Fedora 環境下達到預期的換行符處理效果。

{% editor_note() %}
我最後使用

```gitignore
[core]
    autocrlf = input
    safecrlf = warn
```

{% end %}

[^1]: [在git中自动把CRLF转换到LF的方法原创 - CSDN博客](https://blog.csdn.net/star19830909/article/details/136431031)
[^2]: [git换行符导致的提交整个文件的解决方案 - CSDN博客](https://blog.csdn.net/gooooooal/article/details/78963502)
[^3]: [Configuring Git to handle line endings - GitHub Docs](https://docs.github.com/articles/dealing-with-line-endings)
[^4]: [代码提交Git时CRLF与LF转换原创 - CSDN博客](https://blog.csdn.net/SuiXin_123/article/details/103810390)
[^5]: [Git 自动转换文件换行符 - 狂飙](https://networm.me/2022/05/22/git-eol-conversion/)
[^6]: [Confused about how to handle line endings in git repo](https://stackoverflow.com/questions/69522074/confused-about-how-to-handle-line-endings-in-git-repo)
[^9]: [Git在Windows下自动转换换行符LF CRLF 的解决方案原创](https://blog.csdn.net/qq_29687271/article/details/129904957)
[^13]: [Git 多平台换行符问题(LF or CRLF) - 旷世的忧伤](http://kuanghy.github.io/2017/03/19/git-lf-or-crlf)
[^14]: [git 乱改你的换行符？一句话设置让git 不再碰你某个文件的换行符](https://cloud.tencent.com/developer/article/2349002)
[^15]: [Git and normalization of line-endings - DEV Community](https://dev.to/kevinshu/git-and-normalization-of-line-endings-228j)
[^17]: [【Git】Git行结束符CRLF、LF自动转换全局 - 博客园](https://www.cnblogs.com/Flat-White/p/17428358.html)
[^20]: [配置Git 处理行结束符- GitHub 文档](https://docs.github.com/zh/get-started/getting-started-with-git/configuring-git-to-handle-line-endings)
[^25]: [LF will be replaced by CRLF”最详细解释+解决方案 - 知乎专栏](https://zhuanlan.zhihu.com/p/586324681)
[^32]: [Azure DevOps Server (TFS）中代码文件换行问题解决方案（Git)](https://www.cnblogs.com/danzhang/p/10197265.html)
[^34]: [Git Handles Carriage Returns Differently When Committed ...](https://docs.copado.com/articles/copado-ci-cd-publication/git-handles-carriage-returns-differently-when-committed-from-both-mac-and-windows)
[^38]: [Git因换行符不一致导致反复有修改记录- ！win - 博客园](https://www.cnblogs.com/xwwin/p/18300528)
[^39]: [Git中CRLF与LF的转换 - CSDN博客](https://blog.csdn.net/u011069294/article/details/103809307)
[^46]: [8.1.3. 换行符问题— GotGit](https://www.worldhello.net/gotgit/08-git-misc/040-eol.html)
[^47]: [Configuring Git to handle line endings - GitHub Docs](https://docs.github.com/articles/dealing-with-line-endings)
[^50]: [配置Git 处理行结束符- GitHub 文档](https://docs.github.com/zh/get-started/getting-started-with-git/configuring-git-to-handle-line-endings)
[^54]: [Git 多平台换行符问题(LF or CRLF) - 旷世的忧伤](http://kuanghy.github.io/2017/03/19/git-lf-or-crlf)
[^63]: [代码提交Git时CRLF与LF转换原创 - CSDN博客](https://blog.csdn.net/SuiXin_123/article/details/103810390)
[^68]: [配置 Git 处理行结束符 - GitHub 文档](https://docs.github.com/zh/get-started/getting-started-with-git/configuring-git-to-handle-line-endings)
[^70]: [CRLF vs. LF: Normalizing Line Endings in Git](https://www.aleksandrhovhannisyan.com/blog/crlf-vs-lf-normalizing-line-endings-in-git/)
[^71]: [Forces all line endings to LF in your git repo. - GitHub Gist](https://gist.github.com/ajdruff/16427061a41ca8c08c05992a6c74f59e)
[^72]: [How to handle line endings in git - CodeAndWeb](https://www.codeandweb.com/babeledit/tutorials/how-to-handle-line-endings-in-git)
[^73]: [Is there any way to 100% enforce certain line endings ... - Reddit](https://www.reddit.com/r/git/comments/cqgtgw/is_there_any_way_to_100_enforce_certain_line/)
[^77]: [What does "!eol" in gitattributes do? - Stack Overflow](https://stackoverflow.com/questions/21302670/what-does-eol-in-gitattributes-do)
[^81]: [gitattributes Documentation - Git](https://git-scm.com/docs/gitattributes/2.9.5)
[^85]: [Git tip: Add a .gitattributes file to deal with line endings](https://tsalikis.blog/posts/gitattributes/)
[^90]: [git: How do I get rid of "warning: CRLF will be replaced by LF ...](https://stackoverflow.com/questions/16873848/git-how-do-i-get-rid-of-warning-crlf-will-be-replaced-by-lf-without-disablin)
[^92]: [Mind the End of Your Line - Adaptive Patchwork](https://adaptivepatchwork.com/2012/03/01/mind-the-end-of-your-line/)
