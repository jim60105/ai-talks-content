+++
title = "如何在 Fedora Atomic Desktop 中保存 rpm-ostree 系統狀態"
description = "學習如何在進行系統更改前保存 Fedora Atomic Desktop 的 rpm-ostree 狀態，包括使用 pin 指令固定部署版本和回復機制，確保系統更新的安全性。"
date = "2024-12-14T00:47:22Z"
updated = "2025-07-25T13:59:37.656Z"
draft = false

[taxonomies]
tags = [ "Linux", "RHEL/Fedora", "System Admin", "Package Manager" ]
providers = [ "Felo Search" ]

[extra]
withAI = "<https://felo.ai/search/iNLVpPRCiPvMxQobkJtpMT?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
I'm using fedora atomic desktop. What should I do so I can save current rpm-ostree status before processing some changes?
{% end %}

{% chat(speaker="felo") %}
根據您的問題，我將說明如何在 Fedora Atomic Desktop 上保存當前的 rpm-ostree 狀態。
{% end %}

## 查看當前系統狀態

首先，您可以使用以下指令查看目前系統的部署狀態：

```bash
rpm-ostree status
```

這個指令會顯示當前系統的部署狀態，包括目前使用中的部署版本和待處理的更新[^2]。

## 保存系統狀態的方法

在 Fedora Atomic Desktop 系統中，rpm-ostree 預設會保留兩個可開機的部署版本[^6]。在進行任何更改之前，您可以：

1. **使用 pin 指令固定當前部署**：

    ```bash
    sudo ostree admin pin 0
    ```

    這裡的數字 0 代表在 `rpm-ostree status` 輸出中從零開始計數的部署編號[^10]。

2. **系統更新安全性**：

    - 如果更改出現問題，您可以通過開機選單選擇之前的部署版本來回復系統[^6]
    - 系統更新是離線進行的，所有變更都是以交易式方式完成，確保系統安全性[^5]

    {% alert(edit=true) %}
    我永遠記不住 pin 前面的 admin🫠
    {% end %}

## 注意事項

- 在進行重要更改之前，建議先確認目前系統狀態是穩定的
- 系統會在開機選單中保留先前的部署版本，方便您在需要時回復[^6]
- 任何更改都需要重新開機才能生效[^6]
- 如果您要進行系統更新，可以使用 `--preview` 選項預覽變更內容：

```bash
sudo rpm-ostree upgrade --preview
```

這些機制確保了您可以安全地進行系統更改，同時保留回復到之前狀態的能力。

[^2]: [Basic use - Mirantis Kubernetes Engine](https://docs.mirantis.com/mke/3.7/mkex/rpm-ostree-operation/basic-use.html)
[^5]: [Client administration | rpm-ostree](https://coreos.github.io/rpm-ostree/administrator-handbook/)
[^6]: [Updates and Rollbacks :: Fedora Docs](https://docs.fedoraproject.org/en-US/iot/applying-updates-UG/)
[^10]: [To Atomic or not to Atomic? : r/Fedora](https://www.reddit.com/r/Fedora/comments/1dadbsk/to_atomic_or_not_to_atomic/)
