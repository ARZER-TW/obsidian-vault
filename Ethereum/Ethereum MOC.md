---
tags: [ethereum, MOC, index, blockchain]
aliases: [以太坊知識地圖, Ethereum Map of Content]
---

# Ethereum MOC

本筆記是整個 Ethereum 密碼學與協議知識庫的主索引。所有筆記按資料夾分組，依賴關係透過雙向連結追蹤。

> 新手起點：[[Ethereum 全景概覽]] -- 從這裡開始，了解 Ethereum 全貌和六條學習路徑

---

## 密碼學基礎

| 分類 | 筆記 | 說明 |
|------|------|------|
| 雜湊函數 | [[雜湊函數概述]] | 雜湊函數的定義、性質與安全模型 |
| | [[Keccak-256]] | Ethereum 核心雜湊函數 |
| | [[SHA-256]] | Bitcoin 與部分 Ethereum 元件使用 |
| 橢圓曲線 | [[橢圓曲線密碼學]] | ECC 數學基礎 |
| | [[secp256k1]] | Ethereum 簽章曲線 |
| | [[BLS12-381]] | Beacon Chain 使用的配對友好曲線 |
| 數位簽章 | [[數位簽章概述]] | 簽章方案的定義與安全性 |
| | [[ECDSA]] | Ethereum 交易簽章演算法 |
| | [[ECRECOVER]] | 從簽章反推公鑰 |
| | [[BLS Signatures]] | Beacon Chain 聚合簽章 |
| 其他 | [[公鑰密碼學]] | 非對稱加密基礎 |
| | [[CSPRNG]] | 密碼學安全隨機數生成器 |

## Ethereum 資料結構

| 分類 | 筆記 | 說明 |
|------|------|------|
| 編碼 | [[RLP 編碼]] | 遞迴長度前綴編碼 |
| | [[SSZ 編碼]] | Simple Serialize（Beacon Chain） |
| | [[ABI 編碼]] | 合約呼叫資料編碼 |
| Merkle 結構 | [[Merkle Tree]] | 基本 Merkle 樹 |
| | [[Merkle Patricia Trie]] | Ethereum 狀態儲存核心結構 |
| | [[State Trie]] | 全域帳戶狀態樹 |
| | [[Storage Trie]] | 合約儲存樹 |
| | [[Transaction Trie]] | 區塊內交易樹 |
| | [[Receipt Trie]] | 交易回執樹 |
| 過濾器 | [[Bloom Filter]] | 日誌事件快速查詢 |

## 帳戶與交易

| 筆記 | 說明 |
|------|------|
| [[EOA]] | 外部擁有帳戶 |
| [[合約帳戶]] | 智能合約帳戶 |
| [[地址推導]] | 從公鑰到地址的推導流程 |
| [[EIP-55 地址校驗]] | 混合大小寫校驗和 |
| [[Nonce]] | 交易序號與重放防護 |
| [[Gas]] | 計算資源計量 |
| [[EIP-155 重放保護]] | Chain ID 簽章保護 |
| [[EIP-1559 費用市場]] | 基礎費用與優先費用 |

## 交易流程

| 筆記 | 說明 |
|------|------|
| [[交易生命週期]] | **Hub** - 交易從建立到最終確認的完整流程 |
| [[密鑰生成與帳戶創建]] | 私鑰生成、公鑰推導、地址計算 |
| [[交易構建]] | 交易欄位組裝與 RLP 編碼 |
| [[交易簽名]] | ECDSA 簽章流程與 v/r/s 值 |
| [[交易廣播與驗證]] | P2P 網路傳播與節點驗證 |
| [[記憶池]] | Mempool 管理與交易排序 |
| [[區塊生產]] | Proposer 選擇與區塊組裝 |
| [[共識與最終性]] | Casper FFG 投票與最終確認 |
| [[狀態轉換]] | EVM 執行與狀態樹更新 |

## 區塊與共識

| 筆記 | 說明 |
|------|------|
| [[共識入門]] | 為什麼需要共識、PoW vs PoS、CL+EL 架構 |
| [[Beacon Chain]] | PoS 共識層 |
| [[Slot 與 Epoch]] | 時間模型：12 秒 slot、32 slot epoch |
| [[PoS 與質押入門]] | 質押經濟安全、32 ETH 門檻、Proposer vs Attester |
| [[Validators]] | 驗證者角色與生命週期 |
| [[區塊結構]] | 區塊的組成元素（雙層架構） |
| [[區塊 Header]] | Header 各欄位定義 |
| [[RANDAO]] | 鏈上隨機數 |
| [[Attestation]] | 驗證者投票機制 |
| [[LMD GHOST]] | Fork choice rule |
| [[Casper FFG]] | Finality gadget |
| [[Gasper]] | Casper FFG + LMD GHOST 統一共識協議 |
| [[Slashing]] | 懲罰機制 |
| [[Ethash]] | PoW 時代挖礦演算法（歷史參考） |

## 進階主題

| 筆記 | 說明 |
|------|------|
| [[Ethereum 擴展性路線圖]] | 統一敘事：L1 瓶頸與三大擴展方向 |
| [[多項式承諾入門]] | 承諾的三步驟、KZG vs IPA 比較 |
| [[橢圓曲線配對導論]] | Bilinear pairing 直覺與 BLS12-381 三群 |
| [[EIP-4844 Proto-Danksharding]] | Blob 交易與資料可用性 |
| [[KZG Commitments]] | 多項式承諾方案 |
| [[Verkle Trees]] | 下一代狀態樹結構 |
| [[Precompiled Contracts]] | 預編譯合約與密碼學操作 |
| [[zkSNARKs 支援]] | 零知識證明在 Ethereum 的支援 |

---

## 協議升級時間線

### 已完成

| 升級 | 日期 | 重點 EIP |
|------|------|---------|
| Dencun | 2024/3 | EIP-4844（Proto-Danksharding、blob transaction） |
| **Pectra** | **2025/5/7** | EIP-2537（[[BLS12-381]] precompile）、EIP-7691（blob target 6 / max 9） |
| **Fusaka** | **2025/12/3** | EIP-7594（PeerDAS）、EIP-7892（BPO 機制）、EIP-7918（blob 費用掛鉤）、EIP-7951（secp256r1 precompile） |

**Fusaka 後的 BPO 調整：**
- BPO1（2025/12/9）：blob target 10, max 15
- BPO2（2026/1/7）：blob target 14, max 21

### Pectra 重點

- **EIP-2537**：[[BLS12-381]] 曲線 9 個操作的 [[Precompiled Contracts]]（0x0B-0x13），使 EVM 原生支援 128-bit 安全的配對運算
- **EIP-7691**：[[EIP-4844 Proto-Danksharding]] blob 容量翻倍（target 3->6, max 6->9）

### Fusaka 重點

- **EIP-7594 PeerDAS**：資料可用性取樣，節點不需下載完整 blob，透過 [[KZG Commitments]] 取樣驗證
- **EIP-7892 BPO**：Blob Parameter Only 機制，允許在硬分叉間調整 blob 參數
- **EIP-7918**：Blob 費用與 L1 gas 掛鉤
- **EIP-7951**：[[secp256k1|secp256r1]] 簽名預編譯（passkey / FIDO2 / WebAuthn 支援）

### 路線圖（規劃中）

| 升級 | 預計時間 | 可能內容 |
|------|---------|---------|
| Glamsterdam | 2026 H1 | EOF（EVM Object Format）可能回歸 |
| Hegota | 2026 H2 | [[Verkle Trees]]（狀態樹結構重構） |

**EOF（EVM Object Format）**：原本考慮納入 Pectra 或 Fusaka，最終都未入選。EOF 重構 EVM bytecode 的結構（分離 code/data section、靜態跳轉等），可能在 Glamsterdam 升級回歸。

---

## 學習路徑建議

**推薦起點**：先讀 [[Ethereum 全景概覽]]，再選擇以下路徑。

**交易流程（推薦第一條路徑）**：[[交易生命週期]] → [[密鑰生成與帳戶創建]] → [[交易構建]] → [[交易簽名]] → [[交易廣播與驗證]] → [[記憶池]] → [[區塊生產]] → [[共識與最終性]] → [[狀態轉換]]

**密碼學路徑**：[[雜湊函數概述]] → [[SHA-256]] → [[Keccak-256]] → [[橢圓曲線密碼學]] → [[secp256k1]] → [[數位簽章概述]] → [[ECDSA]] → [[ECRECOVER]] → [[BLS12-381]] → [[BLS Signatures]]

**資料結構路徑**：[[RLP 編碼]] → [[SSZ 編碼]] → [[Merkle Tree]] → [[Merkle Patricia Trie]] → [[State Trie]] → [[Storage Trie]] → [[Transaction Trie]] → [[Receipt Trie]] → [[Bloom Filter]]

**共識路徑**：[[共識入門]] → [[Beacon Chain]] → [[Slot 與 Epoch]] → [[PoS 與質押入門]] → [[Validators]] → [[區塊結構]] → [[RANDAO]] → [[Attestation]] → [[LMD GHOST]] → [[Casper FFG]] → [[Gasper]] → [[Slashing]]

**進階路徑**：[[Ethereum 擴展性路線圖]] → [[多項式承諾入門]] → [[橢圓曲線配對導論]] → [[EIP-4844 Proto-Danksharding]] → [[KZG Commitments]] → [[Verkle Trees]] → [[Precompiled Contracts]] → [[zkSNARKs 支援]]
