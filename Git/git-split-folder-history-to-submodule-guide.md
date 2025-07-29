+++
title = "一步步拆分 Git 資料夾歷史並轉換為 Submodule 指南"
description = "想將專案中的某個資料夾獨立成 Git Submodule？本指南詳解如何分離 commit 歷史、建立新倉庫並無縫導入主專案。掌握 subtree split、filter-repo 與日常管理技巧！"
date = "2025-05-31T17:07:16Z"
updated = "2025-07-28T09:01:25Z"
draft = false

[taxonomies]
tags = [ "Git" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/UqbWZeznB3uZARr3aTMD2e?invite=dOLYGeJyZJqVX>"
+++
{% editor_note() %}
我後來使用 `git filter-repo` 搞定部落格的 content 拆分，使得內容部落格更新和部落格程式碼的歷史不會混在一起。
{% end %}

{% chat(speaker="jim") %}
生成一篇指南，如何將 git commit 記錄拆開，將涉及變更了某一個指定的資料夾下檔案的 commit 全部分開到另一個樹? 我的目的是將某一個資料夾分開為 git submodule
{% end %}

{% chat(speaker="felo") %}
本指南旨在詳細說明如何將 Git 專案中特定資料夾的變更歷史記錄分離出來，並將該資料夾轉換為一個獨立的 Git Submodule。
{% end %}

這個過程主要包含兩個階段：首先，將目標資料夾及其完整的提交歷史提取到一個全新的 Git 倉庫；其次，在原始專案中移除該資料夾，並將新建立的倉庫作為 Submodule 引入。透過這種方式，可以實現更模組化的專案管理，讓主專案和子模組各自獨立發展和版本控制。

## 第 1 步：將目標資料夾的歷史記錄分離到獨立的 Git 倉庫

此步驟的目的是為您想要分離的資料夾建立一個全新的 Git 倉庫，同時完整保留其在原專案中的提交歷史記錄。

**選項 A：使用 `git subtree split`**

`git subtree split` 命令能夠從現有倉庫中提取一個子目錄的歷史記錄，並在一個新的分支中建立一個以該子目錄為根的全新專案歷史 [^3][^15]。

1. 在您的主專案倉庫根目錄執行以下命令。請將 `<folder_path>` 替換成您要分離的資料夾路徑（例如 `src/my_module`），並將 `<new_branch_name>` 替換為新分支的自訂名稱（例如 `feature/my_module_history`）：

    ```bash
    git subtree split -P <folder_path> -b <new_branch_name>
    ```

2. 建立一個新的資料夾，用於存放即將分離出來的子模組程式碼，並將其初始化為一個 Git 倉庫：

    ```bash
    mkdir ../<submodule_repo_name>
    cd ../<submodule_repo_name>
    git init
    ```

    請將 `<submodule_repo_name>` 替換為您子模組倉庫的名稱，`<original_repo_name>` 則是您原始主專案資料夾的名稱。

3. 從原專案剛剛建立的 `<new_branch_name>` 分支，將程式碼拉取到這個新的子模組倉庫中：

    ```bash
    git pull ../<original_repo_name> <new_branch_name>
    ```

    完成此步驟後，新的 `<submodule_repo_name>` 倉庫將擁有原資料夾的完整歷史記錄 [^3]。

4. 在您的 Git 代管服務（如 GitHub、GitLab 或 Bitbucket）上，為這個子模組建立一個新的遠端倉庫。

5. 將本地新建立的子模組倉庫連接到遠端倉庫，並推送程式碼：

    ```bash
    git remote add origin <remote_repository_url>
    git push -u origin main # 或 master，取決於您的預設分支名稱
    ```

    如果在執行 `git push` 時遇到 `refusing to merge unrelated histories` 的錯誤訊息，可以嘗試先執行 `git pull origin main --allow-unrelated-histories`（將 `main` 替換成您的遠端預設分支名），然後再重新推送 [^3]。

**選項 B：使用 `git filter-repo` (功能更強大，但需額外安裝)**

`git filter-repo`是 Git 官方推薦用於重寫歷史的工具，相較於舊有的 `git filter-branch`，它執行速度更快且使用上更安全 [^5]。使用前需先安裝此工具。

1. **重要：在進行任何歷史重寫操作之前，強烈建議您先備份原始倉庫。**

2. 為了安全起見，可以複製一份原始倉庫到新的位置，然後在新位置進行操作：

    ```bash
    cp -r <original_repo_name> <submodule_repo_name>_temp
    cd <submodule_repo_name>_temp
    ```

3. 執行 `git filter-repo` 命令，將指定的資料夾提升為倉庫的根目錄，並移除所有其他檔案及資料夾的歷史。請將 `<folder_path>` 替換成您要分離的資料夾路徑：

    ```bash
    git filter-repo --subdirectory-filter <folder_path>
    ```

    此命令會改寫當前操作倉庫的歷史記錄，使其只包含 `<folder_path>` 的內容及其相關的提交歷史，並將 `<folder_path>` 作為新倉庫的根目錄。

4. 同樣地，為這個子模組在您的 Git 代管服務上建立一個新的遠端倉庫。

5. 將這個經過歷史重寫的本地倉庫連接到遠端倉庫，並推送程式碼：

    ```bash
    git remote add origin <remote_repository_url>
    git push -u origin main # 或 master
    ```

## 第 2 步：在原專案中移除舊資料夾並添加 Submodule

在成功將目標資料夾的歷史分離到獨立倉庫後，接下來的步驟是在原始主專案中，用新建立的 Submodule 來取代原有的資料夾。

1. 返回到您的**原始主專案**倉庫的根目錄。

2. 從主專案中移除舊的資料夾。建議先從 Git 的追蹤中移除，然後再實際刪除資料夾檔案 [^1][^6][^90][^91]：

    ```bash
    git rm -r --cached <folder_path> # 從 Git 追蹤中移除，但保留本地檔案
    rm -rf <folder_path>             # 刪除實際的資料夾和檔案
    git commit -m "Removed <folder_path> to replace with submodule"
    ```

    另一種方式是使用 `git rm -rf <folder_path>`，這會直接刪除檔案並將刪除操作加入到 Git 的暫存區 [^10]。

3. 添加新的 Submodule。請將 `<remote_repository_url>` 替換成您在第 1 步中為子模組建立的遠端倉庫 URL，`<folder_path>` 通常應保持和原資料夾的名稱及路徑一致 [^4][^12][^91]：

    ```bash
    git submodule add <remote_repository_url> <folder_path>
    ```

    這個命令會執行以下操作：
    * 克隆子模組的遠端倉庫到主專案中指定的 `<folder_path>` 路徑。
    * 在主專案的根目錄下建立或更新 `.gitmodules` 檔案，此檔案會記錄子模組的相關資訊（如路徑和 URL）[^2][^6][^12][^70]。
    * 將 `.gitmodules` 檔案的變更和新加入的子模組資料夾（它本身是一個指向特定 commit 的 Git "link"）加入到 Git 的暫存區。

4. 提交將 Submodule 添加到專案的變更：

    ```bash
    git commit -m "Added <folder_path> as a submodule"
    ```

5. 推送主專案的變更到其遠端倉庫：

    ```bash
    git push
    ```

    在 GitHub 等平台上，您會注意到子模組資料夾的圖示與一般資料夾不同，點擊它可以直接跳轉到該子模組的獨立倉庫頁面 [^91]。

## 第 3 步：Submodule 的日常協作與管理

### 克隆包含 Submodule 的專案

當團隊成員或其他使用者克隆包含 Submodule 的主專案時，有以下幾種方式來獲取 Submodule 的實際內容：

* **一次性克隆主專案及所有 Submodule**：這是最推薦的方式，可以在克隆主專案時一併初始化並更新所有子模組 [^2][^18][^91]。

    ```bash
    git clone --recurse-submodules <main_project_repository_url>
    ```

* **分步獲取 Submodule**：如果已經克隆了主專案但未使用 `--recurse-submodules` 選項（此時 Submodule 對應的資料夾會是空的），則需要執行以下命令來初始化並拉取 Submodule 的內容 [^13][^18][^20][^91]。

    ```bash
    git submodule init      # 初始化本地 .git/config 檔案，註冊 .gitmodules 中定義的子模組
    git submodule update --recursive # 拉取所有子模組的資料，並遞迴更新巢狀子模組（如果有的話）
    ```

    `git submodule update --init --recursive` 也可以合併這兩步 [^91]。

### 更新 Submodule

主專案本身記錄的是 Submodule 在某個特定 commit 的參照。如果 Submodule 的遠端倉庫有了新的提交，主專案需要手動更新其對 Submodule 的參照。

* **方法一：進入 Submodule 目錄進行更新**
    1. 進入子模組所在的目錄：`cd <folder_path>`。
    2. 切換到您希望追蹤的分支（例如 `main` 或 `master`）：`git checkout main`。
    3. 拉取該分支的最新變更：`git pull`。
    4. 返回主專案的根目錄：`cd ..`。
    5. 主專案會偵測到 Submodule 指向的 commit ID 發生了變化。將此變更加入暫存區並提交：

        ```bash
        git add <folder_path>
        git commit -m "Updated submodule <folder_path> to latest commit"
        git push
        ```

        此時主專案的 `git diff` 會顯示 Submodule 的 commit ID 變更 [^8]。

* **方法二：在主專案目錄使用命令直接更新**

    ```bash
    git submodule update --remote <folder_path>
    ```

    此命令會查找 Submodule 遠端倉庫設定的追蹤分支（或預設分支）的最新 commit，並更新本地 Submodule 工作區。如果沒有指定 `<folder_path>`，則會嘗試更新所有 Submodule [^8][^18][^79]。
    更新後，同樣需要在主專案中執行 `git add <folder_path>` 和 `git commit` 來儲存 Submodule 指向新 commit ID 的變更。

### 在 Submodule 中進行開發

在 Submodule 中進行開發工作，並將變更推送回 Submodule 的遠端倉庫：

1. 進入 Submodule 的目錄：`cd <folder_path>` [^18]。
2. **重要**：執行 `git submodule update` 後，Submodule 通常會處於 "detached HEAD" (分離頭) 狀態，即 HEAD 直接指向一個 commit 而非分支。在此狀態下直接提交的變更，若未被特定分支引用，則在下次執行 `git submodule update` 時可能會遺失。因此，強烈建議在開始開發前先切換到一個已存在的分支，或建立一個新的分支 [^12][^27][^64]。

    ```bash
    git checkout main  # 或其他您想在其上開發的分支
    # 或建立並切換到新分支
    # git checkout -b new_feature_branch
    ```

3. 進行程式碼修改，然後像在一般 Git 倉庫中一樣執行 `git add`、`git commit`。
4. 將變更推送到 Submodule 自身的遠端倉庫：`git push`。
5. 返回主專案的根目錄。
6. 主專案會偵測到其追蹤的 Submodule commit ID 已不再是 Submodule 最新 HEAD 所指向的 commit。執行 `git add <folder_path>` 將此更新（即 Submodule 指向了新的 commit）加入暫存區。
7. 提交此變更到主專案：`git commit -m "Update submodule <folder_path> with new features"`。
8. 推送主專案的變更。

### 指定 Submodule 追蹤特定分支

在添加 Submodule 時，可以透過 `-b` 參數指定希望它追蹤的分支 [^7][^80]：

```bash
git submodule add -b <branch_name> <remote_repository_url> <folder_path>
```

如果 Submodule 已經存在，也可以編輯 `.gitmodules` 檔案，在對應的 Submodule 區塊中添加 `branch = <branch_name>` 欄位，然後執行 `git submodule sync` 和 `git submodule update --remote` 來使設定生效 [^7]。

## 注意事項

* **備份優先**：在執行任何涉及重寫 Git 歷史的操作（例如 `git filter-repo` 或 `git filter-branch`）之前，請務必完整備份您的原始倉庫，以防萬一發生意外。
* **協作影響**：重寫主專案的歷史（如果選擇了會影響主專案歷史的方法來分離資料夾）會改變 commit ID。如果您的主專案是多人協作的，這將會對其他協作者的本地倉庫造成困擾，可能需要他們進行強制拉取或 rebase。務必在執行此類操作前與團隊充分溝通。將資料夾分離為 Submodule 本身也會改變專案結構，需要團隊適應。
* **Submodule 的管理複雜性**：雖然 Git Submodule 提供了良好的模組化能力，但它們也引入了一定的管理複雜度（例如，處理 detached HEAD、更新流程等）。確保團隊所有成員都理解 Submodule 的工作方式及其潛在的陷阱是非常重要的 [^12][^64]。
* **Detached HEAD 狀態**：在子模組目錄中工作時，要特別留意 "detached HEAD" 狀態。`git submodule update` 命令會將子模組的 HEAD 指向父專案記錄的那個特定 commit ID，這通常不是任何分支的頂端。如果在 detached HEAD 狀態下進行了新的 commit，這些 commit 不屬於任何分支，若後續沒有將它們納入某個分支並更新父專案的引用，這些 commit 可能會在下次執行 `git submodule update` 時變得難以找回（實際上並未遺失，但不再被直接引用）。因此，在子模組中進行修改前，務必先 `git checkout <branch_name>` 切換到一個明確的分支 [^12][^27][^64]。

[^1]: [将git文件夹转换为子模块- 瑞诺拉- 博客园](https://www.cnblogs.com/ruinuola/articles/17595675.html)
[^2]: [Git中submodule的使用原创 - CSDN博客](https://blog.csdn.net/m2099797280/article/details/124486663)
[^3]: [git多模块项目拆分原创 - CSDN博客](https://blog.csdn.net/nikoHuang/article/details/103160744)
[^4]: [git submodule add子模块的添加 - CSDN博客](https://blog.csdn.net/Mark_md/article/details/121640755)
[^5]: [如何使用git submodule和git filter-repo从仓库中分离特定目录](https://tanjoe.github.io/posts/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8git-submodule%E5%92%8Cgit-filter-repo%E4%BB%8E%E4%BB%93%E5%BA%93%E4%B8%AD%E5%88%86%E7%A6%BB%E7%89%B9%E5%AE%9A%E7%9B%AE%E5%BD%95/)
[^6]: [Git: submodule 子模块简明教程](https://iphysresearch.github.io/blog/post/programing/git/git_submodule/)
[^7]: [git 使用submodule 如何指定分支原创 - CSDN博客](https://blog.csdn.net/weixin_43191327/article/details/135506655)
[^8]: [git submodule - 网易数帆- 博客园](https://www.cnblogs.com/163yun/p/9920777.html)
[^10]: [如何将现有git 仓库中的子目录分离为独立仓库并保留其提交历史](https://prinsss.github.io/splitting-a-subfolder-out-into-a-new-git-repository/)
[^12]: [子模組(Submodules) | GitPro - GitBook](https://willh.gitbook.io/gitpro/b71cee50e82413aeb78040ff34b9e8b6/ea86b3bc304f79534d2008bc8402d46c)
[^13]: [git submodule 子模块的基本使用原创 - CSDN博客](https://blog.csdn.net/DisMisPres/article/details/132604665)
[^15]: [Git 将子文件夹分离为一个新的库转载 - CSDN博客](https://blog.csdn.net/weixin_30535043/article/details/97228264)
[^18]: [Git Submodules 介绍（通俗易懂，总结了工作完全够用的 ...](https://cloud.tencent.com/developer/article/2136829)
[^20]: [Git 版本控制教學- submodule - MyApollo](https://myapollo.com.tw/blog/git-tutorial-submodule/)
[^27]: [子模块— git-reference 0.1 文档](https://git-reference.readthedocs.io/zh_CN/latest/Git-Tools/Submodules/)
[^64]: [子模組(Submodules) | GitPro - GitBook](https://willh.gitbook.io/gitpro/b71cee50e82413aeb78040ff34b9e8b6/ea86b3bc304f79534d2008bc8402d46c)
[^70]: [Git: submodule 子模块简明教程](https://iphysresearch.github.io/blog/post/programing/git/git_submodule/)
[^79]: [Git Submodules 介绍（通俗易懂，总结了工作完全够用的 ...](https://cloud.tencent.com/developer/article/2136829)
[^80]: [git 使用submodule 如何指定分支原创 - CSDN博客](https://blog.csdn.net/weixin_43191327/article/details/135506655)
[^90]: [将git文件夹转换为子模块 - 瑞诺拉 - 博客园](https://www.cnblogs.com/ruinuola/articles/17595675.html)
[^91]: [Git: submodule 子模块简明教程 | A Quest After Perspectives](https://iphysresearch.github.io/blog/post/programing/git/git_submodule/)
