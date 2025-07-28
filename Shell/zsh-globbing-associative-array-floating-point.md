+++
title = "Zsh 實戰：通配符、關聯陣列與浮點運算全攻略"
description = "深入解析 Zsh 在通配符擴展、關聯陣列與浮點數運算的強大功能，並以實際程式碼範例展示其語法優勢。比較 Zsh 與 Bash 的語法差異，協助開發者寫出更高效的 shell script。想提升命令列生產力，這篇你不能錯過！"
date = "2025-07-01T00:00:00Z"
updated = "2025-07-28T18:03:57Z"
draft = false

[taxonomies]
tags = [ "Zsh" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/mbHKVG9UzZisrxkaHHnSxU?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
深入研究 what are the advantages of Zsh over Bash?  
對於一個開發者，在撰寫兩種 shell script 時要特別注意什麼它們不一樣的地方?  
說明在撰寫 .zsh shell script 時和 bash script 有什麼不同之處，有哪些不同的語言特性可供使用。  
Why should I choose Zsh?
{% end %}

{% chat(speaker="felo") %}
根據搜尋結果，我將詳細分析相關問題。
{% end %}

Zsh (Z Shell) 是一種功能強大且高度可自訂的 Unix shell，它在 Bash (Bourne-Again Shell) 的基礎上進行了擴展，特別是在互動式使用方面提供了顯著的優勢。對於開發者而言，Zsh 透過其進階的自動補全、豐富的插件生態系統、美觀的主題以及拼字校正等功能，大幅提升了命令列操作的效率與體驗[^3][^10][^107]。儘管 Zsh 在互動性上勝出，Bash 憑藉其在絕大多數 Linux 發行版中的預設地位，依然是撰寫具備最高可攜性腳本的首選[^11][^36][^163]。

在撰寫 shell script 時，開發者必須意識到兩者之間存在的關鍵語法差異。Zsh 在陣列處理（預設為 1-based 索引）、通配符擴展（globbing）、參數擴展的預設行為以及原生浮點數運算支援等方面，都與 Bash 有所不同[^27][^113][^148][^163]。理解這些差異對於撰寫正確、高效且可維護的腳本至關重要。選擇 Zsh 意味著擁抱一個更現代、更具生產力的互動式命令列環境，而選擇 Bash 則優先考慮了腳本的穩定性與跨平台相容性[^51][^114]。

## Shell 的世界：Bash 與 Zsh

Shell 是一個命令列直譯器，作為使用者與作業系統核心之間的核心介面[^14][^51]。它解譯使用者輸入的指令，並與系統核心溝通以執行從簡單的檔案列表到複雜的自動化工作流程等各種任務[^14]。

### Bash (Bourne-Again Shell)

Bash 是 GNU 計畫為取代早期 Bourne Shell (sh) 而開發的自由軟體，現已成為絕大多數 Linux 發行版（如 Ubuntu, Fedora, CentOS）和舊版 macOS 的預設 shell[^11][^33][^45][^162]。它以其簡潔性、廣泛的可用性和強大的腳本編寫能力而聞名，是系統管理和自動化任務的基石[^67][^162]。Bash 的語法相對直觀，使其成為初學者進入命令列世界的絕佳起點[^162]。

### Zsh (Z Shell)

Zsh 由 Paul Falstad 於 1990 年開發，是一種功能極為豐富的 Unix shell，它整合並擴展了 Bash、ksh 和 tcsh 等主流 shell 的功能[^9][^13][^71][^75]。Zsh 的設計目標是提供一個更強大、更具互動性的使用者體驗，並允許深度自訂[^3][^4][^49]。自 2019 年起，Apple 在 macOS Catalina 版本中將 Zsh 設為預設 shell，Kali Linux 也隨後跟進，這標誌著其在現代 Unix-like 系統中的重要地位日益提升[^1][^13][^49]。

## 為何選擇 Zsh？對開發者的主要優勢

對於日常大量使用終端機的開發者來說，從 Bash 轉換到 Zsh 可以帶來顯著的生產力提升。這些優勢主要體現在互動式操作上[^16][^108]。

### 進階的自動補全

這是 Zsh 最受推崇的功能之一[^7][^23]。與 Bash 僅能列出可能的補全選項不同，Zsh 的補全系統是可互動的，它會以選單形式呈現選項，使用者可以透過方向鍵或 Tab 鍵進行導航和選擇[^26][^38][^106]。這種上下文感知的補全不僅適用於指令和檔案路徑，還擴展到指令的選項和參數，甚至可以顯示選項的簡短說明，極大地方便了新指令的探索[^33][^119]。

### 豐富的插件與主題生態系

透過像「Oh My Zsh」這樣的開源框架，使用者可以極其輕鬆地管理 Zsh 的設定、插件和主題[^7][^19][^110]。這個龐大的生態系提供了數百個插件，功能涵蓋 Git 狀態顯示、語法高亮、Docker 整合等，能將終端機打造成一個資訊豐富且視覺上吸引人的工作空間[^10][^107][^124]。

### 智慧功能提升效率

* **拼字校正**：當輸入錯誤的指令時，Zsh 會自動偵測並提示可能的正確指令，例如將 `sl` 修正為 `ls`[^94][^107][^110]。
* **共享指令歷史**：所有開啟的 Zsh 終端機視窗共享同一個指令歷史記錄檔案，方便使用者在不同工作階段中查找和重用指令[^13][^49][^89]。
* **智慧目錄導航**：Zsh 允許使用者直接輸入目錄名稱來切換路徑，無需輸入 `cd` 指令[^33][^110]。此外，`d` 指令可以列出最近訪問的目錄歷史，使用者只需輸入對應的數字即可快速跳轉，比 Bash 的 `cd -` 更為強大[^129]。

## 腳本對決：Bash vs. Zsh

儘管 Zsh 在互動式使用上優勢明顯，但在撰寫 shell script 時，開發者必須謹慎對待其與 Bash 的差異，特別是當腳本需要在不同環境中執行時[^11][^163]。

### 相容性與可攜性

首先，一個重要的觀念是，即使您的預設互動 shell 是 Zsh，任何以 `#!/bin/bash` 作為 shebang 的腳本仍然會由 Bash 直譯器執行[^5][^59][^193]。這確保了現有的 Bash 腳本在切換到 Zsh 後仍能正常運作[^5]。

然而，Zsh 的語法並非與 Bash 100% 相容[^7][^43]。雖然 Zsh 提供了模擬模式（`emulate sh`）來提高與 Bourne shell 的相容性，但這並非預設行為[^57][^59]。因此，對於需要最大可攜性的腳本（例如，部署在多種 Linux 伺服器或嵌入式系統上），堅持使用 Bash 或 POSIX sh 語法是更安全的選擇[^36][^114]。

### 關鍵語法與功能差異

以下表格總結了 Zsh 和 Bash 在腳本編寫方面的一些主要不同之處：

| 特性 | Bash | Zsh | 說明 |
| :--- | :--- | :--- | :--- |
| **陣列索引** | 從 0 開始 (`array[^0]`) | 預設從 1 開始 (`array[^1]`) | Zsh 的行為更像某些程式語言，但可透過 `setopt KSH_ARRAYS` 改為 0-based 索引[^27][^109][^113]。 |
| **通配符擴展 (Globbing)** | 基本功能，需 `shopt -s extglob` 開啟擴展功能 | 極其強大，原生支援遞迴搜尋 (`**`) 和元資料篩選 (`*(.)`) | Zsh 的 globbing 功能強大到可以取代許多 `find` 指令的用法，使腳本更簡潔[^5][^14][^107]。 |
| **參數擴展** | 未加引號的變數會進行分詞和檔名擴展，易引發錯誤 | 未加引號的變數預設不會進行分詞和擴展，行為更安全可預測 | Zsh 的處理方式減少了因忘記加引號而導致的常見 shell 腳本錯誤[^5][^148][^164]。 |
| **浮點數運算** | 不支援，需借助 `bc` 等外部工具 | 原生支援浮點數運算 | 在需要進行數學計算的腳本中，Zsh 提供了更大的便利性[^31][^46][^113]。 |
| **關聯陣列** | v4.0+ 支援，鍵必須是字串 | 原生支援，功能更豐富，鍵可以是多種資料類型 | Zsh 的關聯陣列實現更為先進和靈活[^73][^143]。 |
| **提示字元自訂** | 使用反斜線轉義 (`\u`, `\h`) | 使用百分比轉義 (`%n`, `%m`)，支援右側提示字元 | 兩者的語法完全不同，Zsh 提供更靈活的提示字元自訂選項[^35][^80][^121]。 |

## 實務遷移與共存策略

對於決定採用 Zsh 的開發者，順利遷移並理解如何讓兩者共存是關鍵。

### 遷移設定檔

您不能簡單地將 `.bashrc` 或 `.bash_profile` 重新命名為 `.zshrc`[^59]。Zsh 使用不同的啟動檔案（主要是 `.zshrc` 和 `.zprofile`），且設定語法（如 `bindkey`、`setopt`）也與 Bash 不同[^35][^121]。您需要手動將原有的環境變數、別名和函式遷移到 `~/.zshrc` 中，並根據 Zsh 的語法進行調整[^50][^72]。

### 撰寫可攜式腳本

當您撰寫的腳本需要在多個系統上執行時，最佳實踐是在 shebang 中明確指定 Bash 直譯器，推薦使用 `#!/usr/bin/env bash` 以增加可攜性[^59]。在腳本內容中，應盡量遵循 POSIX 標準或 Bash 的通用語法，避免使用 Zsh 獨有的特性[^5][^36]。

### 善用 Zsh 獨有功能

當您為確定使用 Zsh 的環境（如個人 Mac 或特定伺服器）撰寫腳本時，則應充分利用 Zsh 的強大功能。使用遞迴 globbing、改進的陣列處理和參數擴展，可以讓您的腳本更為簡潔、優雅且不易出錯[^25][^182]。

### Oh My Zsh 的影響

「Oh My Zsh」極大地降低了 Zsh 的入門門檻，但其龐大的體積有時會導致 shell 啟動速度變慢[^7][^62]。許多進階使用者在熟悉 Zsh 後，可能會轉向更輕量的插件管理器（如 `zinit`, `antibody`）或完全手動管理自己的設定檔，以在功能和性能之間取得平衡[^7][^104]。

## 結論

在 Bash 與 Zsh 的選擇中，並不存在絕對的優劣，而是取決於具體的使用場景和個人偏好[^114]。

對於開發者而言，**Zsh 是互動式使用的不二之選**。它透過進階的自動補全、強大的插件生態和高度可自訂性，顯著提升了命令列的生產力與使用樂趣[^16][^51]。

另一方面，**Bash 依然是撰寫可攜式腳本的黃金標準**。其無處不在的特性和穩定的行為使其成為跨平台自動化和系統管理任務最可靠的選擇[^36][^117][^163]。

最理想的策略是兩者兼顧：將 Zsh 設定為您的預設登入 shell，享受其帶來的高效互動體驗；同時，在撰寫需要部署到多個環境的腳本時，繼續使用 Bash，並在 shebang 中明確指定 `#!/bin/bash`[^146]。如此一來，開發者便能同時享有 Zsh 的現代化便利與 Bash 的穩定可攜性。

{% chat(speaker="jim") %}
請你以實際的程式碼詳細展示 Zsh 的 通配符擴展 (Globbing)、關聯陣列、浮點數運算 的使用方法
{% end %}

{% chat(speaker="felo") %}
我將透過實際的程式碼範例來詳細展示 Zsh 的通配符擴展 (Globbing)、關聯陣列和浮點數運算功能。
{% end %}

## Zsh 通配符擴展 (Globbing)

Zsh 的通配符擴展功能遠比 Bash 強大，提供了豐富的模式匹配和檔案篩選功能。

## 基本通配符擴展

```zsh
#!/bin/zsh

# 基本通配符
echo "=== 基本通配符 ==="
echo "所有 .txt 檔案："
ls *.txt

echo "所有以 test 開頭的檔案："
ls test*

echo "單一字元匹配："
ls file?.txt
```

## 遞迴目錄搜尋

```zsh
#!/bin/zsh

# 遞迴搜尋所有子目錄中的檔案
echo "=== 遞迴搜尋 ==="
echo "所有子目錄中的 .log 檔案："
ls **/*.log

echo "所有子目錄中的 Python 檔案："
ls **/*.py

# 限制遞迴深度
echo "最多搜尋 2 層子目錄的 .js 檔案："
ls **2/*.js
```

## 進階通配符模式

```zsh
#!/bin/zsh

# 啟用擴展通配符
setopt EXTENDED_GLOB

echo "=== 進階通配符模式 ==="

# 排除特定檔案
echo "所有檔案，但排除 .git 目錄："
ls ^.git*

# 匹配多個模式之一
echo "所有 .txt 或 .md 檔案："
ls *.(txt|md)

# 匹配特定長度的檔名
echo "檔名恰好 4 個字元的檔案："
ls ????

# 數字範圍匹配
echo "檔名包含 1-5 數字的檔案："
ls *<1-5>*

# 檔案類型篩選
echo "只顯示目錄："
ls *(/)

echo "只顯示一般檔案："
ls *(.)

echo "只顯示可執行檔案："
ls *(*)

echo "只顯示符號連結："
ls *(@)
```

## 檔案屬性篩選

```zsh
#!/bin/zsh
setopt EXTENDED_GLOB

echo "=== 檔案屬性篩選 ==="

# 按檔案大小篩選
echo "大於 1MB 的檔案："
ls *(Lm+1)

echo "小於 100KB 的檔案："
ls *(Lk-100)

# 按修改時間篩選
echo "最近 7 天修改的檔案："
ls *(mh-168)  # 168 小時 = 7 天

echo "超過 30 天未修改的檔案："
ls *(mh+720)  # 720 小時 = 30 天

# 按權限篩選
echo "可讀寫的檔案："
ls *(r:u:w:u:)

# 組合條件
echo "大於 1MB 且最近修改的 .log 檔案："
ls **/*.log(Lm+1mh-24)
```

## Zsh 關聯陣列

Zsh 原生支援關聯陣列（類似其他語言的字典或雜湊表），功能比 Bash 更強大。

## 基本關聯陣列操作

```zsh
#!/bin/zsh

echo "=== 基本關聯陣列操作 ==="

# 宣告關聯陣列
typeset -A user_info
typeset -A server_config

# 設定鍵值對
user_info[name]="張三"
user_info[age]=30
user_info[department]="資訊部"
user_info[email]="zhang.san@company.com"

# 批次設定
server_config=(
    [host]="192.168.1.100"
    [port]=8080
    [protocol]="https"
    [timeout]=30
    [max_connections]=1000
)

# 讀取值
echo "使用者姓名: ${user_info[name]}"
echo "使用者年齡: ${user_info[age]}"
echo "伺服器主機: ${server_config[host]}"
echo "伺服器埠號: ${server_config[port]}"
```

## 關聯陣列的進階操作

```zsh
#!/bin/zsh

typeset -A database_config
database_config=(
    [mysql_host]="db1.example.com"
    [mysql_port]=3306
    [mysql_user]="app_user"
    [mysql_password]="secret123"
    [redis_host]="cache.example.com"
    [redis_port]=6379
    [postgres_host]="pg.example.com"
    [postgres_port]=5432
)

echo "=== 關聯陣列進階操作 ==="

# 檢查鍵是否存在
if [[ -n ${database_config[mysql_host]} ]]; then
    echo "MySQL 主機已設定: ${database_config[mysql_host]}"
fi

# 取得所有鍵
echo "所有設定項目:"
for key in ${(k)database_config}; do
    echo "  $key"
done

# 取得所有值
echo "所有設定值:"
for value in ${(v)database_config}; do
    echo "  $value"
done

# 同時遍歷鍵和值
echo "完整設定:"
for key value in ${(kv)database_config}; do
    echo "  $key = $value"
done

# 陣列大小
echo "設定項目總數: ${#database_config}"

# 刪除特定鍵
unset "database_config[mysql_password]"
echo "刪除密碼後的設定項目數: ${#database_config}"
```

## 實用的關聯陣列範例

```zsh
#!/bin/zsh

# 建立一個簡單的翻譯字典
typeset -A translations
translations=(
    [hello]="你好"
    [goodbye]="再見"
    [thank_you]="謝謝"
    [please]="請"
    [yes]="是"
    [no]="否"
    [computer]="電腦"
    [software]="軟體"
    [engineer]="工程師"
)

# 翻譯函式
translate() {
    local word=$1
    if [[ -n ${translations[$word]} ]]; then
        echo "${translations[$word]}"
    else
        echo "找不到 '$word' 的翻譯"
    fi
}

echo "=== 翻譯字典範例 ==="
echo "hello -> $(translate hello)"
echo "computer -> $(translate computer)"
echo "unknown -> $(translate unknown)"

# 計數器範例
typeset -A word_count
text="apple banana apple orange banana apple"

echo "=== 單字計數範例 ==="
for word in ${=text}; do
    ((word_count[$word]++))
done

for word count in ${(kv)word_count}; do
    echo "$word: $count 次"
done
```

## Zsh 浮點數運算

Zsh 原生支援浮點數運算，這是相較於 Bash 的一大優勢[^1001][^1003]。

## 基本浮點數運算

```zsh
#!/bin/zsh

echo "=== 基本浮點數運算 ==="

# 宣告浮點數變數
float pi=3.14159
float radius=5.5
integer count=10

echo "圓周率: $pi"
echo "半徑: $radius"

# 基本運算
float area=$((pi * radius * radius))
float circumference=$((2 * pi * radius))

echo "圓面積: $area"
echo "圓周長: $circumference"

# 混合整數和浮點數運算
float average=$((25.5 + 30.2 + 28.7) / 3)
echo "平均值: $average"
```

## 進階浮點數運算

```zsh
#!/bin/zsh

# 載入數學函式庫
zmodload zsh/mathfunc

echo "=== 進階浮點數運算 ==="

float x=2.5
float y=3.7

# 基本數學函式
echo "x = $x, y = $y"
echo "x 的平方根: $(sqrt(x))"
echo "x 的 y 次方: $(pow(x, y))"
echo "x 的自然對數: $(log(x))"
echo "x 的常用對數: $(log10(x))"
echo "e 的 x 次方: $(exp(x))"

# 三角函式
float angle=1.57079632679  # π/2
echo "角度 (弧度): $angle"
echo "sin($angle): $(sin(angle))"
echo "cos($angle): $(cos(angle))"
echo "tan($angle): $(tan(angle))"

# 反三角函式
echo "arcsin(0.5): $(asin(0.5))"
echo "arccos(0.5): $(acos(0.5))"
echo "arctan(1): $(atan(1))"
```

## 實用的浮點數運算範例

```zsh
#!/bin/zsh
zmodload zsh/mathfunc

echo "=== 實用浮點數運算範例 ==="

# 溫度轉換函式
celsius_to_fahrenheit() {
    float celsius=$1
    float fahrenheit=$((celsius * 9.0 / 5.0 + 32.0))
    echo "$celsius°C = $fahrenheit°F"
}

fahrenheit_to_celsius() {
    float fahrenheit=$1
    float celsius=$(((fahrenheit - 32.0) * 5.0 / 9.0))
    echo "$fahrenheit°F = $celsius°C"
}

echo "溫度轉換:"
celsius_to_fahrenheit 25.5
fahrenheit_to_celsius 77.9

# 複利計算
compound_interest() {
    float principal=$1    # 本金
    float rate=$2        # 年利率 (小數)
    integer years=$3     # 年數
    
    float amount=$((principal * pow(1 + rate, years)))
    float interest=$((amount - principal))
    
    echo "本金: $principal"
    echo "年利率: $((rate * 100))%"
    echo "年數: $years"
    echo "最終金額: $amount"
    echo "利息: $interest"
}

echo ""
echo "複利計算 (本金 10000, 年利率 5%, 10 年):"
compound_interest 10000 0.05 10

# 統計計算
calculate_statistics() {
    local numbers=($@)
    float sum=0
    float sum_squares=0
    integer count=${#numbers}
    
    # 計算總和和平方和
    for num in $numbers; do
        sum=$((sum + num))
        sum_squares=$((sum_squares + num * num))
    done
    
    float mean=$((sum / count))
    float variance=$((sum_squares / count - mean * mean))
    float std_dev=$(sqrt(variance))
    
    echo "數據: ${numbers[*]}"
    echo "個數: $count"
    echo "總和: $sum"
    echo "平均值: $mean"
    echo "變異數: $variance"
    echo "標準差: $std_dev"
}

echo ""
echo "統計計算:"
calculate_statistics 12.5 15.3 18.7 14.2 16.8 13.9 17.1
```

## 浮點數比較和條件判斷

```zsh
#!/bin/zsh

echo "=== 浮點數比較 ==="

float price1=25.99
float price2=26.50
float tolerance=0.01

# 浮點數比較
if (( price1 < price2 )); then
    echo "$price1 < $price2"
fi

if (( price1 > 25.0 )); then
    echo "$price1 > 25.0"
fi

# 浮點數相等比較 (考慮精度問題)
float diff=$((price2 - price1))
if (( diff < tolerance && diff > -tolerance )); then
    echo "價格相近 (差異小於 $tolerance)"
else
    echo "價格差異: $diff"
fi

# 範圍檢查
check_range() {
    float value=$1
    float min=$2
    float max=$3
    
    if (( value >= min && value <= max )); then
        echo "$value 在範圍 [$min, $max] 內"
        return 0
    else
        echo "$value 不在範圍 [$min, $max] 內"
        return 1
    fi
}

echo ""
echo "範圍檢查:"
check_range 15.5 10.0 20.0
check_range 25.0 10.0 20.0
```

這些範例展示了 Zsh 在通配符擴展、關聯陣列和浮點數運算方面的強大功能。相較於 Bash，Zsh 提供了更直觀、更強大的語法，特別是在處理複雜的檔案操作、資料結構和數值計算時，能夠大幅提升腳本的可讀性和功能性[^1001][^1003]。

[^1]: [考虑一下把ZSH 设置成默认shell 怎么样？ - Reddit](https://www.reddit.com/r/linuxquestions/comments/n4p1ss/how_about_set_zsh_as_default_shell/?tl=zh-hans)
[^3]: [Zsh和Bash有啥区别？为什么要指定用户Shell？一文搞懂](https://blog.csdn.net/2501_91107759/article/details/147372795)
[^4]: [Zsh vs. Bash不完全对比解析,zsh是一种更强大的被成为"终极 ...](https://blog.csdn.net/weixin_34314962/article/details/86358466)
[^5]: [What are the practical differences between Bash and Zsh?](https://apple.stackexchange.com/questions/361870/what-are-the-practical-differences-between-bash-and-zsh)
[^7]: [为什么使用zsh 以及为什么这样使用 - Bigshans' Blog](https://bigshans.github.io/post/why-and-how-use-zsh-this-way/)
[^9]: [面向初学者的Linux Shell——解释Bash、Zsh 和Fish](https://www.freecodecamp.org/chinese/news/linux-shells-explained/)
[^10]: [你们最喜欢哪个shell，为什么？ : r/archlinux - Reddit](https://www.reddit.com/r/archlinux/comments/yayeun/whats_your_preferred_shell_why/?tl=zh-hans)
[^11]: [Why don't I use ZSH ? | Medium - DevProgramming](https://devprogramming.medium.com/why-i-dont-use-zsh-632eb8e7e4a4)
[^13]: [什么是zsh？我是否应该使用zsh - POLOXUE's BLOG](https://www.poloxue.com/posts/2023-09-16-what-how-to-use-zsh/)
[^14]: [Zsh vs Bash: Key Differences, Features, and Which to Choose](https://codeparrot.ai/blogs/zsh-vs-bash-key-differences-features-and-which-one-to-choose)
[^16]: [Zsh vs. Bash | Better Stack Community](https://betterstack.com/community/guides/linux/zsh-vs-bash/)
[^19]: [Shell的唯一选择--zsh 原创 - CSDN博客](https://blog.csdn.net/wy7651421/article/details/104091351)
[^23]: [为什么说zsh 是shell 中的极品？ - 韦易笑的回答- 知乎](https://www.zhihu.com/question/21418449/answer/300879747)
[^25]: [Zsh 开发指南（第一篇变量和语句） - 知乎专栏](https://zhuanlan.zhihu.com/p/28887846)
[^26]: [面向初学者的Linux Shell——解释Bash、Zsh 和Fish](https://www.freecodecamp.org/chinese/news/linux-shells-explained/)
[^27]: [What are the practical differences between Bash and Zsh?](https://apple.stackexchange.com/questions/361870/what-are-the-practical-differences-between-bash-and-zsh)
[^31]: [Is it recommended to use zsh instead of bash scripts? [closed]](https://unix.stackexchange.com/questions/69001/is-it-recommended-to-use-zsh-instead-of-bash-scripts)
[^33]: [Shell、Bash、Zsh这都是啥啊原创 - CSDN博客](https://blog.csdn.net/u011291072/article/details/122782942)
[^35]: [Bash Scripting - Difference between Zsh and Bash](https://www.geeksforgeeks.org/linux-unix/bash-scripting-difference-between-zsh-and-bash/)
[^36]: [What's the Difference Between Bash, SH, and ZSH? - Medium](https://medium.com/@fulton_shaun/whats-the-difference-between-bash-sh-and-zsh-e10e5c55a574)
[^38]: [Those of you who prefer ZSH to BASH, why? : r/linuxquestions](https://www.reddit.com/r/linuxquestions/comments/p50jvl/those_of_you_who_prefer_zsh_to_bash_why/)
[^43]: [whats the difference between zsh and bash? - Reddit](https://www.reddit.com/r/learnprogramming/comments/qw5bw3/whats_the_difference_between_zsh_and_bash/)
[^45]: [What Is Bash? Features, Major Concepts, Commands, & More](https://unstop.com/blog/what-is-bash)
[^46]: [What are the advantages of zsh for scripting? - Reddit](https://www.reddit.com/r/zsh/comments/kwio6m/what_are_the_advantages_of_zsh_for_scripting/)
[^49]: [Z shell 深度解析：演进、特性与实践原创 - CSDN博客](https://blog.csdn.net/i042416/article/details/147762255)
[^50]: [Mac 从Bash切换到Zsh的注意事项原创 - CSDN博客](https://blog.csdn.net/guo_zhen_qian/article/details/52682790)
[^51]: [Zsh and Bash - Refine dev](https://refine.dev/blog/zsh-vs-bash/)
[^57]: [Zsh和Bash，究竟有何不同坑很深-转载自 - 博客园](https://www.cnblogs.com/jadecard/articles/10147692.html)
[^59]: [Are all bash scripts compatible with `zsh`?](https://unix.stackexchange.com/questions/38172/are-all-bash-scripts-compatible-with-zsh)
[^62]: [Zsh vs. Bash | Better Stack Community](https://betterstack.com/community/guides/linux/zsh-vs-bash/)
[^67]: [Help me understand the possibilities of Bash (best uses, etc)](https://www.reddit.com/r/bash/comments/136rgkx/help_me_understand_the_possibilities_of_bash_best/)
[^71]: [Z shell - 维基百科，自由的百科全书](https://zh.wikipedia.org/zh-hans/Z_shell)
[^72]: [bash 和zsh 有什么区别 - 稀土掘金](https://juejin.cn/post/7376999346755092495)
[^73]: [Zsh vs. Bash Scripting. What's the Difference? - MakeUseOf](https://www.makeuseof.com/zsh-bash-scripting-difference/)
[^75]: [Z shell - Wikipedia](https://en.wikipedia.org/wiki/Z_shell)
[^80]: [A brief difference between zsh and bash - DEV Community](https://dev.to/jasmin/a-brief-difference-between-zsh-and-bash-5ebp)
[^89]: [My favourite Zsh features - JoeJag](https://code.joejag.com/2014/why-zsh.html)
[^94]: [What is the difference between the bash shell and the Z shell?](https://www.quora.com/What-is-the-difference-between-the-bash-shell-and-the-Z-shell)
[^104]: [为什么使用zsh 以及为什么这样使用 - Bigshans' Blog](https://bigshans.github.io/post/why-and-how-use-zsh-this-way/)
[^106]: [Those of you who prefer ZSH to BASH, why? - Reddit](https://www.reddit.com/r/linuxquestions/comments/p50jvl/those_of_you_who_prefer_zsh_to_bash_why/)
[^107]: [Zsh vs Bash: Key Differences, Features, and Which to Choose](https://codeparrot.ai/blogs/zsh-vs-bash-key-differences-features-and-which-one-to-choose)
[^108]: [为什么说zsh 是shell 中的极品？ - 知乎](https://www.zhihu.com/question/21418449)
[^109]: [zsh 和bash 的簡單區別 - CodeLove 論壇](https://codelove.tw/@tony/post/YqZKJx)
[^110]: [What is ZSH, and Why Should You Use It Instead of Bash?](https://www.howtogeek.com/362409/what-is-zsh-and-why-should-you-use-it-instead-of-bash/)
[^113]: [Bash与Zsh对比原创 - CSDN博客](https://blog.csdn.net/u010711495/article/details/146769108)
[^114]: [Zsh and Bash - Refine dev](https://refine.dev/blog/zsh-vs-bash/)
[^117]: [揭秘Bash 与Zsh：你的Mac 选择了哪一种强大的命令行Shell？](https://juejin.cn/post/7330295665866080308)
[^119]: [终端环境：zsh 、oh-my-zsh、提示主题与7 效率插件](https://www.poloxue.com/posts/2023-10-16-zsh-themes-and-plugins/)
[^121]: [What are the practical differences between Bash and Zsh?](https://apple.stackexchange.com/questions/361870/what-are-the-practical-differences-between-bash-and-zsh)
[^124]: [Why don't I use ZSH ? | Medium - DevProgramming](https://devprogramming.medium.com/why-i-dont-use-zsh-632eb8e7e4a4)
[^129]: [mac 装了oh my zsh 后比用bash 具体好在哪儿？ - 知乎](https://www.zhihu.com/question/29977255)
[^143]: [Zsh vs. Bash Scripting. What's the Difference? - MakeUseOf](https://www.makeuseof.com/zsh-bash-scripting-difference/)
[^146]: [Is there any reason to use bash over zsh? [closed] - Server Fault](https://serverfault.com/questions/93388/is-there-any-reason-to-use-bash-over-zsh)
[^148]: [What are the practical differences between Bash and Zsh?](https://apple.stackexchange.com/questions/361870/what-are-the-practical-differences-between-bash-and-zsh)
[^162]: [Zsh vs Bash: Key Differences, Features, and Which to Choose](https://codeparrot.ai/blogs/zsh-vs-bash-key-differences-features-and-which-one-to-choose)
[^163]: [Zsh vs. Bash Scripting. What's the Difference?](https://www.makeuseof.com/zsh-bash-scripting-difference/)
[^164]: [terminal - What are the practical differences between Bash and Zsh? - Ask Different](https://apple.stackexchange.com/questions/361870/what-are-the-practical-differences-between-bash-and-zsh)
[^182]: [Understanding the Unique Features of zsh Scripting - Packt](https://www.packtpub.com/en-nz/product/the-ultimate-linux-shell-scripting-guide-9781835463574/chapter/introduction-to-z-shell-scripting-22/section/understanding-the-unique-features-of-zsh-scripting-ch22lvl1sec56?srsltid=AfmBOopuOpg-6HEaOMNvQ3qubWaR7szz5pIvk2dcsD_GO8txedpV8tLn)
[^193]: [Are all bash scripts compatible with `zsh`? - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/38172/are-all-bash-scripts-compatible-with-zsh)
[^1001]: [\[鐵人賽第12天\] 四則運算 - iT 邦幫忙](https://ithelp.ithome.com.tw/articles/10187627?sc=pt)
[^1003]: [07 数值计算— ZshGuide latest 文档](https://zshguide.readthedocs.io/zh/latest/content/ch07.html)
