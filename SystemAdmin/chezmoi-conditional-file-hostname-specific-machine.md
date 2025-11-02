+++
title = "Chezmoi 檔案僅限特定主機部署技巧"
description = "學會如何利用 chezmoi 的 .chezmoiignore 樣板與條件式樣板功能，讓檔案只在特定主機上存在。本文提供詳細步驟與比較，協助你安全管理多機環境下的專屬檔案。點擊深入了解最佳實踐！"
date = "2025-11-02T09:07:17Z"
updated = "2025-11-02T09:25:08Z"
draft = false

[taxonomies]
tags = [ "System Admin" ]
providers = [ "Felo Search" ]

[extra]
withAI = "<https://felo.ai/search/92aYu5ejeuAzFQLtJH6YNM?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
I'm using chezmoi, I need to add a file that only apply to specific machine.

The file is `~/.local/bin/brave` and the condition is `if eq .chezmoi.hostname "Jim-Fedora"`.

My chezmoi dotfile repo was used on several different machine, this file MUST NOT apply on other machine.

Please research and teach me how to do this.
{% end %}

{% chat(speaker="felo") %}

要在 `chezmoi` 中管理一個僅適用於特定機器的檔案，主要有兩種策略。

第一種，也是最直接的方法，是利用 `.chezmoiignore` 檔案。這個檔案本身可以是一個樣板，允許您根據主機名稱等變數，有條件地忽略某些檔案，從而防止它們被應用到非目標機器上。

第二種方法是將檔案本身轉換為樣板，並在其中嵌入條件邏輯。透過這種方式，只有在滿足特定條件時（例如，主機名稱匹配），樣板才會產生內容；否則，它會產生一個空字串，`chezmoi` 會將此視為刪除目標檔案的指令，從而確保該檔案不會存在於其他機器上。

對於僅控制檔案是否存在的需求，`.chezmoiignore` 方法通常更為清晰和具宣告性。
{% end %}

`chezmoi` 的核心功能之一是其強大的樣板系統，它能讓您跨多個異質環境管理您的 dotfiles[^10][^40][^41]。當您需要一個檔案（例如一個可執行腳本）只存在於某一台特定機器時，您可以利用這個樣板系統來實現精確控制[^28][^59]。

## 方法一：使用 `.chezmoiignore` 進行條件式忽略 (建議方法)

控制檔案在不同機器上部署的最直接方法是使用 `.chezmoiignore` 檔案[^51]。這個檔案位於您的 `chezmoi` 來源目錄中，其作用類似於 `.gitignore`，但它本身也是一個樣板，可以根據變數動態決定要忽略哪些檔案[^27][^51]。

這個方法的邏輯是「反向」的：您不是指定在哪裡 *包含* 檔案，而是指定在哪裡 *忽略* 它[^51]。要讓 `~/.local/bin/brave` 這個檔案只出現在主機名稱為 `Jim-Fedora` 的機器上，您需要在 `.chezmoiignore` 中設定一條規則，告訴 `chezmoi` 在所有主機名稱 *不等於* `Jim-Fedora` 的機器上忽略該檔案。

**實作步驟：**

1. **新增檔案至 chezmoi：**
    首先，像平常一樣將您的檔案新增到 `chezmoi` 的管理中。`chezmoi` 會將其複製到來源目錄。

    ```sh
    chezmoi add ~/.local/bin/brave
    ```

2. **建立或編輯 `.chezmoiignore` 檔案：**
    在您的 `chezmoi` 來源目錄（通常是 `~/.local/share/chezmoi`）中，建立或開啟 `.chezmoiignore` 檔案[^66]。

3. **新增條件式忽略規則：**
    在 `.chezmoiignore` 檔案中，加入以下內容。這段程式碼使用 `ne` (not equal) 函式來檢查當前機器的 `.chezmoi.hostname` 是否 *不是* `Jim-Fedora`。如果是，則將 `brave` 腳本的路徑加入忽略列表[^51]。

    ```go
    {{/* 在所有非 Jim-Fedora 的機器上忽略 brave 腳本 */}}
    {{ if ne .chezmoi.hostname "Jim-Fedora" -}}
    .local/bin/brave
    {{- end }}
    ```

    *注意：`.chezmoiignore` 中的路徑是相對於 `chezmoi` 來源目錄的目標路徑。*

4. **套用變更：**
    提交您的變更到版本控制，然後在您的所有機器上執行 `chezmoi apply`。只有在 `Jim-Fedora` 上，`~/.local/bin/brave` 檔案會被建立或更新。在其他所有機器上，如果該檔案存在，`chezmoi` 會將其移除；如果不存在，則會忽略它。

您可以使用 `chezmoi ignored` 命令來驗證哪些檔案目前被忽略[^51]。

## 方法二：在檔案樣板中使用條件式內容

另一種方法是將檔案本身變成一個樣板，並將其全部內容包裹在一個 `if` 條件區塊中[^19][^51]。`chezmoi` 有一個特性：如果一個樣板執行後的結果是空字串，`chezmoi` 將會從目標位置移除該檔案[^51]。這完美地滿足了您的需求。

**實作步驟：**

1. **將檔案新增為樣板：**
    使用 `add` 命令時加上 `--template` 旗標，`chezmoi` 會自動在來源目錄中建立一個以 `.tmpl` 結尾的樣板檔案[^9][^50]。

    ```sh
    chezmoi add --template ~/.local/bin/brave
    ```

    這會在您的來源目錄中建立一個類似 `private_dot_local/bin/brave.tmpl` 的檔案（`private_` 前綴表示這是一個可執行檔案）。

2. **編輯樣板檔案：**
    使用 `chezmoi edit ~/.local/bin/brave` 來編輯這個新建立的樣板檔案[^51]。將其內容修改為如下結構：

    ```go
    {{ if eq .chezmoi.hostname "Jim-Fedora" -}}
    #!/bin/sh
    # 這是 brave 腳本的內容
    # 只有在 Jim-Fedora 主機上才會產生這段內容
    echo "This is the brave script running on Jim-Fedora"
    exit 0
    {{- end }}
    ```

    - `{{ if eq .chezmoi.hostname "Jim-Fedora" }}`：這個條件判斷只有在主機名稱完全匹配時才為真[^50]。
    - `{{- end }}`：`{{-` 和 `-}}` 中的連字號 `-` 用於去除樣板標籤前後的空白字元（包含換行符），確保在條件不滿足時，檔案內容是完全空的[^50][^51]。

3. **套用變更：**
    執行 `chezmoi apply`。在 `Jim-Fedora` 上，`~/.local/bin/brave` 將會被建立並包含腳本內容。在任何其他機器上，由於樣板產生空字串，`chezmoi` 將確保 `~/.local/bin/brave` 檔案不存在。

## 方法三：使用 `include` 函式組織檔案內容

如果您的機器專用檔案內容非常龐大，或者您想為多個不同的機器提供不同的檔案版本，使用 `include` 函式會讓您的來源目錄更整潔[^7][^51]。

這個方法是方法二的變體。您將實際的腳本內容儲存在一個單獨的檔案中，然後在主樣板檔案中有條件地 `include` 它。

**實作步驟：**

1. 在您的 `chezmoi` 來源目錄中建立兩個檔案：
    - **內容檔案** (例如 `.brave_script_for_fedora`)：

      ```sh
      #!/bin/sh
      # 這是 brave 腳本的內容
      echo "This is the brave script running on Jim-Fedora"
      exit 0
      ```

    - **主樣板檔案** (`private_dot_local/bin/brave.tmpl`)：

      ```go
      {{ if eq .chezmoi.hostname "Jim-Fedora" -}}
      {{   include ".brave_script_for_fedora" -}}
      {{- end }}
      ```

2. 當您執行 `chezmoi apply` 時，`chezmoi` 會處理 `private_dot_local/bin/brave.tmpl`。在 `Jim-Fedora` 上，它會將 `.brave_script_for_fedora` 的內容包含進來並建立目標檔案。在其他機器上，它會產生空內容並移除目標檔案[^51]。

## 方法比較

| 方法 | 優點 | 缺點 | 適用情境 |
| --- | --- | --- | --- |
| **`.chezmoiignore`** | - 邏輯清晰，具宣告性[^51]<br>- 檔案內容保持純淨，不含樣板語法<br>- 集中管理忽略規則 | - 忽略邏輯是「反向」的，初次使用可能不直觀[^51] | 最適合簡單地控制單一檔案在某些機器上是否存在。 |
| **條件式樣板** | - 邏輯直接內嵌於檔案中<br>- 易於擴展，可在同一檔案中為不同機器定義不同內容 | - 檔案內容與樣板邏輯混合，可能降低可讀性 | 檔案需要在不同機器上存在但內容不同，或僅在某些機器上存在。 |
| **`include` 函式** | - 保持來源目錄結構清晰[^51]<br>- 易於管理多個大型、機器專用的檔案版本 | - 檔案結構較為複雜，需要管理多個檔案 | 當機器專用的設定非常龐大，或需要為多台機器維護不同版本的檔案時。 |

## 驗證與除錯

在套用變更之前，您可以使用以下 `chezmoi` 命令來驗證您的設定是否正確：

- `chezmoi diff`：在目標機器上執行此命令，它會顯示將要建立或修改 `~/.local/bin/brave` 的差異。在其他機器上，它應該顯示移除該檔案的差異（如果它已存在）或不顯示任何內容。
- `chezmoi execute-template`：用於快速測試樣板片段的輸出，例如 `chezmoi execute-template '{{ .chezmoi.hostname }}'`[^50][^51]。
- `chezmoi cat ~/.local/bin/brave`：在套用前預覽 `chezmoi` 將為指定檔案產生的最終內容。

[^7]: [Include other files in templates · Issue #787](https://github.com/twpayne/chezmoi/issues/787)
[^9]: [Dotfiles: Managing machine-specific ~/.gitconfig with chezmoi ...](https://jpcaparas.medium.com/dotfiles-managing-machine-specific-gitconfig-with-chezmoi-user-defined-template-variables-400071f663c0)
[^10]: [Take back your dotfiles with Chezmoi](https://fedoramagazine.org/take-back-your-dotfiles-with-chezmoi/)
[^19]: [Managing dotfiles with Chezmoi | Nathaniel Landau](https://natelandau.com/managing-dotfiles-with-chezmoi/)
[^27]: [Create the same file differently, depending on a condition?](https://github.com/twpayne/chezmoi/discussions/2277)
[^28]: [Templating - chezmoi](https://chezmoi.io/user-guide/templating/)
[^40]: [Managing dotfiles with Chezmoi | Nathaniel Landau](https://natelandau.com/managing-dotfiles-with-chezmoi/)
[^41]: [twpayne/chezmoi: Manage your dotfiles across ... - GitHub](https://github.com/twpayne/chezmoi)
[^50]: [Templating - chezmoi](https://chezmoi.io/user-guide/templating/)
[^51]: [Manage machine-to-machine differences - chezmoi](https://chezmoi.io/user-guide/manage-machine-to-machine-differences/)
[^59]: [Manage machine-to-machine differences - chezmoi](https://chezmoi.io/user-guide/manage-machine-to-machine-differences/)
[^66]: [Managing dotfiles with Chezmoi - Adrian Vasile](https://adrianvasile.dev/2023/01/12/dotfiles-using-chezmoi.html)
