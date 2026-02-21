---
tags: [ethereum, consensus, gasper, casper-ffg, lmd-ghost, fork-choice, finality]
aliases: [Gasper, Gasper 共識協議, Gasper Protocol]
---

# Gasper 共識協議

## 概述

Gasper 是 Ethereum Proof of Stake 共識協議的正式名稱，由 [[Casper FFG]]（Friendly Finality Gadget）和 [[LMD GHOST]]（Latest Message Driven Greedy Heaviest Observed Sub-Tree）兩個機制組合而成。Casper FFG 負責 finality（不可逆性），LMD GHOST 負責 fork choice（分叉選擇）。兩者解決的是共識中兩個不同但互補的問題。

## 為什麼需要兩個機制？

分散式共識必須同時保障兩個屬性：

- **Liveness（活性）**：網路必須持續前進，不能卡住。即使部分節點離線或網路分區，剩下的節點應該能繼續出塊。
- **Safety（安全性）**：已確認的區塊不能被推翻。如果系統告訴你「這筆交易已經 finalize」，它就不應該消失。

根據 FLP impossibility theorem，沒有單一演算法能在非同步網路中同時完美保障這兩者。Ethereum 的解法是用兩個專門化的機制分工：

| 面向 | 機制 | 運作頻率 | 保障 |
|------|------|---------|------|
| **現在往哪走** | LMD GHOST | 每個 [[Slot 與 Epoch\|slot]]（12 秒） | Liveness |
| **哪些不可逆** | Casper FFG | 每個 [[Slot 與 Epoch\|epoch]]（6.4 分鐘） | Safety |

## LMD GHOST：每 Slot 的分叉選擇

LMD GHOST 回答一個問題：「在目前已知的所有區塊中，canonical chain 的最新頭部（head）是哪一個？」

運作方式：

1. 從最新的 justified checkpoint 開始（由 Casper FFG 決定）
2. 在每個分叉點，計算各分支上的 validator 投票權重（以最新的 attestation 為準）
3. 選擇權重最高的分支
4. 重複直到到達葉節點——該節點就是 chain head

「Latest Message Driven」的含義：每個 validator 只計算最新的一票。如果 validator A 先投給分支 X、後來改投分支 Y，只有投給 Y 的那票被計算。這防止了「重複投票灌票」的攻擊。

LMD GHOST 的安全假設是**超過 50% 的質押權重由誠實節點持有**。在此條件下，誠實節點的投票總是能主導分叉選擇。

## Casper FFG：每 Epoch 的最終性

Casper FFG 回答另一個問題：「哪些 checkpoint 已經確認為永久不可逆？」

每個 epoch 的第一個 slot 對應一個 [[Slot 與 Epoch|checkpoint]]。Validator 在每次 [[Attestation]] 中包含：

- **Source**：最新的 justified checkpoint
- **Target**：當前 epoch 的 checkpoint

當超過 2/3 質押權重的 validator 投出從 source 到 target 的一致投票時，target 被 **justified**。當連續兩個 epoch 的 checkpoint 都被 justified，前一個就被 **finalized**。

Finalized 的區塊不可逆轉——除非攻擊者控制並願意銷毀超過 1/3 的總質押量（目前約 1000 萬 ETH）。任何試圖推翻 finalized checkpoint 的行為都會觸發 [[Slashing]]，讓攻擊者付出巨大經濟代價。

Casper FFG 的安全假設是**超過 2/3 的質押權重由誠實節點持有**。

## 網路分裂時的處理

用一個具體例子說明兩個機制如何配合：

假設網路暫時分裂為 A、B 兩群，各持 50% 的 validator：

**LMD GHOST 維持出塊**：
- A 群和 B 群各自繼續出塊，形成兩個分支
- 每個群內的 validator 用 LMD GHOST 選擇自己看到的最重分支作為 head
- 網路不會停擺（liveness 得到保障）

**Casper FFG 暫停 finality**：
- 兩群各只有 50% 的投票權重，都無法達到 2/3 的 supermajority
- 沒有新的 checkpoint 被 justified 或 finalized
- 已經 finalized 的區塊仍然安全（safety 得到保障）

**網路恢復後**：
- 所有 validator 重新看到完整的區塊和投票
- LMD GHOST 自動選擇權重更高的分支（通常是較長的那條）
- 投票重新集中，Casper FFG 恢復 finalization
- 較短的分支被孤立（orphaned），其上的交易需要重新收錄

網路分裂期間超過 4 個 epoch 未 finalize 時，會觸發 **inactivity leak**：離線 validator 的餘額加速衰減，直到線上的 validator 佔比重新超過 2/3，finality 自動恢復。

## 安全性保證總結

| 機制 | 誠實節點需求 | 保障 | 失敗後果 |
|------|------------|------|---------|
| LMD GHOST | > 50% 質押權重 | 正確的 fork choice | 攻擊者可短暫控制 chain head |
| Casper FFG | > 2/3 質押權重 | Finality 不可逆 | Finality 暫停（不會逆轉） |

兩個機制的安全閾值不同。即使 LMD GHOST 被短暫操控（如 51% 攻擊），只要不超過 1/3 的質押被惡意使用，已 finalized 的區塊仍然安全。這提供了分層防禦：短期波動由 LMD GHOST 處理，長期確定性由 Casper FFG 保障。

## 相關概念

- [[Casper FFG]] - Finality 機制的完整技術細節
- [[LMD GHOST]] - Fork choice 規則的完整技術細節
- [[Attestation]] - Validator 投票（同時服務 GHOST 和 FFG）
- [[Slashing]] - 違反 FFG 規則的懲罰
- [[Beacon Chain]] - 執行 Gasper 協議的共識層
- [[Validators]] - Gasper 的參與者
- [[共識入門]] - 共識的基礎概念
- [[Slot 與 Epoch]] - Gasper 運作的時間框架
