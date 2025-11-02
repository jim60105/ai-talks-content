+++
title = "用 chezmoi 同步 DevContainer：打造專開發環境"
description = "學會如何運用 chezmoi 工具同步 DevContainer 的個人化設定，結合 zsh 與 zinit 提升開發效率。本文詳述優點、實作指令與自動化流程。讓你在任何容器中都能快速擁有熟悉的高效開發環境。"
date = "2025-05-29T19:41:40Z"
updated = "2025-07-28T05:03:42Z"
draft = false

[taxonomies]
tags = [ "DevOps", "Zsh", "System Admin" ]
providers = [ "Felo Search" ]

[extra]
withAI = "<https://felo.ai/search/GqYU9zzk74TNxaNG6dJ5LJ?invite=dOLYGeJyZJqVX>"
featured = true
+++
{% chat(speaker="jim") %}
生成一篇指南，內容是如何使用 chezmoi 來同步 devcontainer 環境。指南中要包含如此做的優點以及實際操作指令。另外提及 zsh 以 zinit 管理器安裝應用的說明，和 chezmoi 搭配使用可以在 devcontainer 達成什麼事。
{% end %}

{% chat(speaker="felo") %}
本指南旨在說明如何運用 `chezmoi` 工具來同步化 `devcontainer` 開發環境中的個人化配置，特別是針對 Zsh shell 環境以及如何透過 `zinit` 插件管理器來安裝及管理工具。
{% end %}

透過 `chezmoi`，開發者能夠確保在不同的 `devcontainer` 實例中擁有一致且客製化的 dotfiles（如 `.zshrc`, `.gitconfig` 等設定檔）。結合 `zinit` 管理 Zsh 插件，可以進一步提升終端機的效率與使用體驗，例如自動建議、語法高亮和自訂佈景主題。這種組合讓開發者在任何 `devcontainer` 中都能快速啟動一個熟悉且功能強大的開發環境，大幅提升工作流程的順暢度與生產力。

## Dev Containers 與 Chezmoi 簡介

### Dev Containers

Dev Containers（開發容器）提供了一個完整且隔離的開發環境，它將應用程式程式碼、執行時期、系統工具、函式庫和擴充功能打包在一個 Docker 容器內 [^2][^25][^30][^40]。這種方法的主要優勢包括：

* **一致性**：確保團隊中每位成員以及在不同機器上的開發環境都相同，減少「在我機器上可以跑」的問題 [^2][^51][^53][^104]。
* **隔離性**：專案的依賴項與本機系統的配置完全隔離，避免了版本衝突和系統污染 [^33][^53]。
* **可重現性**：每次啟動容器都能獲得完全相同的環境，確保建置與測試結果的一致性 [^33][^53]。
* **快速上手**：新專案或新成員可以跳過繁瑣的環境設定過程，直接開始編碼 [^2][^33][^53]。

### Chezmoi

`chezmoi` 是一個跨多種不同機器的 dotfiles（點檔案，通常指 Unix-like 系統中以 `.` 開頭的設定檔）管理工具 [^14][^29][^48][^54]。它的主要特色與優勢有：

* **跨平台管理**：輕鬆同步和管理在不同作業系統（Linux, macOS, Windows）和機器上的 dotfiles [^29][^31][^52]。
* **模板引擎**：支援使用模板來處理不同機器間的配置差異 [^31][^52][^54]。
* **秘密管理**：能與密碼管理器整合（如 1Password, pass, LastPass, Bitwarden）或使用 GPG 加密來安全地管理敏感資訊，如 API 金鑰或 token [^31][^34][^52][^54]。
* **單一執行檔**：`chezmoi` 本身是一個 Go 語言撰寫的獨立執行檔，無需額外執行時期依賴 [^4][^54][^107]。

## 為何在 Dev Containers 中使用 Chezmoi？

在 Dev Containers 中整合 `chezmoi` 可以帶來多項好處：

* **個人化與標準化的平衡**：Dev Containers 確保了專案所需工具和依賴的一致性，而 `chezmoi` 則允許開發者在此基礎上快速套用個人的 shell 配置、編輯器設定、Git 設定等，實現個人化的開發體驗，而不會影響專案的標準化設定 [^5][^28][^51][^100][^123]。
* **快速環境重建**：當需要在新的 Dev Container 實例（例如在新專案、Codespaces 或遠端容器）中工作時，`chezmoi` 可以迅速將熟悉的個人化配置套用，大幅縮短環境適應時間 [^54]。
* **簡化多容器配置**：對於同時處理多個專案、使用多個不同 Dev Container 的開發者而言，`chezmoi` 提供了一個集中的方式來管理和同步所有這些環境的共通個人配置 [^11][^42][^55]。
* **提升開發者體驗與效率**：一個配置良好且熟悉的 shell 環境、Git 別名、編輯器快捷鍵等，都能顯著提升開發效率和舒適度 [^2][^12]。`chezmoi` 確保這些配置隨處可用。

## 使用 Chezmoi 同步 Devcontainer 環境：實戰指南

以下步驟將引導您設定 `chezmoi` 以同步您的 Devcontainer 環境。

### 步驟 1：在本機設定 Chezmoi

1. **安裝 Chezmoi**：
    在您的本機（例如 macOS 或 Linux），可以使用套件管理器安裝 `chezmoi`。以 Homebrew 為例 [^29]：

    ```bash
    brew install chezmoi
    ```

    或者，使用官方腳本安裝 [^29]：

    ```bash
    sh -c "$(curl -fsLS get.chezmoi.io)"
    ```

2. **初始化 Chezmoi 並連結到 Git 儲存庫**：
    `chezmoi` 會將您的 dotfiles 存放在一個 Git 儲存庫中。執行以下指令，將 `<your-github-username>` 替換成您的 GitHub 用戶名 [^52]：

    ```bash
    chezmoi init your-github-username
    ```

    這會在 `~/.local/share/chezmoi` 建立一個由 Git 版本控制的目錄，並將其連結到您 GitHub 上的 `dotfiles` 儲存庫（如果不存在，`chezmoi` 會提示您建立）。
3. **將您的 Dotfiles 加入 Chezmoi**：
    將您想要管理的設定檔加入 `chezmoi`。例如，加入 `.zshrc` 和 `.gitconfig` [^52]：

    ```bash
    chezmoi add ~/.zshrc
    chezmoi add ~/.gitconfig
    ```

    完成後，提交並推送變更到您的 dotfiles Git 儲存庫：

    ```bash
    chezmoi cd  # 進入 chezmoi 的原始目錄
    git add .
    git commit -m "Add initial zshrc and gitconfig"
    git push
    ```

### 步驟 2：設定 Dev Container 以整合 Chezmoi

有兩種主要方法可以在 Dev Container 中整合 `chezmoi`：

* **方法一：利用 VS Code 的 Dotfiles Repository 功能**
    VS Code 和 GitHub Codespaces 支援在容器啟動時自動從指定的 Git 儲存庫複製 dotfiles 並執行安裝腳本 [^11][^100][^123][^124][^125]。

    1. **VS Code 設定**：
        在 VS Code 的使用者設定 (JSON) 中，加入以下設定，指向您的 dotfiles 儲存庫：

        ```json
        {
            "dotfiles.repository": "your-github-username/dotfiles",
            "dotfiles.targetPath": "~/dotfiles",
            "dotfiles.installCommand": "~/dotfiles/install.sh"
        }
        ```

        * `dotfiles.repository`：您的 dotfiles 儲存庫 (例如 `github_username/dotfiles`)。
        * `dotfiles.targetPath`：dotfiles 儲存庫將被複製到容器中的路徑。
        * `dotfiles.installCommand`：複製完成後將執行的腳本路徑。

    2. **建立 `install.sh` 腳本**：
        在您的 dotfiles Git 儲存庫的根目錄下，建立一個名為 `install.sh` 的腳本 [^3][^99][^124][^125]。此腳本負責在容器內安裝 `chezmoi`（如果需要）並套用 dotfiles。

        ```bash
        #!/bin/bash
        set -e # 如果任何指令失敗，立即退出

        # 定義變數
        BIN_DIR="${HOME}/.local/bin" # chezmoi 將安裝於此
        CHEZMOI_BIN="${BIN_DIR}/chezmoi"
        DOTFILES_SOURCE_DIR="${HOME}/dotfiles" # VS Code 通常將 dotfiles 複製到此處

        echo "Personalizing container..."

        # 建立 ~/.local/bin 目錄 (如果不存在)
        mkdir -p "${BIN_DIR}"

        # 安裝 chezmoi (如果尚未安裝)
        if [ ! -f "${CHEZMOI_BIN}" ]; then
            echo "Installing chezmoi..."
            if command -v curl >/dev/null 2>&1; then
                sh -c "$(curl -fsLS get.chezmoi.io)" -- -b "${BIN_DIR}"
            elif command -v wget >/dev/null 2>&1; then
                sh -c "$(wget -qO- get.chezmoi.io)" -- -b "${BIN_DIR}"
            else
                echo "Error: curl or wget is required to install chezmoi." >&2
                exit 1
            fi
        else
            echo "chezmoi is already installed."
        fi

        # 套用 dotfiles
        # --source 指向 VS Code 複製 dotfiles 的目錄
        # 最後一個參數是您的 chezmoi 儲存庫名稱 (通常是您的 GitHub 用戶名)
        echo "Applying dotfiles from ${DOTFILES_SOURCE_DIR}..."
        "${CHEZMOI_BIN}" init --apply --source "${DOTFILES_SOURCE_DIR}" $(basename $DOTFILES_SOURCE_DIR) # or your specific repo name if different

        echo "Dotfiles applied. Container personalized."
        ```

        確保此腳本是可執行的：`chmod +x install.sh`，然後將其提交到您的 dotfiles 儲存庫。

    3. **Chezmoi 設定檔 (`.chezmoi.toml.tmpl`)**：
        為了在非互動式環境（如 Dev Container）中順利運作，並正確指定 dotfiles 的來源目錄，您的 `chezmoi` 設定檔 (通常是 `~/.config/chezmoi/chezmoi.toml` 的模板版本，存放在 dotfiles 儲存庫中，例如 `private_dot_config/chezmoi/chezmoi.toml.tmpl`) 可能需要如下調整 [^3][^99][^124]：

        ```toml
        {{- $isCodespaces := env "CODESPACES" | not | not -}}
        {{- $isRemoteContainer := env "REMOTE_CONTAINERS" | not | not -}}
        # sourceDir 是 VS Code/Codespaces 複製 dotfiles 的位置
        sourceDir = {{ .chezmoi.sourceDir | quote }}

        [data]
            codespaces = {{ $isCodespaces }}
            remotecontainer = {{ $isRemoteContainer }}
            name = "Your Name"
        {{- if or $isCodespaces $isRemoteContainer }}
            email = "your-email@example.com" # 非互動式環境中設定預設值
        {{- else }}
            email = {{ promptString "email" | quote }}
        {{- end }}
        ```

        這段設定會偵測是否在 Codespaces 或 Remote Containers 環境中，並相應地調整行為，例如避免互動式提示。`sourceDir = {{ .chezmoi.sourceDir | quote }}` 這一行特別重要，它告訴 `chezmoi` 在容器內何處尋找 dotfiles 的原始檔案。

* **方法二：使用 Dev Container Feature `chezmoi`**
    社群提供了一些 Dev Container Features，可以直接在 `devcontainer.json` 中啟用 `chezmoi` [^28][^38][^51]。

    ```json
    {
        "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
        "features": {
            "ghcr.io/rio/features/chezmoi:1": {
                "repo": "https://github.com/your-github-username/dotfiles.git"
            }
        }
    }
    ```

    這會自動處理 `chezmoi` 的安裝和 dotfiles 的套用。查閱特定 feature 的文件以了解詳細配置選項。

### 步驟 3：在 Dev Container 中自訂 Zsh 與 Zinit

在 `chezmoi` 管理的 dotfiles 中，最核心的通常是 shell 的設定檔，例如 `.zshrc`。

1. **安裝 Zsh (與 Oh My Zsh - 可選)**：
    許多基礎的 Dev Container 映像檔可能不包含 Zsh，或者您想使用特定版本的 Zsh。`ghcr.io/devcontainers/features/common-utils` feature 可以協助安裝 Zsh 和 Oh My Zsh，並將 Zsh 設為預設 shell [^97][^122]。
    在 `devcontainer.json` 的 `features` 區塊加入：

    ```json
    "ghcr.io/devcontainers/features/common-utils": {
        "installZsh": true,
        "installOhMyZsh": true, // 可選，Zinit 可獨立運作或與 OMZ 整合
        "configureZshAsDefaultShell": true,
        "username": "vscode" // 或您在容器中使用的使用者名稱
    }
    ```

2. **Zinit：輕量級 Zsh 插件管理器**
    Zinit 是一個功能強大且執行快速的 Zsh 插件管理器，以其 Turbo 模式（非同步載入插件以加速 shell 啟動）和彈性著稱 [^57][^93][^96]。

3. **透過 `chezmoi` 管理的 `.zshrc` 設定 Zinit**：
    在由 `chezmoi` 管理的 `.zshrc` (模板) 中加入 Zinit 的安裝與設定腳本。這樣每次 `chezmoi apply` 後，Zinit 就會被正確設定。
    Zinit 自動安裝腳本範例 (加入到您的 `.zshrc` 中) [^59][^93][^94]：

    ```zsh
    # ~/.zshrc

    # Zinit 安裝 (如果 ZINIT_HOME 不存在)
    ZINIT_HOME="${XDG_DATA_HOME:-${HOME}/.local/share}/zinit/zinit.git"
    if [[ ! -d "$ZINIT_HOME" ]]; then
      echo "Installing Zinit..."
      command mkdir -p "$(dirname "$ZINIT_HOME")" && command chmod g-rwX "$(dirname "$ZINIT_HOME")"
      command git clone https://github.com/zdharma-continuum/zinit.git "$ZINIT_HOME" && \
        echo "Zinit installation successful." || \
        echo "Zinit installation failed."
    fi

    source "${ZINIT_HOME}/zinit.zsh"
    autoload -Uz _zinit
    (( ${+_comps} )) && _comps[zinit]=_zinit

    # --- Zinit Plugins ---
    # 語法高亮
    zinit light zdharma-continuum/fast-syntax-highlighting
    # 自動建議
    zinit light zsh-users/zsh-autosuggestions
    # 增強補全
    zinit light zsh-users/zsh-completions

    # 範例：安裝 Powerlevel10k 佈景主題 (一個流行的 Zsh 佈景主題)
    # zinit ice depth"1" # Git clone depth 1 for faster clone
    # zinit light romkatv/powerlevel10k
    # ((isAuthenticated && p10k_theme_variant == "lean")) && p10k reload

    # 範例：安裝 fzf (模糊搜尋工具)
    # zinit ice from"gh-r" as"program"
    # zinit light junegunn/fzf

    # 載入 Oh My Zsh 的插件 (如果需要)
    # zinit snippet OMZP::git # 載入 Oh My Zsh 的 git 插件
    ```

    `zinit light ...` 用於輕量載入插件。Zinit 也支援 `ice` 修飾詞來進行更細緻的設定，例如 `as"command"` 或 `from"gh-r"` 可以從 GitHub Releases 安裝二進位工具 [^93][^96]。

### Chezmoi、Zsh 和 Zinit 的協同效益

當 Dev Container 啟動時：

1. VS Code (或 Dev Container feature) 複製您的 dotfiles 儲存庫到容器內。
2. `install.sh`（或 feature 的內部邏輯）執行，安裝 `chezmoi` 並執行 `chezmoi apply`。
3. `chezmoi` 將您 Git 儲存庫中的最新版 dotfiles（包含 `.zshrc`）套用到容器的正確位置。
4. 當您開啟終端機，Zsh 啟動並讀取由 `chezmoi` 管理的 `.zshrc`。
5. `.zshrc` 中的 Zinit 腳本會執行：
    * 如果 Zinit 尚未安裝，則會自動下載並安裝 Zinit。
    * Zinit 接著會根據 `.zshrc` 中的 `zinit light ...` 或其他 Zinit 指令，非同步下載、安裝及載入您指定的 Zsh 插件和工具 [^57][^93]。
6. 最終成果：您會得到一個完全客製化、功能豐富且高效的 Zsh 環境，包含您偏好的佈景主題（如 Powerlevel10k、Agnoster [^6][^112][^122][^125]）、指令自動建議、語法高亮、自訂別名、函數，以及透過 Zinit 安裝的其他命令列工具。所有這些配置都是版本控制的，並且能在任何新的 Dev Container 中自動重現。

### 成果展示與提示

{% alert(edit=true) %}
歡迎查看我的 dotfiles repo: [jim60105/dotfiles](https://github.com/jim60105/dotfiles)

它是本文的配置成果再加上各種改造的產物 😆
{% end %}

* Dev Container 成功啟動後，您的終端機應該已經是配置好的 Zsh 環境。
* **VS Code 終端機設定**：若 Zsh 未自動成為預設 shell，您可能需要在 VS Code 的設定中 (通常是 `devcontainer.json` 內的 `customizations.vscode.settings`) 指定：

    ```json
    "terminal.integrated.defaultProfile.linux": "zsh",
    "terminal.integrated.profiles.linux": {
        "zsh": {
            "path": "/bin/zsh" // 或 zsh 的實際路徑
        }
    }
    ```

* **字型支援**：若您使用像 Powerlevel10k 或某些 Oh My Zsh 佈景主題，它們可能需要 Nerd Fonts 或 Powerline Fonts 來正確顯示圖示和符號。您需要在 `devcontainer.json` 中設定終端機字型，並確保該字型已安裝在容器內或透過 feature 安裝 [^122][^125]。

    ```json
    "terminal.integrated.fontFamily": "MesloLGM Nerd Font" // 範例字型
    ```

    安裝字型通常可以在 Dockerfile 中處理，或透過 `postCreateCommand` 執行腳本安裝 [^122][^125]。

<div hidden>
    ```mermaid
    graph TD
        A[Dev Container 啟動] --> B{Dotfiles機制定序};
        B -- VS Code Dotfiles 功能 --> C[複製 Dotfiles Repo];
        B -- Dev Container Feature --> D[Feature 執行];
        C --> E[執行 install.sh];
        D --> F[Chezmoi 自動化];
        E --> G[安裝/設定 Chezmoi];
        F --> G;
        G --> H[chezmoi apply];
        H --> I[套用 .zshrc 及其他設定檔];
        I --> J[Zsh 啟動];
        J --> K[讀取 .zshrc];
        K --> L[Zinit 初始化並載入插件];
        L --> M[個人化、高效的 Zsh 環境];
    ```
</div>

[^2]: [Simplifying Open Source Contributions with Dev Containers](https://theindiecoder.cloud/posts/dev-containers-and-open-source/)
[^3]: [Containers and VMs - chezmoi](https://chezmoi.io/user-guide/machines/containers-and-vms/)
[^4]: [Developer guide - chezmoi](https://chezmoi.io/developer-guide/)
[^5]: [Custom features for individual users · devcontainers - GitHub](https://github.com/orgs/devcontainers/discussions/60)
[^6]: [VSCode devcontainer with zsh, oh-my-zsh and agnoster theme](https://medium.com/@jamiekt/vscode-devcontainer-with-zsh-oh-my-zsh-and-agnoster-theme-8adf884ad9f6)
[^11]: [chezmoi and VS Code remote containers dotfile support #760](https://github.com/twpayne/chezmoi/issues/760)
[^12]: [Ultimate Guide to Dev Containers - Daytona](https://www.daytona.io/dotfiles/ultimate-guide-to-dev-containers)
[^14]: [Chezmoi - Manage your dotfiles across multiple diverse ...](https://news.ycombinator.com/item?id=32636051)
[^25]: [Where do you run your code? - an intro to devcontainers](https://blog.theredguild.org/where-do-you-run-your-code/)
[^28]: [Adopting Dotfiles for Codespaces and DevContainers](https://blog.v-lad.org/adopting-dotfiles-for-codespaces-and-dev-containers/)
[^29]: [A Step-by-Step Guide to Managing Your Dotfiles with Chezmoi](https://medium.com/@sssanjaya/dotfiles-with-chezmoi-4b5a3b503f36)
[^30]: [Developing inside a Container - Visual Studio Code](https://code.visualstudio.com/docs/devcontainers/containers)
[^31]: [Development Environment Setup with Chezmoi - Daniel Schmidt](https://danielmschmidt.de/posts/2024-07-28-dev-env-setup-with-chezmoi/)
[^33]: [Ultimate Guide to Dev Containers - Daytona](https://www.daytona.io/dotfiles/ultimate-guide-to-dev-containers)
[^34]: [Why use chezmoi?](https://chezmoi.io/why-use-chezmoi/)
[^38]: [rio/features: A collection of devcontainer features - GitHub](https://github.com/rio/features)
[^40]: [Automating Developer Environments with Dev Containers](https://shinesolutions.com/2024/10/21/automating-developer-environments-with-dev-containers/)
[^42]: [chezmoi and VS Code remote containers dotfile support #760](https://github.com/twpayne/chezmoi/issues/760)
[^48]: [Managing dotfiles with chezmoi - | Daniel Stoddart](https://stoddart.github.io/2024/09/08/managing-dotfiles-with-chezmoi.html)
[^51]: [Adopting Dotfiles for Codespaces and DevContainers | Vlad's Blog](https://blog.v-lad.org/adopting-dotfiles-for-codespaces-and-dev-containers/)
[^52]: [Development Environment Setup with Chezmoi | DanielMSchmidt.de](https://danielmschmidt.de/posts/2024-07-28-dev-env-setup-with-chezmoi/)
[^53]: [Ultimate Guide to Dev Containers](https://www.daytona.io/dotfiles/ultimate-guide-to-dev-containers)
[^54]: [Why use chezmoi? - chezmoi](https://chezmoi.io/why-use-chezmoi/)
[^55]: [chezmoi and VS Code remote containers dotfile support · Issue #760 · twpayne/chezmoi · GitHub](https://github.com/twpayne/chezmoi/issues/760)
[^57]: [zdharma-continuum/zinit: Flexible and fast ZSH plugin manager](https://github.com/zdharma-continuum/zinit)
[^59]: [A hands-on guide to setting up zsh, oh my zsh, asdf, and ...](https://dev.to/girordo/a-hands-on-guide-to-setting-up-zsh-oh-my-zsh-asdf-and-spaceship-prompt-with-zinit-for-your-development-environment-91n)
[^93]: [GitHub - zdharma-continuum/zinit: 🌻 Flexible and fast ZSH plugin manager](https://github.com/zdharma-continuum/zinit)
[^94]: [🚀 A hands-on guide to setting up zsh, oh my zsh, asdf, and spaceship prompt with zinit for your development environment - DEV Community](https://dev.to/girordo/a-hands-on-guide-to-setting-up-zsh-oh-my-zsh-asdf-and-spaceship-prompt-with-zinit-for-your-development-environment-91n)
[^96]: [加速你的 zsh —— 最强 zsh 插件管理器 zplugin/zinit 教程 - Aloxaf's Blog](https://www.aloxaf.com/2019/11/zplugin_tutorial/)
[^97]: [Add and Customize Oh My Zsh in a Linux Development ...](https://www.josephguadagno.net/2025/03/27/add-and-customize-oh-my-zsh-in-a-linux-development-container)
[^99]: [Containers and VMs - chezmoi](https://chezmoi.io/user-guide/machines/containers-and-vms/)
[^100]: [Custom features for individual users · devcontainers - GitHub](https://github.com/orgs/devcontainers/discussions/60)
[^104]: [Simplifying Open Source Contributions with Dev Containers](https://theindiecoder.cloud/posts/dev-containers-and-open-source/)
[^107]: [Developer guide - chezmoi](https://chezmoi.io/developer-guide/)
[^112]: [VSCode devcontainer with zsh, oh-my-zsh and agnoster theme](https://medium.com/@jamiekt/vscode-devcontainer-with-zsh-oh-my-zsh-and-agnoster-theme-8adf884ad9f6)
[^122]: [Add and Customize Oh My Zsh in a Linux Development Container - JosephGuadagno.net](https://www.josephguadagno.net/2025/03/27/add-and-customize-oh-my-zsh-in-a-linux-development-container)
[^123]: [Custom features for individual users · devcontainers · Discussion #60 · GitHub](https://github.com/orgs/devcontainers/discussions/60)
[^124]: [Containers and VMs - chezmoi](https://chezmoi.io/user-guide/machines/containers-and-vms/)
[^125]: [VSCode devcontainer with zsh, oh-my-zsh and agnoster theme | by Jamie Thomson | Medium](https://medium.com/@jamiekt/vscode-devcontainer-with-zsh-oh-my-zsh-and-agnoster-theme-8adf884ad9f6)
