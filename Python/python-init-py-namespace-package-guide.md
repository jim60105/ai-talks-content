+++
title = "套件還是資料夾？Python __init__.py 與命名空間套件全解析"
description = "你知道 Python 專案裡的 __init__.py 檔案有什麼作用嗎？本篇帶你深入了解套件、命名空間套件與最佳實踐。想知道什麼時候該放、什麼時候能省略？三分鐘掌握 Python 專案結構設計關鍵！"
date = "2025-08-03T15:20:01.688Z"
updated = "2025-08-03T15:20:01.989Z"
draft = false

[taxonomies]
tags = [ "Python" ]
providers = [ "ChatGPT" ]

[extra]
withAI = "<https://chatgpt.com/share/688f7d37-ccbc-8005-a3d4-a68ff4c609e6>"
+++
{% chat(speaker="jim") %}
What's `__init__.py` file for?
{% end %}

{% chat(speaker="chatgpt") %}
在 Python 中，`__init__.py` 的主要用途是將一個目錄標記為「套件」（package），讓解譯器知道可以從裡面載入模組。以下重點說明它的功用：
{% end %}

1. **標記套件**  
   在 Python 2 和早期的 Python 3，只要該目錄下有 `__init__.py`，Python 就會把它當成一個套件處理，否則無法直接用 `import mypackage.module` 的語法載入。

2. **初始化套件**  
   你可以在 `__init__.py` 裡放一些執行時就要跑的程式碼，例如：

   ```python
   # mypackage/__init__.py
   print("載入 mypackage…")
   from .submodule import helper  # 方便使用者直接 import mypackage.helper
   __all__ = ["submodule", "utils"]  # 指定 from mypackage import * 時要暴露的子模組
   ```

   這樣使用者在 `import mypackage` 時，就會先執行這些初始化邏輯。

3. **統一暴露介面**  
   透過 `__all__`、或在 `__init__.py` 中重新導入（alias）子模組／物件，可以隱藏套件內部結構，只暴露核心 API。例如：

   ```python
   # mypackage/__init__.py
   from .core import Foo, Bar
   __all__ = ["Foo", "Bar"]
   ```

   使用者只要 `from mypackage import *`，就只會拿到 `Foo`、`Bar`。

4. **支援命名空間套件（Namespace Packages）**  
   自 Python 3.3 起，如果你想要跨多個目錄結合成同一個套件，就可以不放 `__init__.py`，稱為「命名空間套件」。它可以讓不同安裝來源（如不同套件）共用同個套件名稱。

   > **注意**：如果你需要在套件初始化時執行程式，或想要完全支援舊版 Python，就還是要放一個空檔案 `__init__.py`。

5. **最佳實踐**

   * **空檔案即可**：若只是想標記套件，檔案可以留空。
   * **輕量化**：避免在裡面放太多邏輯，複雜初始化最好移到專門的模組裡。
   * **清晰 API**：透過 `__all__` 管理暴露介面，讓 import 更直覺。

{% chat(speaker="chatgpt") %}
簡而言之，`__init__.py` 是 Python 套件的「門面」與「開關」，決定目錄能否被視為套件，並且可以在載入時執行必要初始化或重新導出子模組。
{% end %}

{% chat(speaker="jim") %}
explain 命名空間套件（Namespace Packages）
{% end %}

{% chat(speaker="chatgpt") %}
命名空間套件（Namespace Packages）是在 Python 3.3 以後引入，目的是讓同一個「套件名稱」可以分散在多個實體目錄或不同發行版上，解決傳統套件只能綁定到一個資料夾的限制。具體要點如下：
{% end %}

1. **隱式套件**

   * 不需要 `__init__.py` 檔案，只要一個目錄符合套件名稱結構，即可被當成 namespace package。
   * 當你執行 `import foo.bar` 時，Python 會在所有配置的路徑（如 `sys.path`）中，收集所有含有 `foo/bar` 子目錄的路徑，把它們「合併」成一個虛擬套件。

2. **跨套件整合**

   * 不同的第三方套件都可以往同一 namespace 下安裝子模組。例如：

     ```
     site-packages/pkgA/spkg/utils.py
     site-packages/pkgB/spkg/models.py
     ```

     即使兩者分屬不同發行版，使用者只要 `import spkg.utils, spkg.models` 都能載入。

3. **使用場景**

   * **大型商業專案**：多人團隊各自維護不同模組，但統一放在同一邏輯名稱空間下。
   * **插件架構**：主程式定義 `plugins` namespace，外部套件都以 `plugins.xxx` 命名，透過自動搜尋載入。

4. **限制與注意**

   * 不能在這個類型套件裡放初始化程式碼──因為沒有 `__init__.py`，無法執行任何套件層級的邏輯。
   * 若真的需要初始化行為，仍須在子模組或另行設計機制（如 entry point）處理。
   * 若誤把空目錄當 namespace，或混用隱式＋顯式（同時有 `__init__.py`），可能導致載入順序或相依性問題，建議結構規劃要謹慎。

5. **怎麼建立？**

   * 只要在你的套件目錄裡**不**放 `__init__.py`，並確保該路徑在 `sys.path` 中即可。
   * 例如：

     ```
     /project-root/
       └── myapp/
           └── feature_a/      ← 不含 __init__.py
           └── feature_b/      ← 不含 __init__.py
     ```

     這樣 `import myapp.feature_a`、`import myapp.feature_b` 都會被認作同一個 namespace。

{% chat(speaker="chatgpt") %}
命名空間套件讓你把一個「套件」拆散到多個地方、分散維護，但也犧牲了套件層級初始化的能力。使用前要想清楚：你需要的是彈性拆分，還是集中管理？如果後者，就老實地加個空的 `__init__.py` 吧，別讓 namespace 玩得太離譜。
{% end %}

{% chat(speaker="jim") %}
如果我沒有初始化套件的需求，也不在意統一暴露介面和命名空間套件，不放 `__init__.py` 是否有其它負面考量?
{% end %}

{% chat(speaker="chatgpt") %}
即便你不需要在套件載入時執行任何初始化程式，也不在意用 `__all__` 暴露介面、或跨多個路徑共用同一個命名空間，完全省略 `__init__.py` 還是有一些潛在的負面考量：
{% end %}

1. **向下相容性**

   * 只有 Python 3.3 以後支援隱式命名空間套件（PEP 420）。若你的專案需要同時在 Python 2 或更舊的 3.2− 版本上跑，就**一定**要加上空的 `__init__.py`，否則無法匯入套件。

2. **相對匯入（relative imports）**

   * 只有在真正的「套件」（即含 `__init__.py` 的目錄）裡，相對匯入才會如預期運作。若省略了，某些工具（或執行環境）在結合測試或打包時，可能會把你的子模組當成獨立 *module*，導致 `from .submod import X` 失敗。

3. **打包與部署工具偵測**

   * 很多工具（像是 setuptools 的 `find_packages()`、pytest 自動發現 test modules、Sphinx 文件產生、甚至某些 CI/CD pipeline）都是透過尋找目錄下的 `__init__.py` 來判斷「這是一個 package」。省略後，這些工具有可能：

     * 跳過你的子目錄，不當作套件載入
     * 無法帶進型別檢查（mypy）或換測試覆蓋率（coverage）報告
     * 打包時漏掉某些模組

4. **IDE 與靜態分析支援**

   * 大多數 IDE（如 PyCharm、VSCode + Pylance）在瀏覽專案結構、提供自動補全或重構時，也會參考 `__init__.py` 來判斷套件。省略可能降低開發效率（自動完成、跳轉定義、導出重命名等功能不穩定）。

5. **專案結構的可讀性**

   * 其他開發者看到有 `__init__.py` 時，直覺上會明白「這是一個 package」。如果什麼都沒放，資料夾裡只有一堆 `.py`，就不那麼一目瞭然。

{% chat(speaker="chatgpt") %}
如果你的環境**100%** 確定只跑 Python 3.3+、且只用直匯入（`import pkg.submod`）、又不透過任何自動發現工具，那麼真要省略也無妨；但為了兼容性、工具鏈穩定，以及專案易讀性／維護性，建議還是放一個空的 `__init__.py`。
{% end %}

{% chat(speaker="jim") %}
該如何切分它是一個「套件」? 我想我不應該在每個目錄中都放一個
{% end %}

{% chat(speaker="chatgpt") %}
要決定哪個目錄該當成「套件」（放 `__init__.py`），可以從以下幾個面向切分：
{% end %}

1. **功能邏輯邊界**

   * 把具備相近職責、常被一同使用的模組，放在同一個套件裡。
   * 例如：`auth/` 下放所有跟認證相關的程式碼（`login.py`、`jwt.py`、`oauth.py`），就該放 `auth/__init__.py`，讓外部只要 `import auth` 或 `from auth import login`。

2. **公開 API 封裝**

   * 思考使用者（或其他團隊）只要知道哪些模組／功能，剩下的就是內部實作。
   * 比如：`payment/` 套件裡暴露 `process()`、`refund()`，但把細節如 `stripe_utils.py` 隱藏在子子目錄 (`payment/_stripe/`)；底下若不希望被直接 import，就不放 `__init__.py`，讓它維持成普通資料夾。

3. **套件層級初始化需求**

   * 若真的需要執行套件級別的設定（例如全域變數、logging 設定），才放 `__init__.py`；否則就不放，保持乾淨。
   * 沒有初始化需求的資料夾，就當作純資料（assets、templates）或內部實作，不當作套件。

4. **目錄深度與複雜度**

   * 不要為了拆深度而把每個子目錄都做成套件。通常淺層（1\~2層）就夠，底下更深的只有在有強烈模組化需求時才拆。
   * 例如：

     ```
     myproject/
       ├─ api/           ← 套件
       │   ├─ __init__.py
       │   ├─ v1/        ← 子套件
       │   │   ├─ __init__.py
       │   │   └─ users.py
       │   └─ v2/        ← 子套件
       │       ├─ __init__.py
       │       └─ users.py
       └─ utils/         ← 單純工具函式，不放 __init__.py
           └─ helpers.py
     ```

     只有 `api`、`api/v1`、`api/v2` 是對外匯出的套件，其它就當成內部程式碼資料夾。

5. **工具／框架自動發現需求**

   * 如果你在用 pytest、Sphinx、setuptools 等工具自動找 package，就放 `__init__.py` 在需要被發現的目錄，避免漏掃。

---

**總結**：

* 先分門別類「哪些模組要被 import、對外暴露」，在那些目錄下放 `__init__.py`；
* 其餘純內部實作或資源，就當普通資料夾，不必放。
* 保持套件邊界清晰，既能讓專案易維護，也不會胡亂在每層目錄都加冗餘檔案。
