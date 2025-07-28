+++
title = "Rust 程式碼覆蓋率工具大比拼：cargo llvm-cov 與 cargo-tarpaulin"
description = "深入比較 cargo llvm-cov 與 cargo-tarpaulin 的技術特色、平台支援與覆蓋率準確性。解析兩者的適用情境與選擇建議。想知道哪個工具更適合你的 Rust 專案嗎？"
date = "2025-06-01T00:00:00Z"
updated = "2025-07-28T10:14:06Z"
draft = false

[taxonomies]
tags = [ "Rust" ]
providers = [ "Felo Search" ]
licenses = [ "GFDL 1.3" ]

[extra]
withAI = "<https://felo.ai/search/5asxbGvRnbJgTqQxmgj3ZF?invite=dOLYGeJyZJqVX>"
+++
{% chat(speaker="jim") %}
`cargo llvm-cov` vs `cargo-tarpaulin`，請你介紹一下它們的特色和區別，只選一個的話要如何選擇
{% end %}

{% chat(speaker="felo") %}
根據搜尋結果，我將詳細分析相關問題。
{% end %}

`cargo llvm-cov` 和 `cargo-tarpaulin` 都是 Rust 社群中用於衡量程式碼覆蓋率的重要工具。`cargo llvm-cov` 利用 LLVM 的原始碼級別覆蓋率工具，提供精確的行覆蓋率和區域覆蓋率，並且對程序化巨集（proc-macros）和文件測試（doc tests）有良好支援 [^3][^6][^29][^95]。相較之下，`cargo-tarpaulin` 是一個歷史較久的工具，名稱源於覆蓋貨物的防水布，傳統上在 Linux x86_64 環境中運作良好，並致力於提供分支覆蓋等功能 [^2][^28][^39][^40][^58]。近年來，`cargo-llvm-cov` 因其報告的準確性和更現代化的底層技術而受到更多關注 [^5][^7][^15][^17]。

## `cargo llvm-cov`

`cargo llvm-cov` 是一個 Cargo 子命令，它封裝了 `rustc` 編譯器的 `-C instrument-coverage` 選項，以便捷地使用 LLVM 的原始碼基礎程式碼覆蓋率功能 [^1][^12][^25][^55]。

### 特色

* **精準的覆蓋率數據**：利用 LLVM 的能力，可以產生非常精確的覆蓋率報告，包括行覆蓋率和區域覆蓋率（region coverage）[^3][^6][^29][^95]。區域覆蓋率能夠更細緻地顯示程式碼中特定區塊（例如 `?` 運算子導致的提早回傳）是否被執行 [^88]。
    ![LLVM 原始碼級別覆蓋率報告範例[^88]](https://storage.googleapis.com/zenn-user-upload/686cccd01669-20240504.png)
* **廣泛的支援性**：支援對一般的 `cargo test`、`cargo run`，以及程序化巨集（proc-macros）和文件測試（doc tests）進行覆蓋率收集 [^6][^29][^101]。它也能與 `cargo nextest` 等測試執行器良好整合 [^101]。
* **多種輸出格式**：可以產生文字界面的總結報告，也能生成詳細的 HTML 報告以便於瀏覽 [^83][^89]。同時支援 LCOV 格式輸出，方便與 VSCode 等編輯器的擴充功能（如 Coverage Gutters）整合 [^89]。
* **跨平台潛力**：由於基於 LLVM，理論上只要 LLVM 支援的平台，`cargo llvm-cov` 就能運作，這使得它在 macOS 等非 Linux 環境下也能良好工作 [^85][^90][^96]。
* **安裝與使用**：安裝通常透過 `cargo install cargo-llvm-cov` 和 `rustup component add llvm-tools-preview` 完成 [^88][^89]。自從 Rust 1.60.0 穩定化原始碼級別覆蓋率後，其使用變得更加便捷 [^88]。

### 限制

* 雖然 `cargo llvm-cov` 的報告中可能包含分支覆蓋率的欄位，但目前對於完整且穩定的分支覆蓋率支援仍有不足，這也是 `cargo-tarpaulin` 同樣面臨的挑戰 [^5][^7][^89]。

## `cargo-tarpaulin`

`cargo-tarpaulin` 是一個專為 Rust 專案設計的程式碼覆蓋率報告工具，其名稱取自一種用於覆蓋船上貨物的防水布 [^2][^39][^48]。

### 特色

* **穩定版 Rust 支援**：`cargo-tarpaulin` 可以在穩定的 Rust 工具鏈上運作 [^19][^99]。
* **多種報告格式**：能夠產生多種格式的報告，包括 HTML 和 LCOV，方便整合進 CI/CD 流程或與其他工具配合使用 [^74][^99]。
* **歷史悠久**：作為較早出現的 Rust 覆蓋率工具，擁有一定的社群基礎和使用案例 [^4][^22]。
* **分支覆蓋嘗試**：`cargo-tarpaulin` 一直致力於提供分支和條件覆蓋的功能 [^40]。

### 限制

* **平台依賴性**：傳統上，`cargo-tarpaulin` 主要支援 x86_64 架構的 Linux 系統，因其底層實現依賴於特定平台的除錯機制 [^28][^58]。
* **準確性疑慮**：部分使用者回饋其覆蓋率報告的準確性相較於 `cargo-llvm-cov` 可能稍遜一籌 [^15][^17]。
* **分支覆蓋率**：與 `cargo-llvm-cov` 類似，在提供全面且穩定的分支覆蓋率方面仍有進步空間 [^5][^7]。

## 主要區別比較

| 特性         | `cargo llvm-cov`                                  | `cargo-tarpaulin`                                      |
| ------------ | ------------------------------------------------- | ------------------------------------------------------ |
| **底層技術**   | LLVM 原始碼級別覆蓋率 (`-C instrument-coverage`) [^3][^25] | 傳統上使用 ptrace/DWARF 除錯資訊，亦探索 LLVM [^40][^88] |
| **覆蓋精度**   | 行覆蓋、區域覆蓋 (非常精確) [^3][^6][^88]                 | 行覆蓋，嘗試分支/條件覆蓋 [^40]                            |
| **Rust 版本** | 部分功能可能需 Nightly (但會自動安裝)，Rust 1.60 後更易用 [^19][^88] | 穩定版 Rust [^19][^99]                                     |
| **平台支援**   | 較廣 (基於 LLVM)，macOS 支援良好 [^85][^96]             | 傳統上主要為 Linux x86_64 [^28][^58]                       |
| **報告準確性** | 通常被認為更準確 [^15][^17]                               | 可能略遜 [^15][^17]                                        |
| **程序化巨集/文件測試** | 支援 [^6][^29][^101]                                   |  -                                                     |
| **HTML 報告** | 支援，易於閱讀 [^83][^89]                               | 支援 [^74]                                                |
| **分支覆蓋**   | 有限支援 [^5][^7][^89]                                  | 有限支援，持續開發中 [^5][^7][^40]                         |
| **近期趨勢**   | 下載量和社群討論度較高 [^82][^89]                         | 較為老牌的選擇 [^2][^4]                                   |

## 如何選擇？

如果需要在 `cargo llvm-cov` 和 `cargo-tarpaulin` 之間選擇一個，可以考慮以下幾點：

1. **追求最高的報告準確性和細緻度（如區域覆蓋）**：
    * 選擇 `cargo llvm-cov`。它利用 LLVM 的底層能力，能夠提供更精確的行覆蓋和區域覆蓋資訊，幫助發現一些行級別覆蓋可能遺漏的未測試路徑 [^3][^6][^88]。

2. **需要在穩定版 Rust 上運作且對平台有特定限制（如僅限 Linux x86_64 且不願切換工具鏈）**：
    * 可以考慮 `cargo-tarpaulin`，儘管 `cargo-llvm-cov` 在 Rust 1.60 穩定化相關功能後，對穩定版的支援也越來越好 [^19][^88]。

3. **需要更廣泛的平台支援（尤其是 macOS）**：
    * 選擇 `cargo llvm-cov`。其基於 LLVM 的特性使其具有更好的跨平台潛力 [^85][^96]。

4. **對程序化巨集和文件測試的覆蓋率有要求**：
    * 選擇 `cargo llvm-cov`，它對這兩者提供了良好的支援 [^6][^29][^101]。

5. **易用性和現代化整合**：
    * `cargo llvm-cov` 的整合度和報告美觀度（特別是 HTML 報告）受到許多開發者好評 [^5][^7][^83][^89]。

## 總體建議

目前來看，對於大多數 Rust 專案，如果追求準確性、詳細的覆蓋資訊（包含區域覆蓋）以及更好的平台支援，`cargo llvm-cov` 會是更優先的選擇 [^15][^17][^82][^88]。儘管分支覆蓋率在兩個工具中都還不是完美狀態，但 `cargo-llvm-cov` 在其他方面的優勢使其更具吸引力。如果專案有嚴格限制必須在特定舊環境的穩定版 Rust 上使用 `cargo-tarpaulin` 且無法升級，則可繼續使用後者，但應了解其可能的限制。

{% editor_note() %}
我後來在 [subx-cli](https://github.com/jim60105/subx-cli) 專案中採用 cargo llvm-cov
{% end %}

[^1]: [Cargo llvm-cov 项目常见问题解决方案 - CSDN博客](https://blog.csdn.net/gitblog_00454/article/details/144391876)
[^2]: [xd009642/tarpaulin: A code coverage tool for Rust projects](https://github.com/xd009642/tarpaulin)
[^3]: [探索Rust 的代码覆盖率新境界：cargo-llvm-cov 原创 - CSDN博客](https://blog.csdn.net/gitblog_00052/article/details/139017856)
[^4]: [153、掌握Rust代码覆盖率分析：提升软件质量的利器原创](https://blog.csdn.net/silenceallat/article/details/138377483)
[^5]: [今天才知道`cargo llvm-cov` 和`cargo nextest` : r/rust - Reddit](https://www.reddit.com/r/rust/comments/14v3ugw/til_about_cargo_llvmcov_and_cargo_nextest/?tl=zh-hans)
[^6]: [一个Cargo 子命令，方便地使用基于LLVM 源代码的覆盖率: r/rust](https://www.reddit.com/r/rust/comments/pbkpc7/cargollvmcov_cargo_subcommand_to_easily_use_llvm/?tl=zh-hans)
[^7]: [TIL about `cargo llvm-cov` and `cargo nextest` : r/rust - Reddit](https://www.reddit.com/r/rust/comments/14v3ugw/til_about_cargo_llvmcov_and_cargo_nextest/)
[^12]: [cargo-llvm-cov - crates.io: Rust Package Registry](https://crates.io/crates/cargo-llvm-cov/0.4.13)
[^15]: [給cargo-llvm-cov 鼓掌- 真的超好用的覆蓋率報告: r/rust - Reddit](https://www.reddit.com/r/rust/comments/v1j9fy/kudos_to_cargollvmcov_really_useful_coverage/?tl=zh-hant)
[^17]: [给cargo-llvm-cov 点个赞- 真的很有用的覆盖率报告: r/rust - Reddit](https://www.reddit.com/r/rust/comments/v1j9fy/kudos_to_cargollvmcov_really_useful_coverage/?tl=zh-hans)
[^19]: [Current state of code coverage in Rust - help](https://users.rust-lang.org/t/current-state-of-code-coverage-in-rust/63298)
[^22]: [你应该知道的7个Rust Cargo插件转载 - CSDN博客](https://blog.csdn.net/turingbooks/article/details/134903911)
[^25]: [GitHub - taiki-e/cargo-llvm-cov](https://github.com/taiki-e/cargo-llvm-cov)
[^28]: [Rust Crates可能的后门与Cargo - Rust精选](https://rustmagazine.github.io/rust_magazine_2021/chapter_11/crates-backdoor.html)
[^29]: [cargo-llvm-cov - crates.io: Rust Package Registry](https://crates.io/crates/cargo-llvm-cov/0.1.13)
[^39]: [Tarpaulin：Rust项目代码覆盖率分析利器 - CSDN博客](https://blog.csdn.net/gitblog_00896/article/details/141986370)
[^40]: [介紹tarpaulin，一個Rust 的程式碼覆蓋率工具 - Reddit](https://www.reddit.com/r/rust/comments/6ohd9y/introducing_tarpaulin_a_code_coverage_tool_for/?tl=zh-hant)
[^48]: [Test coverage with Tarpaulin - Rust Maven](https://rust.code-maven.com/test-coverage-with-tarpaulin)
[^55]: [Instrumentation-based Code Coverage - The rustc book](https://doc.rust-lang.org/rustc/instrument-coverage.html)
[^58]: [Rust Crates后门与Cargo - 稀土掘金](https://juejin.cn/post/7047775129438257165)
[^74]: [How I can get coverage for cargo test? - Stack Overflow](https://stackoverflow.com/questions/69491669/how-i-can-get-coverage-for-cargo-test)
[^82]: [【Rust】cargo-llvm-covでコードカバレッジを取得する - Zenn](https://zenn.dev/collabostyle/articles/f0654aa0feaba8)
[^83]: [Rustのcargo llvm covによる結果をhtmlで出力する](https://enu23456.hatenablog.com/entry/2022/11/04/210701)
[^85]: [\[code coverage\] cargo-llvm-cov vs tarpaulin #1195 - GitHub](https://github.com/rusqlite/rusqlite/issues/1195)
[^88]: [Rustの新しいコードカバレッジ/Source-based code coverage #テスト - Qiita](https://qiita.com/dalance/items/69e18fe300760f8d7de0)
[^89]: [【Rust】cargo-llvm-covでコードカバレッジを取得する](https://zenn.dev/collabostyle/articles/f0654aa0feaba8)
[^90]: [[code coverage] cargo-llvm-cov vs tarpaulin · Issue #1195 · rusqlite/rusqlite · GitHub](https://github.com/rusqlite/rusqlite/issues/1195)
[^95]: [探索Rust 的代码覆盖率新境界：cargo-llvm-cov 原创 - CSDN博客](https://blog.csdn.net/gitblog_00052/article/details/139017856)
[^96]: [\[code coverage\] cargo-llvm-cov vs tarpaulin #1195 - GitHub](https://github.com/rusqlite/rusqlite/issues/1195)
[^99]: [Current state of code coverage in Rust - help - The Rust Programming Language Forum](https://users.rust-lang.org/t/current-state-of-code-coverage-in-rust/63298)
[^101]: [探索 Rust 的代码覆盖率新境界：cargo-llvm-cov-CSDN博客](https://blog.csdn.net/gitblog_00052/article/details/139017856)
