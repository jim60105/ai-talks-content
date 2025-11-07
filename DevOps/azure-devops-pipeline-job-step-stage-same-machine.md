+++
title = "Azure DevOps Pipeline：Stages、Jobs、Steps 執行機器全解析"
description = "深入解析 Azure DevOps Pipeline 中 stages、jobs、steps 的執行機制。了解它們是否會在同一台機器上運作，以及如何配置才能確保任務在同一機器上執行。掌握正確設計 pipeline 的關鍵技巧！"
date = "2025-10-27T10:07:34Z"
updated = "2025-11-07T03:46:39Z"
draft = false

[taxonomies]
tags = [ "Azure DevOps", "DevOps", "Pipeline" ]
providers = [ "Felo Search" ]

[extra]
withAI = "<https://felo.ai/search/AC8hKuBumJKJcVeQpuKcok?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
For azure devops pipeline, Do jobs run on the same machine?  
Do steps run on the same machine?  
Do stages run on the same machine?

Do I have to write them in the same job if those tasks must run on the same machine?
{% end %}

{% chat(speaker="felo") %}
在 Azure DevOps Pipeline 中，不同層級的任務在機器上的執行方式有明確的規則。以下詳細說明 stages、jobs 和 steps 的執行機制。
{% end %}

## Steps 在同一機器上執行

**Steps 必定在同一台機器上執行**。同一個 job 中的所有 steps/tasks 都會在同一個 agent 機器上執行[^1]。每個 step 在自己的 process 中執行，並且可以存取 agent 上的 pipeline workspace[^7]。這意味著環境變數不會在 steps 之間保留，但檔案系統的變更會被保留[^7]。

## Jobs 可能在不同機器上執行

**Jobs 預設會平行執行，可能在不同的機器上**。除非設定了 `dependsOn` 值，否則 Azure DevOps YAML pipeline 中的 jobs 會平行執行[^14]。每個 agent 一次只能執行一個 job[^1][^14]。

如果您需要確保多個 jobs 在同一台 agent 上執行，可以使用 **demands 和 capabilities** 機制[^12]。重要的是使用每個建構 agent 獨有的 capability，這樣可以確保特定 stage 中的所有 jobs 都在同一個 agent 上執行[^12]。

## Stages 在不同機器上執行

**Stages 預設會在不同的機器上執行**。每個 stage 在 Azure DevOps pipeline 中都會在獨立的 agent 上執行，並且預設會獲得唯一的工作目錄[^3]。例如，stage 1 的工作資料夾是 `_work/1/s`，而 stage 2 的工作資料夾是 `_work/2/s`[^3]。

這種設計確保了隔離性，防止 stages 之間的檔案衝突[^3]。Azure DevOps 為每個 stage 分配新的臨時工作資料夾，因為 agents 可以平行執行、在不同機器上執行，或在不同時間執行[^3]。

### 在 Stages 之間共享檔案

如果需要在 stages 之間傳遞檔案，應該使用 **artifacts 機制**[^3]:

1. 在 stage 1 結束時使用 `PublishBuildArtifacts` 或 `PublishPipelineArtifact` task 上傳檔案
2. 在 stage 2 中使用 `DownloadPipelineArtifact` task 下載這些檔案到自己的工作目錄[^3]

## 必須在同一機器上執行的任務配置

**是的，如果任務必須在同一台機器上執行，您需要將它們寫在同一個 job 中**[^1]。這是確保任務在同一個 agent 上執行的最可靠方式。

如果您有多個 jobs 需要在同一台機器上執行，可以:

- 使用 demands 和 capabilities 來指定特定的 agent[^12]
- 在 stage 層級指定 `pool`，該 stage 中的所有 jobs 都會使用該 pool，除非在 job 層級另外指定[^6]

總結來說，Azure DevOps Pipeline 的執行層級從小到大為:steps(同一機器)→ jobs(可能不同機器)→ stages(不同機器)。如果需要確保任務在同一台機器上執行，最簡單的方式就是將它們放在同一個 job 中。

[^1]: [Azure DevOps YAML Pipelines - Part 7 - Nathan Nellans](https://www.nathannellans.com/post/azure-devops-yaml-pipelines-part-7)
[^3]: [Why are my two stages running on different work folder](https://learn.microsoft.com/en-us/answers/questions/5590603/why-are-my-two-stages-running-on-different-work-fo)
[^6]: [Stages in Azure Pipelines - Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/stages?view=azure-devops)
[^7]: [steps definition - Azure Pipelines - Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema/steps?view=azure-pipelines)
[^12]: [Ensure dependent jobs run in the same self hosted agent](https://stackoverflow.com/questions/79130691/ensure-dependent-jobs-run-in-the-same-self-hosted-agent)
[^14]: [Jobs in Azure Pipelines - Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/phases?view=azure-devops)
