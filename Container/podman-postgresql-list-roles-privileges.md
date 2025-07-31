+++
title = "Podman 容器內查詢 PostgreSQL 所有帳號與權限設定教學"
description = "教你如何在 Podman 容器中，使用指令查詢 PostgreSQL 資料庫所有帳號（role）與權限設定，完整步驟與注意事項，適用於容器化資料庫管理與權限稽核需求。"
date = "2025-01-07T09:26:45Z"
updated = "2025-07-25T16:12:17.032Z"
draft = false

[taxonomies]
tags = [ "Container", "Database", "Podman" ]
providers = [ "Felo Search" ]


[extra]
withAI = "<https://felo.ai/search/aK2qfyrju6TSNXF8fjkUS2?invite=dOLYGeJyZJqVX>"
+++
{% alert(edit=true) %}
這篇的緣由是客戶說他們的資訊部門要求清查所有資料庫的 role，並且調整權限以合乎最小權限原則。

我們的案子是有一台 PostgreSQL，但它是在 Podman 中運行，僅開放同一 Pod 內服務存取。我出了這份查詢指令給客戶，並且努力說服他們不要去碰它的權限，

「建置時已經照最佳實踐用容器化技術做了網路隔離，要接資料庫只能連到主機上以 root 下指令」  
「指令在這裡給你們自己操作。」  
「那個 DB 是程式自動管理的，你們真的不該這樣做喔」  
「壞掉了我沒辦法喔！只能走災難復原喔！」

這樣。最後客戶沒去碰它。😀
{% end %}

{% chat(speaker="jim") %}
我有一台 postgresql，在 podman container 中，name: postgresql。給我指令如何 podman exec 進入 container 之中，列出所有的 database account 和權限設定。一步一步的詳細說明。
{% end %}

{% chat(speaker="felo") %}
以下是詳細的操作步驟，從進入容器到查詢 PostgreSQL 資料庫帳號及其權限設定：
{% end %}

## 使用 Podman 進入 PostgreSQL 容器並列出所有資料庫帳號及其權限設定的步驟

### **步驟一：進入 PostgreSQL 容器**

1. 確保容器名稱為 `postgresql` 並且正在運行：
   - 使用以下指令檢查容器狀態：

     ```bash
     podman ps
     ```

     確認容器名稱是否為 `postgresql` 並處於運行狀態。如果容器未啟動，可使用以下指令啟動：

     ```bash
     podman start postgresql
     ```

2. 使用 `podman exec` 進入容器的互動式終端：

   ```bash
   podman exec -it postgresql bash
   ```

   - `-it` 表示進入互動模式。
   - `bash` 是進入容器內的 Shell。

---

### **步驟二：進入 PostgreSQL 的命令列介面**

1. 在容器內，使用以下指令進入 PostgreSQL 的命令列介面：

   ```bash
   psql -U postgres
   ```

   - `-U postgres` 指定以 `postgres` 超級使用者身份登入。

2. 如果需要退出 PostgreSQL 命令列介面，可輸入：

   ```bash
   \q
   ```

---

### **步驟三：列出所有資料庫帳號**

1. 查詢所有角色（帳號）名稱：

   ```sql
   SELECT rolname FROM pg_roles;
   ```

   此指令會列出所有角色名稱，包括系統角色與使用者自定義角色。

---

### **步驟四：查詢角色的權限設定**

PostgreSQL 的權限分為多種類型，以下是常見的查詢方式：

1. **查詢角色對資料表的權限：**

   ```sql
   SELECT grantee, table_name, privilege_type
   FROM information_schema.role_table_grants
   WHERE grantee = '角色名稱'; -- 將 '角色名稱' 替換為具體角色名稱
   ```

2. **查詢角色對資料庫的權限：**

   ```sql
   SELECT grantee, database_name, privilege_type
   FROM information_schema.role_database_grants
   WHERE grantee = '角色名稱';
   ```

3. **查詢角色對 Schema 的權限：**

   ```sql
   SELECT grantee, schema_name, privilege_type
   FROM information_schema.role_schema_grants
   WHERE grantee = '角色名稱';
   ```

4. **查詢角色對函數或序列的權限：**
   - 函數權限：

     ```sql
     SELECT * FROM information_schema.routine_privileges WHERE grantee = '角色名稱';
     ```

   - 序列權限：

     ```sql
     SELECT * FROM information_schema.sequence_privileges WHERE grantee = '角色名稱';
     ```

5. **查詢角色的系統層級權限：**

   ```sql
   SELECT * FROM pg_roles WHERE rolname = '角色名稱';
   ```

---

### **步驟五：整合資訊（可選）**

如果需要將所有角色及其權限整合為一個報表，可以使用程式語言（如 Python）或 SQL 的 `UNION ALL` 語法，將上述查詢結果結合。

---

### **注意事項**

1. **權限要求：** 執行上述查詢需要使用具有足夠權限的角色（如 `postgres`）。
2. **資料庫版本：** 不同版本的 PostgreSQL 在系統目錄表結構上可能略有差異，請參考對應版本的官方文件。
3. **退出容器：** 完成操作後，可輸入以下指令退出容器：

   ```bash
   exit
   ```

---

透過以上步驟，您可以成功進入 Podman 容器中的 PostgreSQL，並查詢所有資料庫帳號及其權限設定。
