+++
title = "uv sync 生產級指令：凍結依賴、排除開發套件一次搞懂"
description = "深入解析 `uv sync --frozen --no-dev --no-editable` 的每個選項，確保生產環境穩定一致。學會如何只安裝鎖定版本、跳過開發套件與可編輯模式。最後再教你一招，把專案同時裝成可呼叫工具！"
date = "2025-04-08T15:06:06Z"
updated = "2025-07-27T12:49:03.774Z"
draft = false

[taxonomies]
tags = [ "Python", "uv", "Package Manager" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/RF5tZ3XFB9w6TZrBtcXTXM?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
explain `uv sync --frozen --no-dev --no-editable`
{% end %}

{% chat(speaker="felo") %}
`uv sync --frozen --no-dev --no-editable` 是一個用於管理 Python 專案依賴的命令，這個命令的各個選項有其特定的功能，以下是對這些選項的詳細解釋：
{% end %}

## **命令選項解釋**

- **`--frozen`**:
  - 此選項表示在同步過程中，將不會更新任何依賴的版本。這意味著，只有在當前鎖定檔（lockfile）中已經指定的版本會被安裝，而不會嘗試安裝更新的版本。這對於確保環境的一致性非常重要，特別是在生產環境中。

- **`--no-dev`**:
  - 使用此選項會告訴 `uv` 在同步時不安裝開發依賴（dev dependencies）。開發依賴通常是用於開發和測試的包，這樣做可以減少生產環境中的不必要包，從而提高安全性和性能。

- **`--no-editable`**:
  - 此選項表示不會安裝可編輯模式的包。可編輯模式通常用於開發過程中，允許開發者在本地修改代碼並立即看到變更。如果使用此選項，所有包都將以標準模式安裝，而不是可編輯模式。

## **總結**

綜合來看，執行 `uv sync --frozen --no-dev --no-editable` 命令的目的是在不更新任何依賴版本的情況下，安裝專案的生產依賴，並且不包括開發依賴和可編輯安裝的包。這樣的做法有助於保持生產環境的穩定性和一致性，特別是在需要確保所有依賴版本不變的情況下。這對於部署和維護生產應用程式來說是非常重要的。

{% chat(speaker="jim") %}
I use `uv sync --frozen --no-dev --no-editable` to install my project. I want to install this project as tool too with the same env, What's the uv tool command to do this?
{% end %}

{% chat(speaker="felo") %}
根據您的問題，您想要在使用 `uv sync --frozen --no-dev --no-editable` 安裝專案後，也將該專案作為工具安裝到相同的環境中。以下是關於這個需求的詳細說明。
{% end %}

## uv sync 命令解析

首先，讓我們了解您目前使用的命令：

- `uv sync --frozen --no-dev --no-editable` 這個命令會：
  - `--frozen`：使用鎖定檔案中的版本，不嘗試更新依賴[^1003]
  - `--no-dev`：不安裝開發依賴，只安裝生產環境所需的依賴[^1015]
  - `--no-editable`：以非可編輯模式安裝，這意味著不會依賴於源代碼[^1004]

## 安裝專案作為工具

要將您的專案作為工具安裝到相同的環境中，您可以使用 `uv tool install` 命令。以下是建議的步驟：

1. 首先確保您的專案已經正確設置了 `pyproject.toml` 文件，包含了必要的元數據和入口點。

2. 使用以下命令安裝您的專案作為工具：

```bash
uv tool install --system .
```

這裡的 `.` 表示當前目錄（您的專案目錄）。`--system` 選項表示將工具安裝到系統 Python 環境中，這樣可以確保與您之前使用 `uv sync` 命令安裝的環境保持一致[^1004]。

## 在 Docker 中的應用

{% editor_note() %}
這是我自己在容器中處理 uv 的方法:  
[kohya-ss-gui.Dockerfile - jim60105/Containerfile-template](https://github.com/jim60105/Containerfile-template/blob/ccba4a92584e486d1efa061cee4f99677f0f8ce3/python/ML/kohya-ss-gui.Dockerfile#L54-L100)
{% end %}

如果您是在 Docker 環境中工作，您可能需要確保工具的二進制目錄在 PATH 中。您可以通過以下方式實現：

```dockerfile
ENV PATH="/root/.local/bin:$PATH"
```

或者，您可以使用 `uv tool dir --bin` 命令來確定工具二進制目錄的位置，然後將其添加到 PATH 中[^1004]。

## 注意事項

1. 確保您的 `pyproject.toml` 文件正確配置了 `[project.scripts]` 部分，以定義您的工具的入口點。

2. 如果您需要在同一環境中安裝多個工具，可以重複使用 `uv tool install` 命令。

3. 在 Docker 環境中，您可能需要考慮使用緩存來提高構建性能：

```dockerfile
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev --no-editable && \
    uv tool install --system .
```

這樣可以確保您的專案及其工具都安裝在相同的環境中，並且使用相同的依賴版本。

[^1003]: [Commands | uv](https://docs.astral.sh/uv/reference/cli/)
[^1004]: [Using uv in Docker - Astral Docs](https://docs.astral.sh/uv/guides/integration/docker/)
[^1015]: [Locking and syncing | uv - Astral Docs](https://docs.astral.sh/uv/concepts/projects/sync/)
