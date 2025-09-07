+++
title = "KDE Plasma 設定檔轉移全攻略：位置、備份與遷移技巧"
description = "想將 KDE Plasma 桌面設定從舊電腦完整搬到新電腦？本文詳解設定檔位置、備份方法與遷移工具，並解析常見問題與解決對策。助你輕鬆無痛還原個人化桌面體驗！"
date = "2025-09-06T09:25:53Z"
updated = "2025-09-07T14:46:41Z"
draft = false

[taxonomies]
tags = [ "KDE Plasma", "Linux", "System Admin" ]
providers = [ "Felo Search" ]

[extra]
withAI = "<https://felo.ai/search/a5cRjmpd3EBQ3tx8txhQKX?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
Find the location of KDE Plasma configuration files and provide a complete guide to transfer them from one computer to another.
{% end %}

{% chat(speaker="felo") %}
本研究報告旨在提供一份關於 KDE Plasma 桌面環境設定檔位置的完整分析，並詳細闡述如何將這些設定從一台電腦轉移至另一台電腦。
{% end %}

研究發現，KDE Plasma 的設定檔並未集中存放於單一目錄，而是分散在使用者家目錄下的多個隱藏資料夾中，主要是 `~/.config` 和 `~/.local/share` [^12][^17][^24]。這種分散式結構使得手動備份與遷移變得複雜 [^14][^25][^47]。

轉移設定主要有兩種策略：手動複製和使用專用工具。手動方法提供最大程度的控制，使用者可以精確選擇要備份的檔案，但此方法容易出錯且可能遺漏重要設定 [^62]。專用工具，如 Konsave（命令列工具）和 Plasma Config Saver（桌面小工具），則大幅簡化了此過程，透過自動化打包和還原，提高了遷移的可靠性與效率 [^2][^16][^41]。

無論採用何種方法，遷移至不同硬體的電腦時，特別需要注意顯示器設定等硬體相關的組態檔，這些檔案可能需要手動重設以避免登入問題 [^8]。本報告將深入探討設定檔的具體位置、詳細的遷移步驟，以及遷移後可能遇到的問題與解決方案。

## KDE Plasma 設定檔結構與位置

KDE Plasma 的設定檔採用一種分散式架構，自 Plasma 5 模組化以來，設定檔不再儲存於單一資料夾中 [^17][^38][^56]。這種設計雖然有其架構上的優勢，但也為使用者備份與遷移帶來了挑戰 [^25][^50]。

### 主要設定目錄

使用者個人化的 KDE Plasma 設定主要存放在其家目錄 (`~/`) 下的兩個隱藏目錄中：

* **`~/.config`**: 這是最主要的設定檔存放目錄。絕大多數 KDE 應用程式和 Plasma 桌面本身的設定都儲存在這裡，通常以 `.rc` 結尾的檔案形式存在，例如 `kwinrc`（視窗管理器設定）或 `dolphinrc`（Dolphin 檔案管理器設定）[^17][^24][^34]。
* **`~/.local/share`**: 此目錄主要存放使用者資料，包括 Plasma 的佈景主題、圖示、桌面小工具以及部分應用程式的資料 [^12][^24][^30]。
* **`~/.cache`**: 此目錄存放快取資料，用於加速應用程式啟動和運作。在進行備份或遷移時，應*排除*此目錄，因為其內容是暫時性的且可以重新產生 [^15][^51]。

### 關鍵設定檔識別

雖然設定檔數量繁多，但了解幾個核心檔案的位置對於手動管理至關重要。以下表格列出了一些關鍵的設定檔及其用途：

| 檔案/目錄路徑 | 用途描述 |
| :--- | :--- |
| `~/.config/plasma-org.kde.plasma.desktop-appletsrc` | 儲存所有面板 (Taskbar)、桌面小工具 (Widgets) 和桌面佈局的設定 [^35][^38]。 |
| `~/.config/kwinrc` | KWin 視窗管理器的設定，包括視窗特效、標題列按鈕和視窗規則。 |
| `~/.config/kglobalshortcutsrc` | 全域鍵盤快捷鍵設定 [^3]。 |
| `~/.config/kwinoutputconfig.json` | Plasma 6 及更新版本中的顯示器和螢幕佈局設定 [^8]。 |
| `~/.config/dolphinrc` | Dolphin 檔案管理器的設定，如檢視模式、分割面板等 [^3]。 |
| `~/.config/konsolerc` | Konsole 終端機模擬器的設定，包括外觀設定檔、字型等。 |
| `~/.config/kdeconnect/` | KDE Connect 的設定，包括已配對的裝置資訊 [^62]。 |
| `~/.config/kdeglobals` | 包含顏色配置、字型、圖示主題等全域外觀設定。 |

一個由社群維護的 GitHub 儲存庫 `shalva97/kde-configuration-files` 提供了更詳盡的 KDE 設定檔位置清單，可作為進階參考 [^23][^32][^62]。

### 設定檔格式

KDE 的設定檔（通常是 `.rc` 檔案）使用一種基於純文字的 INI 格式，具備良好的可讀性和可編輯性 [^11][^39]。其主要特點包括：

* **群組 (Groups)**: 設定項被組織在以方括號 `[]` 包圍的群組名稱下，例如 `[KDE]` [^39]。
* **鍵值對 (Key-Value Pairs)**: 每個設定項由 `鍵=值` 的形式組成 [^11][^39]。
* **註解**: 以 `#` 開頭的行被視為註解，但在應用程式儲存設定時可能會被移除 [^39]。
* **特殊功能**: 格式支援透過 `[$i]` 鎖定設定以防止使用者修改、透過 `[$e]` 進行 shell 環境變數擴展，以及透過 `[zh_TW]` 等標籤實現本地化 [^39]。

以下是一個設定檔的簡單範例：

```ini
# 範例 KDE 設定檔
[KDE]
SingleClick=true

[General][$i]
ColorScheme=BreezeDark
```

## 設定轉移方法：手動與自動化

將 KDE Plasma 設定從一台電腦轉移到另一台，主要可分為手動操作和使用專用工具兩種途徑。

### 方法一：手動複製與封存

此方法給予使用者完全的控制權，但需要對檔案系統有基本的了解。

#### 目標式檔案備份

這是一種更精確的手動方法，僅備份與 KDE 相關的設定，可有效減小備份檔案的大小 [^62]。

1. **建立備份目錄**: 在家目錄中建立一個專門的資料夾來存放備份檔案。

    ```bash
    mkdir ~/plasma-backups
    ```

2. **複製核心設定**: 從 `~/.config` 目錄中複製關鍵的 KDE 設定檔和目錄到備份資料夾。

    ```bash
    cd ~/.config
    cp -R kdeconnect/ k* plasma* ~/plasma-backups/
    ```

    此指令會複製 `kdeconnect` 目錄，以及所有以 `k` 和 `plasma` 開頭的檔案和目錄 [^62]。使用者可依據 `ls` 指令的輸出，手動增加其他需要的設定檔（例如非 KDE 應用程式的設定）。

    {% alert(edit=true) %}
    `k*` 和 `kdeconnect/` 是重覆選取，正確指令應為：

```bash
cp -R k* plasma* ~/plasma-backups/
```

    {% end %}

3. **封存備份**: 將整個備份目錄壓縮成一個 tar 存檔，方便傳輸。

    ```bash
    cd ~/
    tar -jcvf plasma-backups.tar.bz2 plasma-backups/
    ```

4. **還原程序**: 在新電腦上，將 `plasma-backups.tar.bz2` 檔案解壓縮到 `~/.config` 目錄中。

    ```bash
    # 假設備份檔位於家目錄
    mv plasma-backups.tar.bz2 ~/.config/
    cd ~/.config/
    tar -xvf plasma-backups.tar.bz2
    # 將解壓縮出來的檔案移動到上一層目錄
    mv plasma-backups/* .
    # 清理空目錄和備份檔
    rmdir plasma-backups
    rm plasma-backups.tar.bz2
    ```

    完成後，登出目前的 Plasma 工作階段再重新登入，新的設定便會生效 [^62]。

### 方法二：使用專用工具

{% alert(edit=true) %}
我個人使用了手動方法，以下工具未經測試。
{% end %}

為了簡化遷移過程並降低出錯風險，社群開發了多款專用工具 [^2][^7][^13]。

以下是幾款常用工具的比較：

| 工具 (Tool) | 類型 (Type) | 描述 (Description) | 參考資料 (Sources) |
| :--- | :--- | :--- | :--- |
| **Konsave** | 命令列工具 (CLI Tool) | 允許使用者儲存、匯出、匯入和套用多個 KDE 設定檔 (profiles)。功能強大，適合進階使用者。 | [^2][^16][^41][^55] |
| **Plasma Config Saver** | Plasma 小工具 (Widget) | 提供圖形化介面，直接從桌面匯出和匯入外觀與風格設定，操作直觀。 | [^2][^18][^42][^45] |
| **Transfuse** | 指令稿 (Script) | 一個指令稿工具，用於備份和還原桌面設定。 | [^7] |
| **Deja Dup** | 圖形化備份工具 (GUI) | 通用備份軟體，可設定自動排程備份 `~/.config` 目錄，並支援排除特定資料夾。 | [^19][^58][^62] |
| **KDE Migrant** | 專用工具 (Specialized Tool) | 專為將現有 KDE 設定遷移到新電腦而設計的工具。 | [^13] |

#### Konsave 使用範例

Konsave 是其中一個功能最全面的工具，其基本操作如下：

1. **儲存目前設定**: 將目前的桌面設定儲存為一個名為 `main-desktop` 的設定檔。

    ```bash
    konsave -s main-desktop
    ```

2. **匯出設定檔**: 將設定檔匯出成一個單一檔案，以便傳輸。

    ```bash
    konsave -e main-desktop
    ```

    這會建立一個 `main-desktop.knsv` 檔案。
3. **匯入設定檔**: 在新電腦上，匯入該檔案。

    ```bash
    konsave -i main-desktop.knsv
    ```

4. **套用設定**: 套用已匯入的設定檔。

    ```bash
    konsave -a main-desktop
    ```

    套用後同樣需要重新登入 Plasma 工作階段。

## 遷移後的考量與疑難排解

成功遷移檔案後，仍可能遇到一些問題，特別是在硬體環境不同的情況下。

### 顯示器設定問題

這是最常見的問題之一，因為顯示器設定與特定的硬體（顯示卡、螢幕）緊密相關 [^8]。如果遷移後遇到黑畫面、解析度錯誤或無法登入圖形介面的問題，很可能是舊的顯示器設定與新硬體不相容。

* **解決方案**: 刪除或重新命名儲存顯示器設定的檔案，讓 Plasma 在下次登入時重新偵測硬體並產生預設設定。
  * 在 Plasma 6 或更新版本中，該檔案是 `~/.config/kwinoutputconfig.json` [^8]。
* **緊急修復步驟**:
    1. 使用 `Ctrl+Alt+F3` (或 F1-F6) 切換到一個虛擬終端 (TTY) 並登入。
    2. 執行指令刪除設定檔：`rm ~/.config/kwinoutputconfig.json`。
    3. 停止並重新啟動您的顯示管理器（例如 `sudo systemctl restart sddm`）。
    4. 使用 `Ctrl+Alt+F7` (或 F1) 切換回圖形介面，此時應能正常登入 [^8]。

### 系統層級與新使用者預設值

對於需要在多台機器上部署相同環境的系統管理員，可以修改系統層級的預設設定。Linux 系統使用 `/etc/skel` 目錄作為建立新使用者帳號時的範本 [^9][^38][^48]。

* **操作流程**:
    1. 建立一個新的測試使用者。
    2. 登入該使用者，並將 KDE Plasma 環境設定成您想要的預設樣式。
    3. 登出後，以 root 權限將該測試使用者家目錄中的 `~/.config` 和 `~/.local/share` 內的相關設定檔複製到 `/etc/skel/` 對應的位置。
    4. 之後，系統上任何新建立的使用者都將自動獲得這套預設設定 [^38]。

### 權限問題

當手動將檔案從一個使用者家目錄複製到另一個時，檔案的擁有者和群組可能不正確。這會導致應用程式無法讀取或寫入其設定檔。

* **解決方案**: 在新電腦上完成檔案複製後，執行 `chown` 指令來遞迴地修正整個家目錄的權限。

    ```bash
    sudo chown -R newuser:newgroup /home/newuser
    ```

    其中 `newuser` 和 `newgroup` 應替換為新系統上的使用者名稱和群組名稱。

[^2]: [Is there a way to transfer KDE Plasma settings from one laptop ...](https://www.reddit.com/r/kde/comments/13ori4n/is_there_a_way_to_transfer_kde_plasma_settings/)
[^3]: [KDE Plasma: a complete setup guide | by Rphlm - Medium](https://medium.com/tech-notes-and-geek-stuff/kde-plasma-a-complete-setup-guide-8cb253d081a5)
[^7]: [TRANSFUSE A Script to Backup and Restore KDE Plasma ...](https://forum.garudalinux.org/t/transfuse-a-script-to-backup-and-restore-kde-plasma-desktop-configurations/43384)
[^8]: [How to reset KDE / display settings after a move to a new ...](https://askubuntu.com/questions/299241/how-to-reset-kde-display-settings-after-a-move-to-a-new-machine)
[^9]: [How to duplicate KDE Desktop/Application settings from PC to ...](https://forum.manjaro.org/t/how-to-duplicate-kde-desktop-application-settings-from-pc-to-laptop/101377)
[^11]: [KDE System Administration/Configuration Files](https://userbase.kde.org/KDE_System_Administration/Configuration_Files)
[^12]: [Can you copy KDE configs to a new PC - Reddit](https://www.reddit.com/r/kde/comments/14q1yjb/can_you_copy_kde_configs_to_a_new_pc/)
[^13]: [migrate existing KDE/Plasma/apps configuration to a new ...](https://www.linuxlinks.com/kde-migrant-migrate-existing-kde-plasma-apps-configuration/)
[^14]: [There should be a way to collect config files in home](https://discuss.kde.org/t/there-should-be-a-way-to-collect-config-files-in-home/10296)
[^15]: [How do I migrate my KDE settings? - Help](https://discuss.kde.org/t/how-do-i-migrate-my-kde-settings/3074)
[^16]: [How to copy settings from a user to another - openSUSE Forums](https://forums.opensuse.org/t/how-to-copy-settings-from-a-user-to-another/176697)
[^17]: [Where are default Plasma 5 settings stored?](https://unix.stackexchange.com/questions/209317/where-are-default-plasma-5-settings-stored)
[^18]: [Moving KDE settings to new system - Manjaro Linux Forum](https://forum.manjaro.org/t/moving-kde-settings-to-new-system/113724)
[^19]: [How To Backup KDE Plasma 5 Desktop Settings On Linux](https://www.addictivetips.com/ubuntu-linux-tips/backup-kde-plasma-5-desktop-linux/)
[^23]: [KDE System Settings Config Files Locations - Reddit](https://www.reddit.com/r/kde/comments/fjl7re/kde_system_settings_config_files_locations/)
[^24]: [Where are default Plasma 5 settings stored?](https://unix.stackexchange.com/questions/209317/where-are-default-plasma-5-settings-stored)
[^25]: [Configuration notes and bloatometry of KDE Plasma 5 - FlaterCo](https://flaterco.com/kb/KDE.html)
[^30]: [locateion of kde configureation files - openSUSE Forums](https://forums.opensuse.org/t/locateion-of-kde-configureation-files/143864)
[^32]: [shalva97/kde-configuration-files - GitHub](https://github.com/shalva97/kde-configuration-files)
[^34]: [Where does Kubuntu save its configurations - Ask Ubuntu](https://askubuntu.com/questions/1524994/where-does-kubuntu-save-its-configurations)
[^35]: [Panel config file location - Help - KDE Discuss](https://discuss.kde.org/t/panel-config-file-location/30764)
[^38]: [configuration - Where are default Plasma 5 settings stored? - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/209317/where-are-default-plasma-5-settings-stored)
[^39]: [KDE System Administration/Configuration Files - KDE UserBase Wiki](https://userbase.kde.org/KDE_System_Administration/Configuration_Files)
[^41]: [Is there a way to transfer KDE Plasma settings from one laptop ...](https://www.reddit.com/r/kde/comments/13ori4n/is_there_a_way_to_transfer_kde_plasma_settings/)
[^42]: [What's the best way to move all my KDE config files to new ...](https://www.reddit.com/r/kde/comments/yvpuvs/whats_the_best_way_to_move_all_my_kde_config/)
[^45]: [Moving KDE settings to new system - Manjaro Linux Forum](https://forum.manjaro.org/t/moving-kde-settings-to-new-system/113724)
[^47]: [There should be a way to collect config files in home](https://discuss.kde.org/t/there-should-be-a-way-to-collect-config-files-in-home/10296)
[^48]: [How to duplicate KDE Desktop/Application settings from PC to ...](https://forum.manjaro.org/t/how-to-duplicate-kde-desktop-application-settings-from-pc-to-laptop/101377)
[^50]: [Configuration notes and bloatometry of KDE Plasma 5 - FlaterCo](https://flaterco.com/kb/KDE.html)
[^51]: [How do I migrate my KDE settings? - Help](https://discuss.kde.org/t/how-do-i-migrate-my-kde-settings/3074)
[^55]: [How to copy settings from a user to another - openSUSE Forums](https://forums.opensuse.org/t/how-to-copy-settings-from-a-user-to-another/176697)
[^56]: [Where are default Plasma 5 settings stored?](https://unix.stackexchange.com/questions/209317/where-are-default-plasma-5-settings-stored)
[^58]: [How To Backup KDE Plasma 5 Desktop Settings On Linux](https://www.addictivetips.com/ubuntu-linux-tips/backup-kde-plasma-5-desktop-linux/)
[^62]: [How To Backup KDE Plasma 5 Desktop Settings On Linux](https://www.addictivetips.com/ubuntu-linux-tips/backup-kde-plasma-5-desktop-linux/)
