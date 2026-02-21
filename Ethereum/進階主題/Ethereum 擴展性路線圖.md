---
tags: [ethereum, scaling, roadmap, rollup, danksharding, statelessness]
aliases: [Ethereum Scaling Roadmap, 擴展性路線圖, The Surge, The Verge]
---

# Ethereum 擴展性路線圖

## 為什麼需要擴展

Ethereum L1 每秒處理約 15 筆交易（TPS）。這個數字由 gas limit 決定：每個區塊約 30M gas，每 12 秒一個 slot，簡單轉帳消耗 21,000 gas。對比 Visa 的數千 TPS，L1 吞吐量遠遠不夠支撐全球規模的金融基礎設施。

直接提高 gas limit 會增加節點的計算和儲存負擔，降低去中心化程度。Ethereum 的策略不是單純拉高 L1 容量，而是沿三條互補方向同時推進。

## 方向一：資料可用性 -- The Surge

L2 Rollup 需要將交易資料發布到 L1，讓任何人都能驗證或重建 L2 狀態。2024 年以前，rollup 使用 calldata（每 byte 16 gas）發布資料，成本高昂。

**Proto-Danksharding（[[EIP-4844 Proto-Danksharding|EIP-4844]]，2024/3）**
引入 blob transaction：獨立定價、約 18 天後自動過期的資料空間。每個 blob 128KB，初始 target 3 / max 6 per block。上線後 L2 資料成本降低 10-100 倍。Blob 資料用 [[KZG Commitments]] 承諾，確保不可偽造。

**PeerDAS（EIP-7594，Fusaka 預計 2026）**
Data Availability Sampling 讓節點只需下載部分 blob 資料（取樣），不再需要下載全部。這打破了「容量成長 = 頻寬需求等比成長」的限制。搭配 BPO（Blob Parameter Only）機制，blob target 從 3 逐步提升到 14，max 從 6 到 21。

**Full Danksharding（遠期）**
最終目標是每區塊 64-128 個 blob（~8-16 MB 資料），配合 proposer-builder separation（PBS）和完整的 DAS 網路。屆時 L2 將擁有接近無限的廉價資料空間。

## 方向二：狀態管理 -- The Verge

Ethereum 全節點目前需要儲存超過 100 GB 的全局狀態。這個數字持續膨脹，墊高了運行節點的門檻。

**[[Verkle Trees]]（預計 Hegota，2026 H2）**
用 polynomial commitment（IPA）取代 Merkle hash，將 state proof 從 ~3.5 KB 壓縮到 ~150 bytes。更小的 proof 使得 stateless client 成為可能：驗證者不需要儲存完整狀態，只需要區塊附帶的 witness 就能驗證狀態轉換。

**Stateless Client（長期）**
Verkle Trees 上線後，輕量級驗證者只需要數十 MB 即可參與共識。這大幅降低硬體門檻，有利於去中心化。

## 方向三：L2 Rollup 生態

L1 專注提供安全性和資料可用性，執行層的高吞吐量交給 L2。

**Optimistic Rollup**（Optimism、Arbitrum）：假設交易有效，允許挑戰期內提交 fraud proof。延遲約 7 天。結構簡單，已經支撐大量 DeFi 應用。

**ZK Rollup**（zkSync、Scroll、Starknet）：用 [[zkSNARKs 支援|零知識證明]] 在 L1 上直接驗證 L2 狀態轉換的正確性。無挑戰期延遲，但 proof 生成計算密集。L1 透過 [[Precompiled Contracts]] 提供高效的橢圓曲線和 pairing 運算支援。

兩種方案都受益於 blob 降低資料成本。隨著 ZK 技術成熟，ZK Rollup 在最終性速度和安全模型上具有優勢。

## 升級時程

| 時間 | 升級 | 關鍵內容 |
|------|------|---------|
| 2024/3 | Dencun | EIP-4844：blob transaction、KZG commitment |
| 2025/5 | Pectra | EIP-7691：blob 擴容 target 6/max 9；EIP-7002/7251：validator 改進 |
| 預計 2026 | Fusaka | PeerDAS（DAS 取樣）、BPO（動態 blob 參數）、EOF |
| 預計 2026 H2 | Hegota | Verkle Trees、stateless client 初步支援 |
| 遠期 | -- | Full Danksharding（64-128 blobs/block）、完整 statelessness |

## 本系列文章導覽

以下是進階主題各篇文章在路線圖中的位置：

```
密碼學工具
  |-- 多項式承諾入門          <- 理解 KZG 和 Verkle 的前置概念
  |-- 橢圓曲線配對導論        <- 理解 KZG 驗證和 BLS 簽名的數學基礎

具體技術
  |-- KZG Commitments         <- Blob 驗證的核心（The Surge）
  |-- EIP-4844                <- 資料可用性方案（The Surge）
  |-- Verkle Trees            <- 狀態管理方案（The Verge）
  |-- zkSNARKs 支援           <- L2 ZK Rollup 的證明系統
  |-- Precompiled Contracts   <- 底層密碼學加速
```

建議閱讀順序：先讀 [[多項式承諾入門]] 和 [[橢圓曲線配對導論]] 建立工具概念，再進入 [[KZG Commitments]] -> [[EIP-4844 Proto-Danksharding]] -> [[Verkle Trees]]。

## 相關概念

- [[EIP-4844 Proto-Danksharding]] - Blob transaction 的具體設計
- [[KZG Commitments]] - Blob 資料的 polynomial commitment
- [[Verkle Trees]] - 取代 Merkle Patricia Trie 的新資料結構
- [[多項式承諾入門]] - KZG 和 IPA 的前置概念
- [[橢圓曲線配對導論]] - Bilinear pairing 的直覺
- [[zkSNARKs 支援]] - 零知識證明系統
- [[Precompiled Contracts]] - L1 密碼學加速
- [[Beacon Chain]] - 共識層與 blob sidecar 的關係
