+++
title = "Rust 複製檔案到 CIFS 共享遇到權限錯誤的真正原因"
description = "即使關閉 SELinux，Rust 程式仍在 CIFS 掛載目錄遇到 Operation not permitted 錯誤。本文深入解析 `std::fs::copy` 在網路檔案系統上的限制，並提供安全可靠的解決方法。想知道如何讓檔案順利寫入 CIFS 嗎？"
date = "2025-06-01T00:00:00Z"
updated = "2025-07-28T11:39:31Z"
draft = false

[taxonomies]
tags = [ "Rust", "Linux" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/SqeGk4iPRXT4UUo943tGug?invite=dOLYGeJyZJqVX>"
+++
{% editor_note() %}
TL;DR: 最後我由 `std::fs::copy` 改用 `std::io::copy` 就搞定了。
{% end %}

{% chat(speaker="jim") %}
我有一個 root:root 的資料夾，其權限為 `drwxrwxrwx@ security.selinux: "system_u:object_r:cifs_t:s0"`, 我想要使用(我寫的)一支 Rust 程式往它底下寫入新檔案時遇到 `File operation error: Operation not permitted (os error 1)`。請你解釋錯誤的成因及解決方法。
{% end %}

{% chat(speaker="felo") %}
好的，這份報告將為您解析 Rust 程式在寫入具有特定 SELinux Context 的目錄時，發生 "Operation not permitted" 錯誤的根本原因，並提供一系列解決方案。
{% end %}

## 總覽

您遇到的 `Operation not permitted (os error 1)` 錯誤，其根本原因並非傳統的 Linux 檔案權限（Discretionary Access Control, DAC），而是由 **SELinux (Security-Enhanced Linux)** 的強制存取控制（Mandatory Access Control, MAC）策略所引發。

儘管目錄權限設定為 `drwxrwxrwx`，允許任何使用者寫入，但其 SELinux Context `system_u:object_r:cifs_t:s0` 限制了存取。`cifs_t` 類型通常用於 CIFS/Samba 網路共享目錄，而您的 Rust 程式是以一個 SELinux 策略中不被允許寫入 `cifs_t` 類型的程序 Context 運行的。因此，SELinux 核心模組攔截並拒絕了此操作。

解決此問題的核心在於調整 SELinux 的設定，使其允許您的程式進行存取，主要方法包括修正目錄的 SELinux Context、調整 SELinux 布林值，或在必要時建立自訂策略。

## 詳細報告

### 錯誤成因分析

#### 1. 權限檢視：超越傳統的 rwx

傳統的 Linux 權限（`owner:group:other`）是自主存取控制（DAC）的一部分。在您的案例中，`drwxrwxrwx` 權限確實賦予了系統上所有使用者對該目錄的讀、寫、執行權限[^30]。然而，權限末尾的 `@` 符號表示該檔案或目錄具有擴充屬性（extended attributes），這正是 SELinux Context 的儲存之處。

#### 2. SELinux 的角色：強制存取控制 (MAC)

SELinux 是一個在 Linux 核心中實作的 MAC 安全子系統，它在傳統 DAC 權限檢查之後，會再進行一次權限檢查[^70]。系統上的每一個程序（Subject）和每一個物件（Object，如檔案、目錄、通訊埠）都被賦予一個稱為「安全上下文 (Security Context)」的標籤[^4]。

安全上下文的格式為 `user:role:type:level`[^4]。當一個程序嘗試存取一個物件時，SELinux 會檢查其載入的策略資料庫，判斷是否有明確的規則允許「來源程序類型 (source type)」對「目標物件類型 (target type)」執行該操作（如 `write`, `create`）[^70]。如果沒有明確的 `allow` 規則，操作將被預設拒絕，無論傳統權限為何[^1]。

#### 3. 問題核心：不匹配的 SELinux Context

* **目標目錄 Context**: `system_u:object_r:cifs_t:s0`
  * `type` 欄位為 `cifs_t`。這個類型通常被 SELinux 策略指定給 CIFS 或 Samba 服務所共享的檔案系統，用於規範網路檔案存取[^18]。
* **來源程序 Context**: 您的 Rust 程式
  * 當您以一般使用者身份執行程式時，其程序通常會被賦予如 `unconfined_u:unconfined_r:unconfined_t` 之類的 Context[^16]。`unconfined_t` 雖然受到的限制較少，但其權限並非無限，它仍然受到 SELinux 策略的約束，例如，預設情況下它可能不被允許寫入專門用途的類型如 `cifs_t`。

錯誤的發生，正是因為 SELinux 預設策略中，沒有一條規則允許一個 `unconfined_t` 的程序在一個 `cifs_t` 類型的目錄中建立檔案。這是一個安全設計，旨在將不同服務的資料隔離開來，防止一個受損的程序（例如網頁伺服器）去竄改其他服務（例如資料庫或網路共享）的檔案[^60]。

#### 4. 其他可能性：不可變屬性 (Immutable Attribute)

雖然可能性較低，但 `Operation not permitted` 錯誤也可能由檔案系統的 `immutable` 屬性造成。此屬性設定後，即使是 root 使用者也無法修改、刪除或更名該檔案/目錄[^60]。您可以使用 `lsattr` 指令來檢查：

```bash
lsattr -d /path/to/your/directory
```

如果輸出中包含 `i` 屬性（例如 `----i-------e--`），則表示該目錄被設為不可變。

### 解決方案

解決此問題應從調整 SELinux 組態著手，以下方案按推薦順序排列。

#### 方案一：修改目錄的 SELinux Context (最推薦)

如果該目錄的用途並非 CIFS 網路共享，那麼最正確的作法是修正其標籤，使其符合實際用途。這是處理 SELinux 標籤錯誤最常見且標準的方法[^2][^3][^18]。

1. **決定正確的 Context Type**：
    * 如果該目錄用於網頁伺服器 (如 Apache, Nginx) 讀寫內容，可使用 `httpd_sys_rw_content_t`。
    * 如果用於其他特定服務，應查找該服務對應的 SELinux 文件，找到建議的檔案 Context。
    * 如果只是一般用途的公共可寫目錄，`public_content_rw_t` 是一個可能的選項。

2. **永久性修改 Context**：使用 `semanage fcontext` 指令來新增一條規則，讓這個路徑未來自動套用正確的 Context。這能確保即使在檔案系統重新標記後，設定依然存在[^18]。

    ```bash
    # 範例：將目錄設定為可供 httpd 讀寫的類型
    # -a 表示新增, -t 表示類型
    sudo semanage fcontext -a -t httpd_sys_rw_content_t "/path/to/your/directory(/.*)?"
    ```

3. **立即套用變更**：使用 `restorecon` 指令讀取 `semanage` 的規則，並將其應用到實際的檔案系統上[^18][^58]。

    ```bash
    # -R 表示遞迴處理, -v 表示顯示詳細過程
    sudo restorecon -Rv /path/to/your/directory
    ```

#### 方案二：調整 SELinux 布林值

SELinux 策略包含許多可調整的「布林值 (Booleans)」，它們像是開關，可以允許或禁止某些特定的行為，而無需修改底層策略[^10]。

1. **查找相關布林值**：您可以搜尋與您的服務或目標 Context (`cifs_t`) 相關的布林值。

    ```bash
    # 列出所有布林值並過濾關鍵字
    getsebool -a | grep "cifs\|samba"
    ```

2. **啟用布林值**：如果找到相關的布林值（例如 `samba_export_all_rw`），您可以使用 `setsebool` 來啟用它。

    ```bash
    # -P 選項可以讓設定在重開機後依然生效
    sudo setsebool -P samba_export_all_rw on
    ```

#### 方案三：建立自訂 SELinux 策略模組 (進階)

當標準 Context 和布林值都無法滿足需求時，您可以建立一個本地的自訂策略模組來明確允許此操作。這通常是最後的手段[^2][^3]。

1. **切換至 Permissive 模式**：此模式下 SELinux 不會阻止任何操作，但會將所有違反策略的行為記錄下來[^6][^54]。

    ```bash
    sudo setenforce 0
    ```

2. **重現錯誤並收集日誌**：再次執行您的 Rust 程式。此時操作會成功，但 SELinux 會在 `/var/log/audit/audit.log` 中產生一條 `AVC denial` 記錄[^17]。

3. **產生策略模組**：使用 `audit2allow` 工具分析日誌並產生策略。

    ```bash
    # 從 audit.log 中查找最近的 selinux 拒絕日誌
    sudo grep "denied" /var/log/audit/audit.log | audit2allow -M my_rust_app_policy
    ```

    這會產生 `my_rust_app_policy.te` (可讀的策略原始碼) 和 `my_rust_app_policy.pp` (已編譯的策略包)。

4. **安裝並載入模組**：

    ```bash
    sudo semodule -i my_rust_app_policy.pp
    ```

5. **切回 Enforcing 模式**：

    ```bash
    sudo setenforce 1
    ```

**警告**：請謹慎使用 `audit2allow`，它產生的規則可能過於寬鬆。務必先確認問題不是由簡單的檔案標籤錯誤所引起[^2][^3]。

#### 方案四：永久切換至 Permissive 模式 (不建議)

您可以編輯 `/etc/selinux/config` 檔案，將 `SELINUX` 的值改為 `permissive`。這會讓系統在開機時就進入寬容模式，所有違反策略的操作都只會被記錄而不會被阻擋。這會大幅降低系統的安全性，**不建議在生產環境中使用**[^1][^5][^6]。

### Rust 程式碼層面的考量

您在 Rust 中遇到的 `std::io::Error` 其 `kind` 為 `PermissionDenied`，並附帶 `os error 1`，這表示錯誤源自於作業系統核心層級[^39][^41]。Rust 的標準檔案系統函式庫 (`std::fs`) 只是忠實地將 OS 回傳的錯誤報告給您。

雖然 Rust 生態系中有 `selinux` crate 這樣的工具，可以讓程式碼與 SELinux API 互動（例如查詢當前模式、獲取檔案 Context 等）[^15][^61][^76]，但它主要用於開發需要感知 SELinux 環境的系統工具，無法用來「繞過」SELinux 的策略限制。您當前遇到的問題屬於系統組態層級，應從上述的系統管理方案著手解決。

{% chat(speaker="jim") %}
我按照這個步驟操作:

1. sudo setenforce 0
2. sudo grep "denied" /var/log/audit/audit.log
3. 執行我的程式，「仍然發生 File operation error: Operation not permitted (os error 1)」
4. sudo grep "denied" /var/log/audit/audit.log
5. 我發現 audit.log 中並沒有新增條目

在這個情況下，我判斷問題可能不是出在 SELinux Context，而是有可能是因為目標確實是掛載的 cifs 目錄。我觀察到有空檔案被建立，但是裡面沒有內容。我使用以下 Rust 程式碼複製檔案:

```rust
std::fs::copy(&op.subtitle_file.path, &final_target)?;
```

這個程式在本地資料夾執行時是沒有問題的。
請給我建議如何排除問題。
{% end %}

{% chat(speaker="felo") %}
好的，這份報告將深入分析您在使用 Rust 的 `std::fs::copy` 函式向 CIFS 掛載目錄寫入檔案時，即使在 SELinux Permissive 模式下仍遭遇 `Operation not permitted` 錯誤的原因，並提供具體的解決方案。
{% end %}

## 總覽

您遇到的問題根源並非 SELinux，而是 **Rust 標準函式庫 `std::fs::copy` 的實作方式與 CIFS 網路檔案系統的權限模型之間存在不相容性**。

`std::fs::copy` 是一個高階函式，它不僅僅是複製檔案內容，還會嘗試複製來源檔案的元資料（Metadata），特別是 POSIX 形式的檔案權限（例如 `rwx` 權限位元）。然而，CIFS/SMB 是一個源自 Windows 的協定，其權限模型與 Linux/Unix 的 POSIX 模型截然不同[^1019][^1107]。

當您的 Rust 程式呼叫 `std::fs::copy` 時，它首先成功在 CIFS 掛載點上建立了一個空檔案，但隨後在試圖設定該新檔案的 POSIX 權限時失敗。CIFS 伺服器或 Linux 核心的 CIFS 模組無法正確處理這個權限設定請求，因此回報了 `Operation not permitted (os error 1)` 的錯誤[^1134][^1136]。

最直接且推薦的解決方案是修改 Rust 程式碼，改用 `std::io::copy` 進行底層的位元流複製，這樣可以完全繞過有問題的權限設定步驟。

## 詳細報告

### 錯誤成因分析

#### 1. `std::fs::copy` 的雙重任務：內容與元資料

與單純的檔案讀寫不同，`std::fs::copy` 的設計目標是盡可能完整地複製一個檔案，這包括兩個主要部分：

* **檔案內容**：將來源檔案的資料位元組（bytes）完整寫入目標檔案。
* **檔案元資料**：嘗試將來源檔案的權限（permissions）複製到目標檔案。在 Linux 系統上，這通常是透過 `fchmod` 系統呼叫來設定檔案的模式（mode）[^1107]。

這個行為在本地檔案系統（如 ext4, XFS）上運作良好，但在處理網路檔案系統時，特別是那些非原生支援 POSIX 標準的系統，就會產生問題。

#### 2. CIFS 檔案系統的權限模型差異

CIFS（Common Internet File System），即 SMB 協定的早期版本，是微軟為「網路上的芳鄰」設計的檔案共享協定[^1102][^1173]。它的核心是基於 Windows 的存取控制清單（ACLs），而非 Unix-like 系統的 `owner/group/other` 和 `rwx` 權限位元[^1019][^1101]。

當 Linux 客戶端掛載一個 CIFS 共享時，Linux 核心的 CIFS 模組 (`cifs.ko`) 會在一個稱為虛擬檔案系統（VFS）的抽象層中，盡力去「模擬」或「轉譯」POSIX 權限[^1112]。然而，這種模擬並非完美，其行為高度依賴於：

* **伺服器支援**：遠端伺服器（Windows 或 Samba）是否支援「CIFS Unix Extensions」。若不支援（例如使用 `nounix` 選項掛載），權限處理會變得非常簡化[^1101][^1194]。
* **掛載選項**：客戶端使用的掛載選項，如 `file_mode`, `dir_mode`, `uid`, `gid` 等，會強制設定客戶端所看到的虛擬權限[^1003][^1192]。

您遇到的問題正是發生在這個轉譯層。`std::fs::copy` 試圖設定一個 POSIX 權限，但 CIFS 檔案系統無法理解或不允許此操作，最終導致核心回傳 `EPERM` (Operation not permitted) 錯誤。

#### 3. 為何會產生空檔案？

錯誤的發生順序可以解釋為何您會看到一個空的目標檔案：

1. **檔案建立**：Rust 程式呼叫 `open` 系統呼叫並帶有 `O_CREAT` 旗標，這一步在 CIFS 共享上成功了，因此一個 0 位元組的空檔案被建立。
2. **內容複製**：在開始複製內容之前或之後（取決於具體實作），`std::fs::copy` 會嘗試進行元資料操作。
3. **權限設定失敗**：`std::fs::copy` 嘗試呼叫 `fchmod` 或類似的系統呼叫來設定新檔案的權限。此操作被 CIFS 檔案系統拒絕，核心回傳錯誤。
4. **程式中止**：`std::fs::copy` 函式接收到這個致命錯誤後，便會中止執行並將錯誤回傳給您的程式，此時檔案內容尚未被寫入。

在其他開發者的類似報告中，使用 `strace` 工具追蹤系統呼叫，也證實了錯誤發生在檔案建立後的一個元資料相關操作上，例如 `statx` 或 `fchmod`[^1136]。

### 解決方案

#### 方案一：修改 Rust 程式碼，使用 `std::io::copy` (最推薦)

這是最根本且可靠的解決方案，因為它直接避開了問題的根源。`std::io::copy` 是一個較低階的函式，它只負責將一個可讀取來源（Reader）的位元組流忠實地傳輸到一個可寫入目標（Writer），完全不涉及檔案權限等元資料操作[^1132][^1136]。

您可以將 `std::fs::copy(...)` 替換為以下邏輯：

```rust
use std::fs::File;
use std::io;
use std::path::Path;

/// 一個能可靠地將檔案複製到 CIFS 掛載點的函式
fn copy_file_to_cifs(source: &Path, destination: &Path) -> io::Result<u64> {
    // 1. 開啟來源檔案進行讀取
    let mut source_file = File::open(source)?;
    
    // 2. 建立目標檔案進行寫入
    //    此處只會建立檔案，不會試圖設定複雜的權限
    let mut dest_file = File::create(destination)?;
    
    // 3. 使用 io::copy 進行單純的位元組流複製
    io::copy(&mut source_file, &mut dest_file)
}

// 在您的程式中呼叫
// copy_file_to_cifs(&op.subtitle_file.path, &final_target)?;
```

這個方法確保了只有檔案內容被複製，而檔案的權限將由 CIFS 掛載時的選項（如 `file_mode`, `dir_mode`）或伺服器的預設值來決定，從而避免了不相容的權限設定操作[^1108]。

#### 方案二：調整 CIFS 掛載選項

如果修改程式碼不可行，您可以嘗試調整客戶端的 CIFS 掛載選項，使其行為更能容忍這類操作。這屬於系統層級的調整。

**在 `/etc/fstab` 或 `mount` 指令中加入 `noperm` 選項**

`noperm` 選項會告知 CIFS 客戶端不要在本地執行權限檢查[^1012]。這有時可以繞過 VFS 層在將請求發送到伺服器之前所做的權限判斷。

一個 `/etc/fstab` 的範例如下：

```fstab
# 在原有的選項基礎上加入 noperm
//192.168.1.100/share /mnt/cifs cifs credentials=/root/.smb,uid=1000,gid=1000,file_mode=0777,dir_mode=0777,noperm 0 0
```

**安全警告**：使用 `noperm` 會帶來一定的安全風險。它意味著掛載點上的檔案權限將完全由遠端伺服器控制，本地系統上的任何使用者只要能存取該掛載路徑，就可能繞過本地的權限設定讀寫檔案[^1012][^1153]。請謹慎評估此風險。

#### 方案三：檢查伺服器端設定

雖然問題主要出在客戶端函式庫與檔案系統的互動上，但確保伺服器端設定正確也是必要的除錯步驟。

* **Windows 共享權限**：在 Windows 伺服器上，請確保用於掛載的帳號在「共用」和「安全性」兩個索引標籤中都具有「完全控制」或至少「修改/寫入」的權限[^1008]。
* **Samba 伺服器設定**：如果伺服器是 Samba，請檢查其設定檔 (`/etc/samba/smb.conf`)，確保相關的分享區塊設定了 `writable = yes` 或 `read only = no`，並且掛載使用者被包含在 `valid users` 或 `write list` 中[^1005][^1047][^1141]。

總結來說，`std::fs::copy` 的行為雖然在標準檔案系統上是合理的，但在與 CIFS 這類非 POSIX 相容的系統互動時顯得不夠穩健。改用 `std::io::copy` 是最能確保程式穩定運行的作法。

[^1]: [4.4. SELinux 状态和模式中的永久性更改](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-security-enhanced_linux-working_with_selinux-changing_selinux_modes)
[^2]: [5.3. 修复分析的SELinux 拒绝问题](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/8/html/using_selinux/fixing-an-analyzed-selinux-denial_troubleshooting-problems-related-to-selinux)
[^3]: [5.3. 修复分析的SELinux 拒绝问题](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/9/html/using_selinux/fixing-an-analyzed-selinux-denial_troubleshooting-problems-related-to-selinux)
[^4]: [關於SELinux Policy. 最近工作上需要研究一下Red Hat 這款Linux](https://medium.com/%E4%B8%80%E5%80%8B%E5%B0%8F%E5%B0%8F%E5%B7%A5%E7%A8%8B%E5%B8%AB%E7%9A%84%E9%9A%A8%E6%89%8B%E7%AD%86%E8%A8%98/%E9%97%9C%E6%96%BC-selinux-policy-15466ef928b4)
[^5]: [Elastic Compute Service：開啟或關閉SELinux - Alibaba Cloud](https://www.alibabacloud.com/help/tc/ecs/use-cases/enable-or-disable-selinux)
[^6]: [解决docker创建的容器使用root用户仍然无法创建文件夹的问题](https://learnku.com/articles/82571)
[^10]: [selinux决策和日志 - i2SOM 西安湃兔核产品文档](https://wiki.i2som.com/docs/embeddeddevwiki/embeddeddevwiki-1gct5djldga21)
[^15]: [selinux - Rust - Docs.rs](https://docs.rs/selinux)
[^16]: [使用SELinux | Red Hat Enterprise Linux | 8](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/8/html-single/using_selinux/index)
[^17]: [Chapter 5. Troubleshooting problems related to SELinux](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/using_selinux/troubleshooting-problems-related-to-selinux_using-selinux)
[^18]: [11.2. 问题最多的三种原因| SELinux 用户和管理员指南](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-security-enhanced_linux-troubleshooting-top_three_causes_of_problems)
[^30]: [如何解决sudo 权限错误 - LabEx](https://labex.io/zh/tutorials/nmap-how-to-resolve-sudo-permission-errors-419594)
[^39]: [OS Error 5 when Writing to file on windows 10](https://users.rust-lang.org/t/os-error-5-when-writing-to-file-on-windows-10/49307)
[^41]: [Failed to load file: Os { code: 5, kind: PermissionDenied ...](https://users.rust-lang.org/t/failed-to-load-file-os-code-5-kind-permissiondenied-message-access-is-denied/95201)
[^54]: [Operation Not Permitted | the Little Projects of Shawn M. Jones](https://www.littleprojects.org/blog/2012/06/02/operation-not-permitted/)
[^58]: [restorecon failed: &#039;Operation not permitted&#039; - Red Hat Customer Portal](https://access.redhat.com/solutions/6068651)
[^60]: [Operation Not Permitted | the Little Projects of Shawn M. Jones](https://www.littleprojects.org/blog/2012/06/02/operation-not-permitted/)
[^61]: [selinux - Rust - Docs.rs](https://docs.rs/selinux)
[^70]: [What is SELinux? - Future Techno India - Medium](https://futuretechnoindia.medium.com/what-is-selinux-ed3ddc44d10a)
[^76]: [selinux - Rust](https://docs.rs/selinux)
[^1003]: [Mounted cifs share but no write permissions - Ask Ubuntu](https://askubuntu.com/questions/334422/mounted-cifs-share-but-no-write-permissions)
[^1005]: [第8 堂課- 網路芳鄰伺服器(SMB Server)](https://dic.vbird.tw/linux_server/unit08.php)
[^1008]: [無法從檔案總管存取共用資料夾- Windows Client](https://learn.microsoft.com/zh-tw/troubleshoot/windows-client/networking/cannot-access-shared-folder-file-explorer)
[^1012]: [mount.cifs](http://www.cs.toronto.edu/~simon/samba3/manpages/mount.cifs.8.html)
[^1019]: [NFS vs CIFS - 檔案系統之間的區別 - AWS](https://aws.amazon.com/tw/compare/the-difference-between-nfs-and-cifs/)
[^1047]: [第十六章、檔案伺服器之二： SAMBA 伺服器 - 鳥哥私房菜](https://linux.vbird.org/linux_server/centos6/0370samba.php)
[^1101]: [www.kernel.org/doc/readme/Documentation-filesystems-cifs-README](https://www.kernel.org/doc/readme/Documentation-filesystems-cifs-README)
[^1102]: [鳥哥私房菜 - 第十六章、檔案伺服器之二： SAMBA 伺服器](https://linux.vbird.org/linux_server/centos6/0370samba.php)
[^1107]: [Weird behavior of std::fs::copy on network-mounted filesystem](https://www.reddit.com/r/rust/comments/57cmm1/weird_behavior_of_stdfscopy_on_networkmounted/)
[^1108]: [std::fs::copy 喺網絡掛載嘅檔案系統度嘅奇怪行為: r/rust - Reddit](https://www.reddit.com/r/rust/comments/57cmm1/weird_behavior_of_stdfscopy_on_networkmounted/?tl=zh-hant)
[^1112]: [存储系列之VFS虚拟文件系统简介- orange-C - 博客园](https://www.cnblogs.com/orange-CC/p/12720341.html)
[^1132]: [Io::copy works, but fs::copy fails - help - Rust Users Forum](https://users.rust-lang.org/t/io-copy-works-but-fs-copy-fails/44649)
[^1134]: [std::fs::copy failed with OS error 1 on Linux when copying over CIFS from local FS · Issue #66760 · rust-lang/rust · GitHub](https://github.com/rust-lang/rust/issues/66760)
[^1136]: [linux - Stale file handles when copying files to network share (SMB) with std::fs::copy - Stack Overflow](https://stackoverflow.com/questions/77233207/stale-file-handles-when-copying-files-to-network-share-smb-with-stdfscopy)
[^1141]: [Mounted cifs share but no write permissions - Ask Ubuntu](https://askubuntu.com/questions/334422/mounted-cifs-share-but-no-write-permissions)
[^1153]: [Mount cifs Network Drive: write permissions and chown](https://unix.stackexchange.com/questions/68079/mount-cifs-network-drive-write-permissions-and-chown)
[^1173]: [比較幾個網路檔案系統：CIFS、NFS、和SSHFS - 伊蜜澤瑞](https://ignatius1895.pixnet.net/blog/post/347861638)
[^1192]: [mount - Mounted cifs share but no write permissions - Ask Ubuntu](https://askubuntu.com/questions/334422/mounted-cifs-share-but-no-write-permissions)
[^1194]: [www.kernel.org/doc/readme/Documentation-filesystems-cifs-README](https://www.kernel.org/doc/readme/Documentation-filesystems-cifs-README)
