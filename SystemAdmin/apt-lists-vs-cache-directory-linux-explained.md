+++
title = "深入解析 APT 兩大目錄：/var/lib/apt/lists 與 /var/cache/apt"
description = "你知道 /var/lib/apt/lists 和 /var/cache/apt 各自儲存什麼嗎？本文帶你了解這兩個目錄的功能與結構。掌握它們，讓你的 Linux 軟體管理更有效率！"
date = "2025-05-01T00:00:00Z"
updated = "2025-07-27T15:30:40Z"
draft = false

[taxonomies]
tags = [ "Linux" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/iqQLfxe2VLU9cpdhCAD939?invite=dOLYGeJyZJqVX>"
+++
{% editor_note() %}
在寫 Containerfile 時常要 mount 或 clear cache，但以前都不曉得這兩個資料夾具體存些什麼 🫠
{% end %}

{% chat(speaker="jim") %}
explain /var/lib/apt/lists and /var/cache/apt , which directory store what
{% end %}

{% chat(speaker="felo") %}
在 Debian 和 Ubuntu 系統中，`/var/lib/apt/lists` 和 `/var/cache/apt` 是兩個重要的目錄，分別用於存儲不同類型的資料。
{% end %}

## **/var/lib/apt/lists**

**功能**：這個目錄主要用來存儲系統中所有可用軟體包的清單資訊。當你執行 `apt update` 命令時，APT 會從配置的軟體源下載最新的包清單，並將這些資訊存儲在此目錄中。這些清單文件包含了每個包的版本、依賴關係和其他元數據，這使得系統能夠知道哪些包是可用的以及如何安裝它們[^1][^5][^8]。

**結構**：該目錄下的文件通常以來源的名稱命名，並包含該來源的包資訊。這些文件的大小可能會隨著可用包的數量而變化，因此有時會需要清理這些文件以釋放磁碟空間。

## **/var/cache/apt**

**功能**：這個目錄主要用來存儲已下載的 `.deb` 包文件。當你安裝或更新軟體包時，APT 會將這些包文件下載到 `/var/cache/apt/archives` 目錄中。這樣做的好處是，如果你需要重新安裝某個包，系統可以直接從這個緩存中提取，而不需要再次從網路下載[^2][^3][^11]。

**結構**：在 `/var/cache/apt/archives` 中，你會找到所有已下載的包文件，這些文件可以在未來的安裝或升級中重複使用。該目錄還包含一個 `partial` 子目錄，用於存儲在下載過程中未完成的包文件。

## **總結**

- **/var/lib/apt/lists**：存儲可用包的清單資訊，主要用於管理和更新包的元數據。
- **/var/cache/apt**：存儲已下載的包文件，方便未來的安裝和升級。

這兩個目錄在包管理過程中扮演著關鍵角色，確保系統能夠有效地管理和安裝軟體包。

[^1]: [[Solved] Unable to parse package file /var/lib/apt/lists - It's FOSS](https://itsfoss.com/unable-to-parse-package-file/)
[^2]: [apt - change location of the "lists" and "archives" folders](https://unix.stackexchange.com/questions/160196/change-location-of-the-lists-and-archives-folders)
[^3]: [Can I delete /var/cache/apt/archives for Ubuntu/Debian Linux?](https://www.cyberciti.biz/faq/can-i-delete-var-cache-apt-archives-for-ubuntu-debian-linux/)
[^5]: [what is the purpose of the files in this directory: /var/lib/apt/lists?](https://serverfault.com/questions/449726/what-is-the-purpose-of-the-files-in-this-directory-var-lib-apt-lists)
[^8]: [package management - /var/lib/apt/lists is huge - Ask Ubuntu](https://askubuntu.com/questions/179955/var-lib-apt-lists-is-huge)
[^11]: [Why does 'apt' do not store downloaded packages anymore?](https://superuser.com/questions/1405001/why-does-apt-do-not-store-downloaded-packages-anymore)
