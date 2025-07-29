+++
title = "Docker Build 快取失效指南：哪些 Dockerfile 指令會導致 Cache Miss"
description = "深入解析 Docker build 過程中導致快取失效的 Dockerfile 指令，包含 RUN、ENV、ARG、COPY 等指令的快取機制，以及如何透過最佳實踐提升建置效率並減少不必要的重建時間"
date = "2024-10-21T11:37:12Z"
updated = "2025-07-25T11:39:31.884Z"
draft = false

[taxonomies]
tags = [ "Container", "Containerfile" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/C89zWhcQnp5EUszyCzMz5R?invite=dOLYGeJyZJqVX>"
+++
{% editor_note() %}
小技巧:

我習慣在 CI 上將每次執行時的流水號以 RELEASE build-arg 傳入，並且在 **不希望使用到 cache** 的步驟前面插入這個 ARG。

請查看這個範本: [CI](https://github.com/jim60105/Containerfile-template/blob/2735f41dd6558dd154146ac2cfd5b2405e4da6fe/.github/workflows/container-reused-steps/action.yml#L73), [Containerfile](https://github.com/jim60105/Containerfile-template/blob/2735f41dd6558dd154146ac2cfd5b2405e4da6fe/fedora-toolbox/base.Containerfile#L83-L84)
{% end %}

{% chat(speaker="jim") %}
docker build 中 哪些 dockerfile 指令會導致不 cache
{% end %}

{% chat(speaker="felo") %}
在使用 `docker build` 建立映像檔時，Docker 會根據 Dockerfile 中的指令建立快取層，以加快後續的建置速度。然而，某些指令的使用方式會導致快取失效，迫使 Docker 重新執行相關步驟。以下是幾種常見會導致快取失效的 Dockerfile 指令：
{% end %}

****1. 使用 `RUN` 指令搭配動態內容****

當 `RUN` 指令中包含動態生成的內容，如時間戳或隨機數，會導致每次建置時該指令的快取層無法重用。例如：

```dockerfile
RUN echo $(date) > build_date.txt
```

每次建置時，`$(date)` 的結果都不同，導致該步驟的快取失效[^2][^3]。

****2. 修改 `ENV` 或 `ARG` 變數****

`ENV` 和 `ARG` 指令用於設置環境變數和建置參數。如果這些變數的值在不同的建置中發生變化，會導致後續指令的快取失效。例如：

```dockerfile
ARG CACHE_DATE=not_a_date
ENV BUILD_DATE=$CACHE_DATE
```

改變 `CACHE_DATE` 的值會使 `ENV` 指令無法使用快取[^2][^10]。

****3. 更改 `COPY` 或 `ADD` 指令的來源內容****

`COPY` 和 `ADD` 指令會將主機上的檔案或目錄複製到映像檔中。如果來源檔案發生變化，Docker 將無法使用快取，需重新執行這些指令。例如：

```dockerfile
COPY . /app
```

每當專案中的檔案有變動，`COPY` 指令的快取將失效，導致整個步驟重新執行[^3][^4]。

****4. 引用未固定的基礎映像版本****

在 Dockerfile 中使用未固定版本的基礎映像（如 `ubuntu:latest`）會導致每次建置時基礎映像可能更新，從而使快取失效。建議使用具體的版本標籤，例如 `ubuntu:20.04`，以確保一致性和快取的有效性[^1][^3]。

****5. 使用 `--no-cache` 標誌****

在建置映像時，使用 `--no-cache` 標誌會強制 Docker 忽略所有快取層，從頭開始執行所有指令。這對於確保每次建置都是全新的，但會增加建置時間[^4][^5][^7]。

```bash
docker build --no-cache -t my-image:my-tag .
```

****6. 引入不穩定的依賴或外部資源****

如果 Dockerfile 中的指令依賴於外部資源（如從網路下載檔案），而這些資源經常變動，會導致快取失效。例如：

```dockerfile
RUN wget https://example.com/file.tar.gz
```

每次建置時，如果 `file.tar.gz` 有更新，該指令的快取將無法使用[^3][^10]。

## 如何有效管理 Docker 快取

為了最大化利用 Docker 的快取機制，建議遵循以下最佳實踐：

- **固定基礎映像版本**：使用具體的版本標籤而非 `latest`。
- **優化指令順序**：將不常變動的指令放在前面，頻繁變動的放在後面，以減少快取失效的範圍。
- **利用多階段建置**：將不同的建置階段分開，避免不必要的快取失效。
- **合理使用 `ARG` 和 `ENV`**：僅在必要時改變變數值，以維持快取的有效性。

透過這些策略，可以有效減少不必要的建置時間，提升開發效率[^12][^10]。

[^1]: [Best practices | Docker Docs](https://docs.docker.com/build/building/best-practices/)
[^2]: [How can I prevent a Dockerfile instruction from being cached? - Stack Overflow](https://stackoverflow.com/questions/31782220/how-can-i-prevent-a-dockerfile-instruction-from-being-cached)
[^3]: [Build cache invalidation | Docker Docs](https://docs.docker.com/build/cache/invalidation/)
[^4]: [Building Dockerfile Without Cache | by Nikhil YN | Medium](https://medium.com/@nikhil.nagarajappa/building-dockerfile-without-cache-4d5eaf09a554)
[^5]: [How to build an image without the Docker cache | LinkedIn](https://www.linkedin.com/pulse/how-build-image-without-docker-cache-razorops)
[^7]: [How to build an image without the Docker cache](https://depot.dev/blog/docker-build-no-cache)
[^10]: [Docker Cache - How to Do a Clean Image Rebuild and Clear Docker's Cache](https://www.freecodecamp.org/news/docker-cache-tutorial/)
[^12]: [Using the build cache | Docker Docs](https://docs.docker.com/get-started/docker-concepts/building-images/using-the-build-cache/)
