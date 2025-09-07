+++
title = "12GB VRAM 顯卡運行 GPT-OSS-20B：vLLM 與統一記憶體實戰指南"
description = "探索如何在 12GB VRAM NVIDIA 顯卡上，結合 vLLM 與 Unified Memory 技術運行 gpt-oss-20b。內容涵蓋安裝步驟、參數調整與效能挑戰。適合想在有限資源下部署大型語言模型的進階用戶。"
date = "2025-08-07T18:58:47Z"
updated = "2025-09-07T16:46:26Z"
draft = false

[taxonomies]
tags = [ "LLM", "Python", "uv", "Container" ]
providers = [ "Felo Search" ]

[extra]
withAI = "<https://felo.ai/search/8UNyhKAzMzCaBkmYGPimXX?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
I have a wsl machine with nvidia gpu. Guide me run gpt-oss-20b with vllm
{% end %}

{% chat(speaker="felo") %}
好的，這是一份關於如何在具備 NVIDIA GPU 的 WSL 環境中，使用 vLLM 執行 `gpt-oss-20b` 模型的詳細研究報告。
{% end %}

## **總覽**

在 Windows Subsystem for Linux (WSL) 環境中，透過 vLLM 高效能推理引擎來執行 OpenAI 的 `gpt-oss-20b` 開源模型是完全可行的。此模型因其約 16GB VRAM 的需求，相當適合在單張高階消費級或工作站級的 NVIDIA GPU 上部署[^1][^4][^17]。整個流程的核心在於正確設定 WSL2 環境，包含安裝適用於 WSL 的 NVIDIA 驅動程式與 CUDA 工具鏈，接著安裝特定版本的 vLLM 及其相依套件，最終啟動一個與 OpenAI API 相容的推理伺服器[^27][^46]。vLLM 專為 `gpt-oss` 系列模型進行了深度優化，支援其獨特的 MXFP4 量化與高效注意力機制，確保了在本地端也能獲得優異的推理性能[^29]。本報告將引導您完成從環境設定到模型部署的每一個步驟，並涵蓋常見的疑難排解方案。

### **模型與硬體需求**

`gpt-oss-20b` 是 OpenAI 開源的強大模型之一，其設計旨在兼顧性能與可及性。

**模型特性**
`gpt-oss-20b` 擁有 209 億個參數，並透過創新的 MXFP4 格式對其 Mixture-of-Experts (MoE) 權重進行量化[^4][^29]。MXFP4 是一種 4-bit 浮點數格式，此技術大幅壓縮了模型的尺寸，使其 VRAM 需求降低至約 14-16GB，從而能夠在單張 GPU 上順利執行[^1][^18][^29]。模型架構還融合了全注意力 (full attention) 與滑動窗口注意力 (sliding window attention)，並由 vLLM 的混合 KV 快取分配器進行高效管理，以達到最佳的記憶體使用效率[^29]。

**硬體建議**
要執行 `gpt-oss-20b`，建議配備至少擁有 16GB VRAM 的 NVIDIA GPU[^1][^27]。這包括了高階消費級顯示卡（如 GeForce RTX 4090）或工作站級顯示卡（如 NVIDIA RTX A6000）[^38][^47]。

為了更清晰地了解其定位，下表比較了 `gpt-oss-20b` 與其更大規模的版本 `gpt-oss-120b`：

| 特性 | `openai/gpt-oss-20b` | `openai/gpt-oss-120b` |
| :--- | :--- | :--- |
| **參數數量** | 20.9B[^4] | ~120B[^1] |
| **所需 VRAM** | ~16GB[^1][^4][^27] | ≥60GB[^1][^27] |
| **MoE 專家數量** | 32[^29] | 128[^29] |
| **適用硬體** | 單張消費級/工作站 GPU (例如 RTX 4090, A6000)[^38][^47] | 伺服器級 GPU (例如 H100, B200) 或多 GPU 設定[^1][^12][^27] |

### **WSL 環境設定**

一個正確設定的 WSL 環境是成功執行 vLLM 的基礎。

**先決條件**
請確保您的系統已安裝並啟用 WSL 2[^46]。

**NVIDIA 驅動程式與 CUDA**
WSL 需要專用的 NVIDIA 驅動程式才能存取 GPU。

1. **檢查 NVCC**：在您的 WSL 終端機中，執行以下指令來檢查 NVIDIA CUDA Compiler (NVCC) 是否已安裝：

    ```bash
    nvcc --version
    ```

    如果出現 `command not found` 的錯誤訊息，代表您需要安裝或設定 CUDA 工具鏈[^46]。

2. **安裝驅動程式**：前往 NVIDIA 官方網站下載適用於 WSL-Ubuntu 的驅動程式[^46]。請依照頁面上的指示完成安裝。

3. **設定環境變數**：安裝完成後，編輯您的 `.bashrc` 檔案（例如 `nano ~/.bashrc`），並在檔案末尾加入以下路徑。請注意，您的 CUDA 版本可能不同，請根據 `/usr/local` 目錄下的實際情況修改路徑[^46]。

    ```bash
    export PATH="/usr/local/cuda-12.6/bin:$PATH"
    export LD_LIBRARY_PATH="/usr/local/cuda-12.6/lib64:$LD_LIBRARY_PATH"
    ```

4. **重新整理設定**：儲存檔案後，執行 `source ~/.bashrc` 來讓設定生效。接著，再次執行 `nvcc --version` 和 `nvidia-smi.exe` 來確認驅動程式與 CUDA 已被正確偵測[^46]。

**Python 虛擬環境**
為了避免套件版本衝突，強烈建議在虛擬環境中進行安裝[^28]。vLLM 官方文件推薦使用 `uv` 來管理環境[^27]。

1. **建立虛擬環境**：

    ```bash
    uv venv
    ```

2. **啟動虛擬環境**：

    ```bash
    source .venv/bin/activate
    ```

### **安裝 vLLM 與相依套件**

執行 `gpt-oss` 模型需要安裝特定版本的 vLLM 及多個前沿的相依套件[^28]。

**安裝指令**
請使用以下指令進行安裝。這個指令會從指定的索引 URL 安裝 vLLM 的預發行版、PyTorch nightly build 以及其他必要的函式庫[^5][^27][^28]。

```bash
uv pip install --pre vllm==0.10.1+gptoss \
    --extra-index-url https://wheels.vllm.ai/gpt-oss/ \
    --extra-index-url https://download.pytorch.org/whl/nightly/cu128 \
    --index-strategy unsafe-best-match
```

此指令確保了所有元件之間的相容性，是成功部署的關鍵步驟[^25][^28]。

### **啟動 vLLM 推理伺服器**

安裝完成後，即可啟動 vLLM 伺服器來載入 `gpt-oss-20b` 模型。

**基本啟動**
執行以下指令，vLLM 將會自動從 Hugging Face 下載模型，並在 `localhost:8000` 啟動一個與 OpenAI API 相容的伺服器[^8][^23][^27]。

```bash
vllm serve openai/gpt-oss-20b
```

**進階啟動選項**
為了提升性能與穩定性，可以加入額外的參數。`--async-scheduling` 可以啟用非同步排程以提高 GPU 利用率，而 `--gpu-memory-utilization` 則可以控制 VRAM 的使用比例[^28][^47]。

```bash
vllm serve openai/gpt-oss-20b \
  --host 0.0.0.0 \
  --port 8000 \
  --gpu-memory-utilization 0.9 \
  --async-scheduling
```

**Docker 替代方案**
如果您偏好使用容器化部署，vLLM 也提供了預先建構好的 Docker image[^5][^28]。

```bash
docker run --gpus all \
    -p 8000:8000 \
    --ipc=host \
    vllm/vllm-openai:gptoss \
    --model openai/gpt-oss-20b
```

### **與模型互動**

伺服器啟動並顯示 `INFO: Application startup complete` 訊息後，您便可以使用 OpenAI 的 Python SDK 與模型進行互動[^28]。

**Python 程式碼範例**
以下 Python 程式碼片段展示了如何向本地端執行的模型發送請求[^27]。請注意，`base_url` 必須指向您的 vLLM 伺服器位址，而 `api_key` 在此情境下可以是任意非空字串[^27]。

```python
from openai import OpenAI

# 初始化用戶端，指向本地 vLLM 伺服器
client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="EMPTY" 
)

# 建立聊天完成請求
result = client.chat.completions.create(
    model="openai/gpt-oss-20b",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain what vLLM is in simple terms."}
    ]
)

# 列印模型的回答
print(result.choices[^0].message.content)
```

### **常見問題與疑難排解**

在設定過程中，您可能會遇到一些特定的問題。

* **`EngineDeadError: no kernel image is available`**
    這個錯誤通常發生在非 Hopper 或 Blackwell 架構的 GPU（如 RTX A6000）上[^47]。它表示您的 GPU 硬體不支援 vLLM 為 `gpt-oss` 的 MXFP4 量化所使用的特定 CUDA 核心。這是因為這些核心是為最新的 GPU 架構編譯的[^29][^47]。解決此問題可能需要等待 vLLM 或 Triton 的更新，以提供對更多 GPU 架構的支援。

* **PyTorch/CUDA `ImportError`**
    如果在啟動伺服器時遇到類似 `undefined symbol: __nvJitLinkComplete_12_4` 的 `ImportError`，這通常是 PyTorch 版本與您的 CUDA 驅動程式不相容所致[^46]。解決方法是卸載現有的 PyTorch，並使用官方提供的指令從指定的 nightly build 索引 URL 重新安裝[^46]。

* **vLLM 版本錯誤**
    若使用標準的 `pip install vllm` 指令，很可能會因為版本過舊而無法支援 `gpt-oss`[^35]。務必使用本報告中提供的、包含 `--pre` 和 `--extra-index-url` 參數的特定安裝指令，以確保安裝正確的版本[^27][^28]。

* **WSL 中 `nvcc: command not found`**
    此問題的根本原因是在您的 shell 設定檔（如 `.bashrc`）中缺少 CUDA 工具鏈的路徑[^46]。請返回「WSL 環境設定」一節，確認 `PATH` 和 `LD_LIBRARY_PATH` 環境變數已正確設定並生效。

{% chat(speaker="jim") %}
Is there any chance to run the gpt-oss-20b model on 8G/12G VRAM NV card? Like using quantization or adjusting latency.
{% end %}

{% chat(speaker="felo") %}
在 8GB 或 12GB VRAM 的 NVIDIA 顯卡上運行 GPT-OSS-20B 模型是具有挑戰性的，但並非完全不可能。以下是一些可能的解決方案與調整策略：
{% end %}

## **1. 使用量化技術**

GPT-OSS-20B 模型原生支援 MXFP4 量化，這種技術可以顯著降低模型的記憶體需求。根據資料，MXFP4 量化可以將記憶體需求減少 50% 至 75%[^1018]。然而，即使如此，模型仍需要約 16GB 的 VRAM 才能在標準配置下運行[^1001][^1004]。

**可能的解決方案：**

* **進一步量化：** 社群中有討論使用更激進的量化技術，例如 4-bit Attention 和 2-bit MLP，這可能進一步降低 VRAM 需求，但可能會影響模型的準確性[^1022]。
* **低精度運算：** 使用 BF16 或 INT8 精度進行推理，這可以進一步減少記憶體使用量。

---

## **2. 使用分片或分層加載技術**

Mixture-of-Experts (MoE) 架構允許 GPT-OSS-20B 僅激活部分參數（每次推理約 3.6B 參數），這為分片或分層加載提供了可能性[^1002][^1020]。

**可能的解決方案：**

* **分片推理：** 將模型分割成多個部分，並在推理過程中動態加載所需的部分參數。
* **分層加載：** 僅加載模型的低層部分以處理簡單任務，這樣可以降低記憶體需求。

---

## **3. 使用優化的推理框架**

一些專為低記憶體環境設計的推理框架可以幫助在 8GB 或 12GB VRAM 的顯卡上運行 GPT-OSS-20B。

**推薦框架：**

* **vLLM：** vLLM 是一個高效的推理引擎，專為大模型設計，支援非同步排程和記憶體優化[^1009][^1013]。
* **DeepSpeed：** DeepSpeed 提供 ZeRO-Inference 技術，可以顯著降低記憶體需求，適合低 VRAM 環境[^1010]。
* **llama.cpp：** 雖然主要針對 LLaMA 模型，但其輕量化設計可能適用於 GPT-OSS-20B 的某些場景[^1021]。

---

## **4. 調整推理參數**

GPT-OSS-20B 支援調整推理參數（如推理深度和上下文長度），這可以降低記憶體需求。

**可能的調整：**

* **降低上下文長度：** 減少輸入的上下文長度可以減少記憶體使用量[^1020]。
* **降低推理努力：** 將推理努力設置為「低」或「中」，以減少計算負擔[^1019]。

---

## **5. 使用混合記憶體技術**

如果顯卡的 VRAM 不足，可以考慮使用混合記憶體技術，將部分模型權重存儲在系統 RAM 中。

**可能的解決方案：**

* **Unified Memory（統一記憶體）：** NVIDIA 的統一記憶體技術允許 GPU 和 CPU 共享記憶體資源[^1011]。
* **分層存儲：** 將不常用的模型部分存儲在 RAM 或磁碟中，僅在需要時加載。

---

## **6. 測試小型模型或微調版本**

如果上述方法仍無法滿足需求，可以考慮使用更小的模型或微調版本。

**可能的選擇：**

* **微調 GPT-OSS-20B：** 微調模型以適應特定任務，並裁剪不必要的參數。
* **使用更小的模型：** 考慮使用 GPT-OSS 的更小版本或其他輕量化模型。

---

## **結論**

在 8GB 或 12GB VRAM 的 NVIDIA 顯卡上運行 GPT-OSS-20B 是困難的，但通過量化、分片推理、優化框架和混合記憶體技術等方法，可以嘗試實現此目標。然而，這些方法可能會影響模型的性能或準確性，因此需要根據具體需求進行權衡。

{% chat(speaker="jim") %}
Guide me run the gpt-oss-20b model on 12G VRAM card with vllm, and Unified Memory tech.
{% end %}

{% chat(speaker="felo") %}
在具有 12GB VRAM 的 NVIDIA 顯卡上運行 `gpt-oss-20b` 模型，即使該模型通常需要約 16GB 的 VRAM，透過結合 vLLM 的記憶體管理功能和 NVIDIA 的統一記憶體 (Unified Memory) 技術，仍存在實現的可能性，但會面臨挑戰。
{% end %}

## **模型與硬體要求**

`gpt-oss-20b` 是一個擁有 209 億參數的開源模型，其 MoE (Mixture-of-Experts) 層使用原生 MXFP4 量化技術，將所需的 VRAM 降低至約 16GB 才能高效運行[^2003][^2004][^2010][^2017]。這表示對於 12GB VRAM 的顯卡而言，直接將整個模型載入 GPU 記憶體會超出容量。

## **使用 vLLM 執行 GPT-OSS-20B 的步驟**

以下是在具備 NVIDIA GPU 的 WSL 環境中，使用 vLLM 執行 GPT-OSS-20B 模型的具體要求和步驟：

**1. 先決條件**

* 確保您的 Windows 機器上已安裝並設定 WSL 2。
* 您的 NVIDIA GPU 應具有 7.0 或更高的運算能力。
* vLLM 支援 Python 版本 3.9 到 3.12。

**2. 安裝 vLLM**
vLLM 建議使用 `uv` 管理您的 Python 環境。若要建立新的虛擬環境並安裝 vLLM，請執行以下命令。這些命令將安裝 vLLM 的預發行版、PyTorch nightly build 以及其他必要的程式庫[^2001][^2003][^2004][^2017]：

```bash
uv venv
source .venv/bin/activate
uv pip install --pre vllm==0.10.1+gptoss \
    --extra-index-url https://wheels.vllm.ai/gpt-oss/ \
    --extra-index-url https://download.pytorch.org/whl/nightly/cu128 \
    --index-strategy unsafe-best-match
```

**3. 啟動伺服器並載入模型**
vLLM 提供了 `serve` 命令，該命令將自動從 Hugging Face 下載模型並在 `localhost:8000` 上啟動一個與 OpenAI 相容的伺服器[^2003][^2005][^2017]。

針對 12GB VRAM 的顯卡，為了盡可能利用有限的記憶體並嘗試啟用統一記憶體等機制，您需要特別留意 `gpu-memory-utilization` 參數。此參數指定了 vLLM 應使用的 GPU 記憶體比例。當值大於 1 時，vLLM 會嘗試使用作業系統管理統一記憶體來擴展到主機記憶體[^2001]。

```bash
vllm serve openai/gpt-oss-20b \
    --host 0.0.0.0 \
    --port 8000 \
    --gpu-memory-utilization 0.95 \ # 盡量提高 GPU 記憶體利用率
    --enforce-eager \             # 確保某些操作不會延遲，可能有利於調試
    --max-model-len 4096 \        # 根據 VRAM 限制調整最大上下文長度
    --max-num-batched-tokens 256  # 減少批次處理的總 token 數，以減少記憶體壓力
```

請注意，儘管設置 `gpu-memory-utilization` 大於 1 可以讓 vLLM 嘗試利用系統記憶體作為額外的 KV 快取或權重卸載空間，但這並非適用於所有模型或所有 GPU 架構。對於 `gpt-oss-20b`，vLLM 團隊提到可以利用 CPU 記憶體進行 KV 快取交換或權重卸載[^2019]。

## **統一記憶體 (Unified Memory) 與效能調整**

NVIDIA 的統一記憶體技術允許 CPU 和 GPU 共享相同的可定址記憶體空間，這對於 VRAM 有限的系統非常有幫助，因為它可以將部分資料或模型權重透明地移出 VRAM 到系統 RAM 中[^2019]。

**挑戰與注意事項：**

* **效能影響：** 即使成功將部分模型數據卸載到系統記憶體，從系統記憶體存取數據會比直接從 VRAM 存取慢得多，這將顯著影響推理速度。
* **CUDA 相容性：** 統一記憶體的支援程度可能因 CUDA 版本和 GPU 架構而異。
* **模型配置：** `gpt-oss-20b` 已經採用 MXFP4 量化，這是它在 16GB VRAM 上運行的主要原因。更進一步的量化（例如 8-bit 或 4-bit 權重和激活量化）可能可以使其適應 12GB VRAM，但這可能需要模型層面的修改，且可能會犧牲模型精確度[^2004]。
* **錯誤處理：** 在 VRAM 不足的情況下，您可能會遇到 `CUDA out of memory` 或 `EngineDeadError` 等錯誤。這通常意味著模型無法完全載入，或者在處理請求時耗盡了記憶體[^2001]。
* **其他優化：**
  * **減少最大上下文長度：** 透過 `--max-model-len` 參數減少模型處理的最大 token 數可以降低記憶體消耗。
  * **降低批次大小：** 減少同時處理的請求數量或每個請求的 token 數量可以降低峰值記憶體使用率。
  * **啟用非同步排程：** 加入 `--async-scheduling` 參數可以啟用非同步排程，有助於提高 GPU 利用率，但在特定情況下可能不相容於結構化輸出[^2001]。

總而言之，在 12GB VRAM 的顯卡上運行 `gpt-oss-20b` 需要仔細的配置和效能監控。如果遇到困難，可能需要考慮使用 VRAM 更大的 GPU，或選擇體積更小的模型。

[^1]: [How to run gpt-oss with vLLM | OpenAI Cookbook](https://cookbook.openai.com/articles/gpt-oss/run-vllm)
[^4]: [Things to Know About OpenAI GPT-OSS: Run it Locally on ...](https://medium.com/@isaakmwangi2018/things-to-know-about-openai-gpt-oss-run-it-locally-on-your-device-hardware-requirements-e266e0f1700f)
[^5]: [vLLM Now Supports gpt-oss](https://blog.vllm.ai/2025/08/05/gpt-oss.html)
[^8]: [openai/gpt-oss-20b - Hugging Face](https://huggingface.co/openai/gpt-oss-20b)
[^12]: [Run OpenAI's gpt-oss model with vLLM | Modal Docs](https://modal.com/docs/examples/gpt_oss_inference)
[^17]: [Things to Know About OpenAI GPT-OSS: Run it Locally on ...](https://medium.com/@isaakmwangi2018/things-to-know-about-openai-gpt-oss-run-it-locally-on-your-device-hardware-requirements-e266e0f1700f)
[^18]: [You can now run OpenAI's gpt-oss model on your local device ...](https://www.reddit.com/r/selfhosted/comments/1mjbwgn/you_can_now_run_openais_gptoss_model_on_your/)
[^23]: [openai/gpt-oss-20b - Hugging Face](https://huggingface.co/openai/gpt-oss-20b)
[^25]: [LMCache supports gpt-oss (20B/120B) on Day 1](https://blog.lmcache.ai/2025-08-05-gpt-oss-support/)
[^27]: [How to run gpt-oss with vLLM](https://cookbook.openai.com/articles/gpt-oss/run-vllm)
[^28]: [GPT OSS - vLLM Recipes](https://docs.vllm.ai/projects/recipes/en/latest/OpenAI/GPT-OSS.html)
[^29]: [vLLM Now Supports gpt-oss | vLLM Blog](https://blog.vllm.ai/2025/08/05/gpt-oss.html)
[^35]: [openai/gpt-oss-20b · Latest vLLM docker image doesn't yet ...](https://huggingface.co/openai/gpt-oss-20b/discussions/6)
[^38]: [GPT‑OSS Explained: OpenAI's Powerful Open-Weight ...](https://medium.com/@sahin.samia/gpt-oss-explained-openais-powerful-open-weight-language-models-for-local-inference-aaa15c7fc210)
[^46]: [Making VLLM work on WSL2 - DEV Community](https://dev.to/docteurrs/making-vllm-work-on-wsl2-482e)
[^47]: [[Bug]:Deploying GPT-OSS-20B: EngineDeadError: EngineCore encountered an issue. See stack trace (above) for the root cause. · Issue #22414 · vllm-project/vllm · GitHub](https://github.com/vllm-project/vllm/issues/22414)
[^1001]: [Everything You Need to Know About GPT-OSS in 2025 - remio](https://www.remio.ai/post/gpt-oss-key-features-open-source-llm-2025-overview)
[^1002]: [Introducing gpt-oss - OpenAI](https://openai.com/index/introducing-gpt-oss/)
[^1004]: [GPT-OSS-120B & 20B: OpenAI's Powerful New Open-Source ...](https://medium.com/@tahirbalarabe2/gpt-oss-120b-20b-openais-powerful-new-open-source-models-explained-1f706123a30f)
[^1009]: [How to run gpt-oss with vLLM - OpenAI Cookbook](https://cookbook.openai.com/articles/gpt-oss/run-vllm)
[^1010]: [Things to Know About OpenAI GPT-OSS: Run it Locally on ...](https://medium.com/@isaakmwangi2018/things-to-know-about-openai-gpt-oss-run-it-locally-on-your-device-hardware-requirements-e266e0f1700f)
[^1011]: [You can now run OpenAI's gpt-oss model on your local device ...](https://www.reddit.com/r/selfhosted/comments/1mjbwgn/you_can_now_run_openais_gptoss_model_on_your/)
[^1013]: [vLLM Now Supports gpt-oss](https://blog.vllm.ai/2025/08/05/gpt-oss.html)
[^1018]: [OpenAI's Bold Return to Open Source: GPT-OSS Changes ...](https://medium.com/aimonks/openais-bold-return-to-open-source-gpt-oss-changes-everything-20d071d3c76f)
[^1019]: [gpt-oss-20b Model by OpenAI - NVIDIA NIM APIs](https://build.nvidia.com/openai/gpt-oss-20b/modelcard)
[^1020]: [OpenAI: gpt-oss-20b (free) – Uptime and Availability](https://openrouter.ai/openai/gpt-oss-20b:free/uptime)
[^1021]: [llama.cpp 才是王道: r/LocalLLaMA - Reddit](https://www.reddit.com/r/LocalLLaMA/comments/1j417qh/llamacpp_is_all_you_need/?tl=zh-hans)
[^1022]: [Unsloth - 微调Mistral 模型，速度提升220% - 征求建议 - Reddit](https://www.reddit.com/r/LocalLLaMA/comments/1af6mq1/unsloth_finetune_mistral_220_faster_asking_for/?tl=zh-hans)
[^2001]: [GPT OSS - vLLM Recipes](https://docs.vllm.ai/projects/recipes/en/latest/OpenAI/GPT-OSS.html)
[^2003]: [OpenAI Just Released Open Models (this is a Very Big Deal)](https://simple.ai/p/openai-just-released-open-models)
[^2004]: [Things to Know About OpenAI GPT-OSS: Run it Locally on ...](https://medium.com/@isaakmwangi2018/things-to-know-about-openai-gpt-oss-run-it-locally-on-your-device-hardware-requirements-e266e0f1700f)
[^2005]: [How to run gpt-oss with vLLM - OpenAI Cookbook](https://cookbook.openai.com/articles/gpt-oss/run-vllm)
[^2010]: [Run OpenAI's new GPT-OSS (open-source) model on Northflank](https://northflank.com/blog/self-host-openai-gpt-oss-120b-open-source-chatgpt)
[^2017]: [openai/gpt-oss-20b - Hugging Face](https://huggingface.co/openai/gpt-oss-20b)
[^2019]: [[Feature]: Support for NVIDIA Unified memory #10267 - GitHub](https://github.com/vllm-project/vllm/issues/10267)
