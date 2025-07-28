+++
title = "Linux CLI 圖像相似度比對工具推薦與 zsh 腳本實作"
description = "想在 Linux CLI 下找出相似度 95% 以上的圖片嗎？本文精選多款圖像比對工具，並附上 zsh 腳本範例，助您高效管理大量圖片。"
date = "2025-07-01T00:00:00Z"
updated = "2025-07-28T17:19:25.730Z"
draft = false

[taxonomies]
tags = [ "Linux", "Zsh" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/JuprGJPBkjSbg7NLJPAQ6C?invite=dOLYGeJyZJqVX>"
+++
{% editor_note() %}
我後來在 [jim60105/image-dataset-prep-tools](https://github.com/jim60105/image-dataset-prep-tools) 專案中使用了 `czkawka_cli` 工具。
{% end %}

{% chat(speaker="jim") %}
我正在寫 zsh 腳本，推薦我幾個可以在 linux cli 執行的圖像相似度比對工具，我需要能把某資料夾下相似度 95% 以上的圖片找出來
{% end %}

{% chat(speaker="felo") %}
基於您的需求，我推薦幾個在 Linux CLI 環境下可以執行的圖像相似度比對工具，這些工具都能幫助您找出資料夾中相似度 95% 以上的圖片。
{% end %}

## **推薦的 CLI 圖像相似度比對工具**

### **1. findimagedupes**

<https://github.com/jhnc/findimagedupes>

`findimagedupes` 是 Linux 平台上最常用的命令列圖像相似度檢測工具[^3]。它專門設計用於尋找視覺上相似的圖片，而不僅僅是完全相同的檔案。

```bash
# 安裝 (Ubuntu/Debian)
sudo apt-get install findimagedupes

# 基本用法 - 在指定資料夾中尋找相似圖片
findimagedupes /path/to/your/images/

# 設定相似度閾值 (0.0-1.0，0.95 表示 95% 相似度)
findimagedupes -t 0.95 /path/to/your/images/
```

### **2. ImageMagick compare**

<https://imagemagick.org/script/compare.php>

ImageMagick 的 `compare` 工具提供強大的圖像比較功能[^7][^16]。雖然主要用於兩張圖片的比較，但可以結合 shell 腳本進行批次處理。

```bash
# 安裝 ImageMagick
sudo yum install ImageMagick  # RHEL/CentOS

# 比較兩張圖片的相似度
compare -metric SSIM image1.jpg image2.jpg null:

# 在 zsh 腳本中批次比較
for img1 in *.jpg; do
    for img2 in *.jpg; do
        if [[ "$img1" != "$img2" ]]; then
            similarity=$(compare -metric SSIM "$img1" "$img2" null: 2>&1)
            # 處理相似度結果
        fi
    done
done
```

### **3. Czkawka (CLI 版本)**

<https://github.com/qarmin/czkawka>

Czkawka 是一個現代化的重複檔案查找工具，支援圖像相似度檢測[^26]。它提供純文字版本，非常適合在 CLI 環境中使用。

```bash
# 下載並安裝 Czkawka CLI 版本
wget https://github.com/qarmin/czkawka/releases/latest/download/linux_czkawka_cli.tar.xz
tar -xf linux_czkawka_cli.tar.xz

# 尋找相似圖片
./czkawka_cli similar -d /path/to/images/ -s High
```

### **4. rmlint**

<https://github.com/sahib/rmlint>

`rmlint` 是一個高效的重複檔案查找工具，也支援圖像相似度檢測[^8]。它以速度快著稱，適合處理大量圖片。

```bash
# 安裝 rmlint
sudo yum install rmlint  # RHEL

# 尋找相似圖片
rmlint --types="duplicates" --algorithm=paranoid /path/to/images/
```

## **針對 95% 相似度的 zsh 腳本範例**

以下是一個使用 `findimagedupes` 的 zsh 腳本範例：

```bash
#!/bin/zsh

# 設定目標資料夾
IMAGE_DIR="/path/to/your/images"
SIMILARITY_THRESHOLD=0.95
OUTPUT_FILE="similar_images_95percent.txt"

# 檢查 findimagedupes 是否已安裝
if ! command -v findimagedupes &> /dev/null; then
    echo "錯誤：findimagedupes 未安裝"
    echo "請執行：sudo yum install findimagedupes"
    exit 1
fi

# 執行相似度檢測
echo "正在搜尋相似度 ${SIMILARITY_THRESHOLD} 以上的圖片..."
findimagedupes -t ${SIMILARITY_THRESHOLD} -R "${IMAGE_DIR}" > "${OUTPUT_FILE}"

# 顯示結果
if [[ -s "${OUTPUT_FILE}" ]]; then
    echo "找到相似圖片，結果已儲存至：${OUTPUT_FILE}"
    echo "相似圖片組數：$(grep -c '^$' "${OUTPUT_FILE}")"
else
    echo "未找到相似度 ${SIMILARITY_THRESHOLD} 以上的圖片"
fi
```

## **效能考量與建議**

根據您的 Linux RHEL 環境，建議：

1. **對於大量圖片**：使用 `rmlint` 或 `Czkawka`，它們在處理大型資料集時效能較佳[^8]
2. **精確度要求高**：使用 `findimagedupes`，它專門針對圖像相似度設計[^3]
3. **需要自訂演算法**：使用 ImageMagick，可以選擇不同的比較指標（SSIM、PSNR 等）[^13]

這些工具都能有效地在您的 zsh 腳本中找出 95% 以上相似度的圖片，您可以根據具體需求和效能要求選擇最適合的工具。

[^3]: [Find visually similar images to a given image on Linux](https://softwarerecs.stackexchange.com/questions/17429/find-visually-similar-images-to-a-given-image-on-linux)
[^7]: [ImageMagick Examples -- Image Comparing](https://imagemagick.org/Usage/compare/)
[^8]: [Find Duplicate and Similar Images with Open-Source Tools](https://www.pixduplicate.com/blog/find-duplicate-similar-images-open-source/)
[^13]: [Compare Two Images in Linux - Baeldung](https://www.baeldung.com/linux/image-comparison)
[^16]: [Command-line Tools: Compare - ImageMagick](https://imagemagick.org/script/compare.php)
[^26]: [Czkawka軟體用法，Linux以圖搜圖、搜尋重複檔案的小工具](https://ivonblog.com/posts/czkawka-find-duplicates/)
