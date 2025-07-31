+++
title = "3 步驟快速切換 Git 子模組到另一個 Fork 儲存庫"
description = "只需編輯 .gitmodules、同步並更新，就能讓子模組指向你的 fork，完整流程一次看懂！"
date = "2025-04-07T12:50:19Z"
updated = "2025-07-27T12:21:08.522Z"
draft = false

[taxonomies]
tags = [ "Git", "DevOps" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/bxRn55eiJVn59BGB9LQBq7?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
how to change github submodule to target another fork repo?
{% end %}

{% chat(speaker="felo") %}
Git 子模組是一種在主要專案中包含其他 Git 倉庫的方式，有時候我們需要將子模組指向自己的 fork 版本而非原始倉庫。以下是完成這項任務的詳細步驟。
{% end %}

## 修改子模組指向的倉庫 URL

### 方法一：編輯 .gitmodules 檔案

1. 使用文字編輯器開啟 `.gitmodules` 檔案[^1][^3][^8]
2. 找到您想要修改的子模組部分，並更新 URL 為您的 fork 倉庫地址[^1][^3]

   ```
   [submodule "some-library"]
     path = some-library
     url = https://github.com/yourusername/some-library.git
   ```

3. 儲存檔案後，執行以下命令同步變更[^1][^3][^8]

   ```
   git submodule sync
   ```

   這會更新 `.git/config` 檔案中的子模組 URL 資訊[^1]

### 方法二：使用 git config 命令

您也可以直接使用 git config 命令修改子模組 URL[^8][^13]：

```
git config --file=.gitmodules submodule.<子模組名稱>.url https://github.com/yourusername/your-fork-repo.git
```

例如：

```
git config --file=.gitmodules submodule.themes/hugo-coder.url https://github.com/yourusername/hugo-coder.git
```

## 更新子模組

修改 URL 後，您需要更新子模組以使變更生效：

1. 同步子模組資訊[^8][^13]

   ```
   git submodule sync
   ```

2. 更新子模組[^8][^9]

   ```
   git submodule update --init --recursive --remote
   ```

## 完整流程範例

假設您想將名為 "some-library" 的子模組從原始倉庫切換到您自己的 fork：

1. 編輯 `.gitmodules` 檔案，將 URL 改為您的 fork 倉庫
2. 執行 `git submodule sync` 同步變更
3. 執行 `git submodule update --init --recursive` 更新子模組
4. 提交變更到您的主倉庫

   ```
   git add .gitmodules
   git commit -m "將子模組指向我的 fork 倉庫"
   git push origin main
   ```

## 注意事項

1. 修改 URL 並同步後，子模組仍會指向相同的提交，只是來源倉庫不同[^1]
2. 如果您需要使用 fork 倉庫中的新提交，需要進入子模組目錄，切換到所需分支，然後在主倉庫中提交這個變更[^1]
3. 在較新版本的 Git 中，您可以使用 `git submodule set-url` 命令來更改子模組 URL[^1]

## 移除並重新添加子模組的方法

如果上述方法不起作用，您也可以考慮完全移除子模組並重新添加您的 fork 版本[^5]：

1. 移除子模組

   {% alert(edit=true) %}
   我每次移除 submodule 總是忘記 deinit 😅
   {% end %}

   ```
   git submodule deinit <子模組路徑>
   git rm <子模組路徑>
   git commit -m "移除舊的子模組"
   ```

2. 添加您的 fork 作為新的子模組

   ```
   git submodule add https://github.com/yourusername/your-fork-repo.git <子模組路徑>
   git commit -m "添加 fork 版本作為子模組"
   ```

透過以上步驟，您可以成功將 Git 子模組指向您自己的 fork 倉庫，以便進行自定義修改或開發。

[^1]: [Swap git submodule with own fork - Stack Overflow](https://stackoverflow.com/questions/11637175/swap-git-submodule-with-own-fork)
[^3]: [Git Submodules — GCHP 14.5.3 documentation](https://gchp.readthedocs.io/en/stable/reference/git-submodules.html)
[^5]: [How to Fork a Repository, Make Changes, and Update a ...](https://www.yopa.page/blog/2024-04-20-how-to-fork-a-repository-make-changes-and-update-a-submodule-in-your-main-project.html)
[^8]: [Changing Git Submodule Repository to other URL/Branch](https://dev.to/serhatteker/changing-git-submodule-repository-to-other-url-branch-356p)
[^9]: [How To Add and Update Git Submodules | by Daniel Hsu](https://medium.com/@nightheronry/how-to-add-and-update-git-submodules-dc1ba035e63b)
[^13]: [How to change the remote repository for a git submodule?](https://stackoverflow.com/questions/913701/how-to-change-the-remote-repository-for-a-git-submodule)
