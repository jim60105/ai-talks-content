+++
title = "Azure DevOps YAML Pipeline 權限控管與安全防護全解析"
description = "深入解析 Classic 與 YAML Pipeline 權限模型的本質差異，並揭示 YAML Pipeline 如何透過多層防護機制提升安全性。本文將帶你了解分支原則、PR 審核與範本控管的實務應用。想知道如何有效防止未授權變更？點擊了解更多！"
date = "2025-10-27T09:10:58Z"
updated = "2025-11-07T04:00:43Z"
draft = false

[taxonomies]
tags = [ "Azure DevOps", "DevOps", "Pipeline", "Security" ]
providers = [ "Felo Search" ]

[extra]
withAI = "<https://felo.ai/search/YJBiZjk43KBzCNwZjxKi9w?invite=dOLYGeJyZJqVX>"
+++
{% alert(edit=true) %}
原本要寫英文問句，寫到一半發現我英文太菜，就變成半英半中了😆
{% end %}
{% chat(speaker="jim") %}
About Azure DevOps Pipeline: YAML vs Classic Pipelines, I had heard that Classic Pipeline 可以設定獨立的權限控管，使得僅 Build Admin 可以變更 Pipeline 而 Contributor 不行；YAML Pipeline 只要能提交變更的 Contributor 都可能變更 Pipeline 內容，所以更推薦使用 Classic Pipeline.

我想請你研究官方文件確認它的真實性，告訴我是否能在 YAML Pipeline 中禁止一般 Contributor 變更 Pipeline YAML 檔案。
{% end %}

{% chat(speaker="felo") %}
您的觀察點出了 Classic 與 YAML Pipelines 權限控管模型的根本差異。

傳統 Classic Pipeline 的權限控管較為直接，但 YAML Pipeline 透過將管線定義程式碼化的方式，提供了更細緻且更符合現代 Git 工作流程的深度防禦策略。雖然一般 Contributor 理論上可以編輯儲存庫中的任何檔案，但 Azure DevOps 提供了多層次的機制來有效禁止他們任意變更 Pipeline YAML 檔案。

**關鍵在於，保護 YAML Pipeline 的重點已從單純的 UI 權限設定轉移到對原始程式碼儲存庫的治理。**

透過結合分支原則、必要拉取請求 (Pull Request)、程式碼審核、執行條件檢查以及 YAML 範本，不僅可以達成與 Classic Pipeline 相同的控管效果，甚至能實現更嚴格、更安全的 CI/CD 流程。直接修改 `Edit build pipeline` 權限對 YAML 檔案本身無效，但結合儲存庫原則可以完全鎖定其變更權限。
{% end %}

## 權限模型的根本差異：Classic vs. YAML

### Classic Pipelines 的權限模型

在傳統的 Classic Pipeline 中，管線的定義（包含任務、步驟、變數等）是作為一個獨立的物件儲存在 Azure DevOps 服務內，而非儲存庫中 [^48]。因此，其權限控管非常直觀：

* **直接物件權限**：您可以直接在管線的安全性設定中，針對特定使用者或群組，設定 `Edit build pipeline`（編輯建構管線）權限 [^79]。
* **Deny 優先原則**：將 Contributor 群組的 `Edit build pipeline` 權限設為 `Deny`，即可有效阻止他們修改管線定義，即使他們在其他群組中擁有 `Allow` 權限，`Deny` 規則依然會覆蓋一切 [^9][^93]。

這種方式簡單明瞭，因為它將管線的「定義權」與程式碼的「提交權」完全分開。

### YAML Pipelines 的權限模型

YAML Pipeline 的核心理念是「管線即程式碼」(Pipeline as Code) [^5][^15]。管線定義本身就是一個 YAML 檔案（通常是 `azure-pipelines.yml`），與您的應用程式程式碼一同存放在 Git 儲存庫中 [^51]。這個轉變導致了權限模型的根本性變化：

* **儲存庫權限為主**：對 YAML 檔案的修改權限，基本上等同於對該儲存庫的寫入權限（Git Push）[^83]。任何有權限推送變更到分支的開發人員，理論上都能修改 `azure-pipelines.yml` 檔案。
* **`Edit build pipeline` 權限的範疇**：在 YAML Pipeline 的情境下，UI 上的 `Edit build pipeline` 權限不再控制 YAML 檔案的 *內容*。它主要管理的是與管線相關的 *設定*，例如管線名稱、排程觸發器、UI 中的安全設定等，但無法阻止使用者在儲存庫中直接編輯 YAML 檔案並推送 [^23][^85]。這正是造成混淆的主要原因。

## YAML Pipeline 的潛在安全風險

您所擔憂的情境是真實存在的，且在社群中有廣泛討論。如果沒有適當的防護措施，一個惡意或無心的 Contributor 可能會利用其儲存庫權限帶來風險：

* **分支漏洞**：使用者可以在自己的功能分支 (feature branch) 或私人分支中修改 `azure-pipelines.yml` 檔案，例如加入竊取祕密金鑰或破壞環境的腳本 [^2][^83]。
* **繞過 PR 審核**：接著，該使用者可以手動觸發管線在此修改過的分支上執行。由於變更不在受保護的主分支（如 `main`）上，因此可能完全繞過 Pull Request (PR) 的審核機制，直接執行惡意程式碼 [^2][^83]。

## 保護 YAML Pipeline 的多層次防禦策略

雖然 Contributor 預設可以編輯 YAML 檔案，但 Azure DevOps 提供了一系列強大的工具來鎖定和保護它。這些策略的核心是將安全控制融入 Git 工作流程中。

### 1. 透過分支原則強制執行程式碼審核

這是最基本且最有效的防線。您可以針對重要的分支（例如 `main`、`releases/*`）設定分支原則，以確保沒有任何變更可以未經審查就合併進去 [^103]。

* **要求 Pull Request**：強制所有對受保護分支的變更都必須透過 PR 進行 [^59]。
* **設定必要審核者**：您可以設定當 PR 中包含對特定路徑（例如存放 `azure-pipelines.yml` 的資料夾）的變更時，必須自動加入特定的審核人員或群組（如 Build Admins 或資安團隊）[^2][^83]。這樣一來，任何對管線的修改都必須經過授權人員的批准。
* **限制直接推送**：為特定群組（例如 Build Admins）授予 `Bypass policies` 權限，讓他們在緊急情況下可以略過原則，而其他所有人都必須遵守 PR 流程 [^2][^83]。

下圖展示了如何在分支原則中針對特定路徑新增自動審核者：

### 2. 在 YAML 內部進行執行階段檢查

為了防堵在個人分支上執行惡意管線的漏洞，您可以在 `azure-pipelines.yml` 檔案的開頭加入條件檢查，限制此管線只能在指定的分支上執行。

* **檢查來源分支**：使用預定義變數 `Build.SourceBranchName` 來判斷當前執行的分支。如果分支名稱不符合預期（例如不是 `refs/heads/main`），則直接讓作業失敗 [^6]。

```yaml
# azure-pipelines.yml

trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
# 第一步：驗證分支是否為 main
- script: |
    echo "Checking branch: $(Build.SourceBranchName)"
    if [ "$(Build.SourceBranchName)" != "refs/heads/main" ]; then
      echo "##vso[task.logissue type=error]This pipeline can only be run on the main branch."
      exit 1
    fi
  displayName: 'Verify source branch'

# 後續的建構與部署步驟...
- script: echo "Running build steps on main branch..."
```

### 3. 使用 YAML 範本 (Templates) 實現集中控管

這是最高級也最安全的作法，它能將管線的核心邏輯抽取出來，由專門的團隊進行維護，而開發人員只能在受限的框架內進行操作 [^84]。

* **`extends` 範本**：您可以建立一個中央的、受高度保護的「範本儲存庫」。在這個儲存庫中定義管線的骨架（`extends` template），包含所有必要的安全掃描、部署流程和權限檢查 [^20][^66]。
* **強制使用範本**：開發人員的 `azure-pipelines.yml` 檔案被要求必須 `extends` 自這個中央範本。他們只能傳遞參數（如建構腳本、應用程式名稱），而無法修改核心的管線結構 [^84]。
* **拒絕不安全的操作**：範本甚至可以設計成能檢查傳入的步驟，如果發現包含不被允許的任務（例如直接執行腳本的 `CmdLine@2`），則會直接讓管線解析失敗，從而阻止執行 [^84]。

以下是一個範例，展示如何使用 `extends` 範本來限制可執行的任務：

```yaml
# 位於受保護的範本儲存庫：templates/secure-template.yml
parameters:
- name: userSteps
  type: stepList
  default: []

jobs:
- job: SecureBuild
  steps:
  - script: echo "Mandatory security scan..." # 強制執行的安全步驟
  
  # 遍歷開發人員傳入的步驟
  - ${{ each step in parameters.userSteps }}:
    # 檢查是否為被禁止的 script 任務
    - ${{ if contains(step, 'script') }}:
        # 若包含 script，則引發 YAML 語法錯誤來中斷管線
        'script-is-not-allowed': error 
    - ${{ else }}:
        # 若非 script，則正常插入步驟
        - ${{ step }}

  - script: echo "Mandatory signing step..." # 強制執行的簽署步驟
```

```yaml
# 位於開發人員的儲存庫：azure-pipelines.yml
# 強制繼承自安全範本
extends:
  template: secure-template.yml@templates # @templates 指向範本儲存庫
  parameters:
    userSteps:
    - task: Npm@1 # 允許的任務
      inputs:
        command: 'install'
    - script: 'echo "This will be rejected"' # 這個步驟會被範本拒絕，導致管線失敗
```

## 結論

您的觀察點出了 Classic 與 YAML Pipeline 在權限管理上的差異，但{{ cr(body="「YAML Pipeline 較不安全」的結論並不成立。") }}事實上，YAML Pipeline 透過將管線程式碼化的方式，將安全控制提升到了另一個層次。

* **控制點轉移**：對 YAML Pipeline 的保護，控制點從 Azure DevOps UI 的單一權限設定，轉移到了對 Git 儲存庫和程式碼本身的治理。
* **深度防禦**：您可以透過分支原則、PR 審核、執行階段分支檢查以及 YAML `extends` 範本，建立一個多層次的深度防禦體系。這套體系不僅能防止未經授權的 Contributor 修改生產環境的管線，還能從根本上杜絕他們在個人分支上執行惡意管線的風險。
* **最佳實踐**：最佳的安全實踐是結合上述所有策略：使用受保護的 `main` 分支並強制 PR 審核，同時要求所有部署管線都繼承自一個集中管理的安全範本。如此一來，即可確保所有管線都符合組織的安全與合規標準，其安全性遠超過傳統的 Classic Pipeline。

{% alert(edit=true) %}
YAML Pipeline 仰賴 Git flow 來防止惡意變更；而 Classic Pipeline 則是透過傳統的 RBAC 管控。

就我的個人觀點來說，**人類總是會犯錯，應該盡可能地將安全防護自動化、程式化，減少對人類判斷的依賴。** 將重要的 Pipeline 檔案放在 Contributor 可以直接修改的儲存庫中和日常變更混在一起，再仰賴人類去審核阻擋它，並不是一個理想的安全設計。但是將 Pipeline 檔案獨立到另一個儲存庫中，在小型專案團隊中也有點過度設計。

因此 Classic Pipeline 仍然有其存在的價值。
{% end %}

[^2]: [Protect Azure Pipeline Yaml File from Being Edited](https://stackoverflow.com/questions/60516460/protect-azure-pipeline-yaml-file-from-being-edited)
[^5]: [Customize your pipeline - Azure Pipelines | Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/pipelines/customize-pipeline?view=azure-devops)
[^6]: [How do I restrict pipelines to specific branches without using ...](https://stackoverflow.com/questions/69444811/how-do-i-restrict-pipelines-to-specific-branches-without-using-triggers)
[^9]: [关于权限、访问和安全组- Azure DevOps - Microsoft Learn](https://learn.microsoft.com/zh-cn/azure/devops/organizations/security/about-permissions?view=azure-devops)
[^15]: [[Azure DevOps] 使用Azure Pipelines 实现CI - dino.c - 博客园](https://www.cnblogs.com/dino623/p/continuous-integration-with-azure-pipelines.html)
[^20]: [How to use YAML templates for reusable and secure pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops)
[^23]: [My "Edit Build Pipeline" Permissions in Azure DevOps are set ...](https://stackoverflow.com/questions/72825800/my-edit-build-pipeline-permissions-in-azure-devops-are-set-to-deny-but-i-can-s)
[^48]: [給我來點自動化- Pipeline 簡介與策略思考 - iT 邦幫忙](https://ithelp.ithome.com.tw/m/articles/10327170)
[^51]: [Customize your pipeline - Azure Pipelines | Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/pipelines/customize-pipeline?view=azure-devops)
[^59]: [Disable creation of classic pipelines - Azure DevOps Blog](https://devblogs.microsoft.com/devops/disable-creation-of-classic-pipelines/)
[^66]: [How to use YAML templates for reusable and secure pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops)
[^79]: [How to restrict edit access to Azure pipeline?](https://developercommunity.visualstudio.com/content/problem/695989/how-to-restrict-edit-access-to-azure-pipeline.html)
[^83]: [git - Protect Azure Pipeline Yaml File from Being Edited - Stack Overflow](https://stackoverflow.com/questions/60516460/protect-azure-pipeline-yaml-file-from-being-edited)
[^84]: [How to use YAML templates for reusable and secure pipelines - Azure Pipelines | Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops)
[^85]: [My "Edit Build Pipeline" Permissions in Azure DevOps are ...](https://stackoverflow.com/questions/72825800/my-edit-build-pipeline-permissions-in-azure-devops-are-set-to-deny-but-i-can-s)
[^93]: [Can't edit pipelines even though I am the administrator.](https://developercommunity.visualstudio.com/t/cant-edit-pipelines-even-though-i-am-the-administr/1275797)
[^103]: [Git branch policies and settings - Azure Repos](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops)
