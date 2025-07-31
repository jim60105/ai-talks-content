+++
title = "RHEL 匯入並信任 SSL 憑證教學（含 .crt 檔操作步驟）"
description = "詳細教學如何在 Red Hat Enterprise Linux（RHEL）系統中匯入 .crt 憑證並設為信任，包括安裝 ca-certificates、複製憑證、更新信任存儲與驗證步驟，適用於 SSL/TLS 應用。"
date = "2025-01-15T07:57:11Z"
updated = "2025-07-25T17:09:41.270Z"
draft = false

[taxonomies]
tags = [ "RHEL/Fedora", "System Admin", "Linux" ]
providers = [ "Felo Search" ]


[extra]
withAI = "<https://felo.ai/search/CYiPmeSgAw3BJ6892jMJrJ?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
how to import and trust a SSL web certificate to rhel, I have crt file downloaded
{% end %}

{% chat(speaker="felo") %}
在 Red Hat Enterprise Linux (RHEL) 中，若您已下載了 `.crt` 憑證檔案並希望將其匯入並信任，以下是詳細的操作步驟：
{% end %}

---

## **步驟 1：確認必要的工具**

確保系統已安裝 `ca-certificates` 套件，該套件包含 Mozilla 基金會的預設憑證集合，並用於管理系統的信任憑證。

執行以下命令以安裝：

```bash
sudo yum install ca-certificates
```

---

## **步驟 2：將憑證檔案複製到信任目錄**

將 `.crt` 憑證檔案複製到系統的信任目錄 `/etc/pki/ca-trust/source/anchors/` 中：

```bash
sudo cp your-certificate.crt /etc/pki/ca-trust/source/anchors/
```

---

## **步驟 3：更新信任存儲**

執行以下命令以更新系統的信任存儲，讓新增的憑證生效：

```bash
sudo update-ca-trust
```

---

## **步驟 4：驗證憑證是否已被信任**

您可以使用以下命令檢查憑證是否已成功添加到信任列表中：

```bash
trust list --filter=ca-anchors | grep "Your Certificate Name"
```

或者，使用 `openssl` 測試連線是否成功：

```bash
openssl s_client -connect yourserver.com:443
```

檢查輸出中是否顯示憑證已被信任。

---

## **注意事項**

1. **憑證格式**：確保 `.crt` 檔案是 PEM 格式。如果是其他格式（例如 DER），可使用以下命令轉換：

   ```bash
   openssl x509 -inform der -in your-certificate.der -out your-certificate.crt
   ```

2. **自簽憑證**：若是自簽憑證，請確認其已正確生成，並包含完整的憑證鏈（例如根憑證和中繼憑證）。

3. **應用程式信任**：某些應用程式可能不使用系統的信任存儲（例如某些版本的 Chrome 或 Java），需要單獨配置其信任憑證。

---

透過上述步驟，您應能成功將 `.crt` 憑證匯入並信任於 RHEL 系統中，並確保相關應用程式能正常使用該憑證進行 SSL/TLS 通信。
