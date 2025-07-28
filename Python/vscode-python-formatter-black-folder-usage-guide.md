+++
title = "Python 格式化工具選擇與 Black 指令教學"
description = "想知道推薦的 Python 格式化工具嗎？本篇詳解如何設定 VSCode 預設 Formatter，並教你用 Black 指令格式化特定資料夾。讓你的程式碼風格一致又專業！"
date = "2025-06-01T00:00:00Z"
updated = "2025-07-28T11:53:02Z"
draft = false

[taxonomies]
tags = [ "Python" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/Hpqxf6EWcDurxoakGUSibY?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
what is the default or 推薦的 python formatter in VSCode?
{% end %}

{% chat(speaker="felo") %}
當您在 VS Code 中編輯 Python 檔案時，格式化工具（Formatter）並不再內建於 Python Extension，而是透過外部擴充功能提供。以下說明預設與常見推薦的 Formatter，以及如何進行設定。
{% end %}

## 格式化工具簡介

**預設 Formatter**  

- 安裝 Microsoft 官方的 Python Extension 後，VS Code 預設即安裝並使用 autopep8 作為 Python 檔案的格式化工具autopep8 可自動修正大多數符合 PEP 8 的程式風格問題[^3]。

**推薦的第三方 Formatter**  

- Black  
  - 「不妥協的」Python 代碼格式化工具，能將程式碼統一為固定風格且幾乎免參數配置，社群採用度極高，與 PEP 8 相容。  
- YAPF  
  - Google 開源的 Python 格式化工具，提供高度可自訂的排版風格選項。  
- Ruff  
  - 主要為超快的 linter，但也支援自動修復與 import 排序，可輔助格式化與程式碼健檢[^2]。  

## 如何設定預設 Formatter

1. 開啟任一 `.py` 檔案。  
2. 於編輯區按右鍵，選擇 **Format Document With…**。  
3. 再選 **Configure Default Formatter…**。  
4. 從清單中挑選您要的格式化擴充，例如 `ms-python.black-formatter` 或 `ms-python.autopep8`[^7]。

設定完成後，即可透過快捷鍵（Windows/Linux：`Shift+Alt+F`；macOS：`Option+Shift+F`）或儲存檔案時自動格式化（於設定中開啟 `editor.formatOnSave`）來統一程式碼風格。

{% editor_note() %}
我後來在 [jim60105/standalone-civitai-shortcut](https://github.com/jim60105/standalone-civitai-shortcut) 中 採用 black
{% end %}

{% chat(speaker="jim") %}
how to run python black formatter in command line for specific folder only?
{% end %}

{% chat(speaker="felo") %}
在命令列針對特定資料夾執行 Black 格式化，只需要在指令後面接上該資料夾的路徑即可。以下示範最常用的做法以及進階參數。
{% end %}

## **前置作業**

- 確認已安裝 Black  

  ```bash
  pip install black
  ```

## **針對特定資料夾格式化**

- 基本指令  

  ```bash
  black path/to/your_folder
  ```  

  這個指令會遞迴掃描 `your_folder` 底下所有 `.py` 檔案，並直接套用 Black 的預設排版規則[^1001]。

## **常用進階選項**

- `-l`, `--line-length`：設定單行最長字元數（預設 88）  

  ```bash
  black path/to/your_folder --line-length 100
  ```  

- `--check`：僅檢查哪些檔案需要被格式化，但不會實際修改  

  ```bash
  black path/to/your_folder --check
  ```  

- `--diff`：顯示格式化前後差異，但不改寫檔案  

  ```bash
  black path/to/your_folder --diff
  ```

## **範例**

假設專案結構如下：  

```
my_project/
├── src/
│   ├── module1.py
│   └── module2.py
└── tests/
    └── test_module.py
```

要只格式化 `src` 資料夾，可執行：  

```bash
black my_project/src
```  

執行後，Black 會在終端顯示被修改或檢查通過的檔案清單。若要一次格式化整個專案，則改為指定 `my_project/` 即可[^1001]。

[^2]: [VSCode Python项目使用Ruff格式化代码配置指南_文心快码](https://comate.baidu.com/zh/page/Jg2H53dpbN)
[^3]: [How to make code formatting work again for Python (Visual ...](https://stackoverflow.com/questions/78778227/how-to-make-code-formatting-work-again-for-python-visual-studio-code-on-mac)
[^7]: [Formatting Python in VS Code](https://code.visualstudio.com/docs/python/formatting)
[^1001]: [Format your Python Code using Black | by Achanandhi M](https://medium.com/@achanandhi.m/format-your-python-code-using-black-fb387aae07b3)
