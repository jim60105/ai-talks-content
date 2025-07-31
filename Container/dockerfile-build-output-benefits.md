+++
title = "使用 Dockerfile 建置專案並導出編譯檔案的優勢"
description = "解析為何在 CI/CD 流程中以 Dockerfile 建置專案並利用 --output 導出編譯檔案，能提升環境一致性、自動化與部署效率。"
date = "2024-12-26T23:11:20Z"
updated = "2025-07-25T14:40:22.581Z"
draft = false

[taxonomies]
tags = [ "Container", "Containerfile", "Docker", "DevOps" ]
providers = [ "Felo Search" ]


[extra]
withAI = "<https://felo.ai/search/GcewVfEAvuidgSQeJeSZXP?invite=dOLYGeJyZJqVX>"
+++
{% alert(edit=true) %}
有同事問我，為什麼我的 CI 是在 Containerfile 裡建置，而不是直接寫在 CI yml 中於 runner 上直接建置 (那是個不會產出 container image 的專案)，這跟一般做法不太一樣。

在 container 中建置確實是有著很多好處，而不是單純因為我喜歡 container 而已。😉
{% end %}

{% chat(speaker="jim") %}
explain what is the befinit of building project in dockerfile and --output the complied files
{% end %}

{% chat(speaker="felo") %}
在使用 Dockerfile 建立專案時，有幾個主要的好處，特別是在編譯和輸出檔案方面：
{% end %}

## **使用 Dockerfile 的好處**

1. **環境一致性**：
   使用 Dockerfile 可以確保在不同環境中執行的應用程式具有相同的依賴和配置。這樣可以避免因環境差異而導致的問題，特別是在開發和生產環境之間。

2. **自動化構建**：
   Dockerfile 提供了一種自動化構建映像的方法。開發者可以定義所有必要的步驟，從安裝依賴到編譯代碼，這樣每次構建都能夠重現相同的結果。

3. **多階段構建**：
   使用多階段構建可以有效地減少最終映像的大小。開發者可以在第一階段中編譯應用程式，然後只將必要的檔案複製到最終映像中，這樣可以避免將不必要的編譯工具和檔案包含在內[^2][^11]。

4. **便於分享和部署**：
   將 Dockerfile 與源代碼一起分享，可以讓其他開發者輕鬆重建相同的環境，而不需要手動配置。這對於團隊合作和持續集成非常重要。

## **使用 `--output` 輸出編譯檔案的好處**

1. **靈活性**：
   使用 `--output` 標誌可以將編譯結果直接導出到主機的指定位置，而不僅僅是生成 Docker 映像。這對於需要將編譯結果作為獨立檔案使用的情況非常有用，例如生成可執行檔或其他輸出檔案[^8][^19]。

2. **減少資源消耗**：
   透過直接導出編譯檔案，可以避免不必要的映像層和資源消耗，特別是在只需要最終編譯結果而不需要整個映像的情況下。

3. **簡化工作流程**：
   將編譯結果導出到主機上，可以簡化後續的部署或測試流程，因為開發者可以直接使用這些檔案，而不需要從 Docker 容器中提取。

總結來說，使用 Dockerfile 來構建專案不僅能夠提高環境的一致性和自動化程度，還能通過 `--output` 標誌靈活地管理編譯結果，這對於現代軟體開發流程是非常有利的。

[^2]: [Dockerfile overview | Docker Docs](https://docs.docker.com/build/concepts/dockerfile/)
[^8]: [Export binaries | Docker Docs](https://docs.docker.com/build/building/export/)
[^11]: [Part 8: Image-building best practices | Docker Docs](https://docs.docker.com/get-started/workshop/09_image_best/)
[^19]: [Feature Request: Allow `docker build` to output additional files from stages · Issue #2577 · docker_cli](https://github.com/docker/cli/issues/2577)
