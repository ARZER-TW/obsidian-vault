---
tags: [ethereum, consensus, pos, staking, validator, economic-security]
aliases: [PoS 與質押入門, Proof of Stake 入門, 質押入門, Staking Intro]
---

# PoS 與質押入門

## 概述

Proof of Stake 的安全基礎是一句話：**作弊就賠錢**。Validator 質押 ETH 作為保證金參與共識，如果行為誠實就獲得獎勵，如果違規就被 [[Slashing|slash]]（沒收部分或全部質押金）。這套經濟激勵讓攻擊 Ethereum 的成本極高——目前需要超過 1000 萬 ETH（約數百億美元）才能威脅網路安全。

## 經濟安全模型

PoW 的安全來自物理世界的能源消耗：攻擊者需要購買大量硬體和電力。PoS 把安全基礎從物理資源轉移到經濟資源：

- 誠實行為 -> 獲得質押獎勵（年化約 3-5%）
- 離線不投票 -> 扣除等同於投票獎勵的金額
- 雙重投票或矛盾投票 -> [[Slashing]]，至少損失 1/32 的質押金
- 大規模協同攻擊 -> correlation penalty 可銷毀全部質押金

這個設計讓攻擊者面臨明確的經濟損失。即使攻擊成功，攻擊者質押的 ETH 也會被大量銷毀，等同於「燒毀自己的武器」。

## 為什麼是 32 ETH？

成為 validator 的最低質押門檻是 32 ETH。這個數字是兩個目標之間的權衡：

**去中心化**（門檻越低越好）：
- 門檻低讓更多人能獨立運行 validator
- 更多 validator 意味著更強的抗審查能力

**網路效率**（門檻越高越好）：
- 每個 validator 都需要在 [[Beacon Chain]] state 中佔一個條目
- Validator 數量越多，每個 epoch 需要處理的 attestation 越多
- 網路和計算開銷隨 validator 數量線性增長

32 ETH 是 Ethereum 研究團隊在 2018-2019 年間經過模擬後選定的平衡點。目前全網有超過 100 萬個 validator，state 大小已經是效能瓶頸之一。Pectra 升級引入的 [[Validators|EIP-7251]] 將單一 validator 的有效餘額上限從 32 ETH 提高到 2048 ETH，讓大型質押者可以合併多個 validator，緩解 validator set 膨脹。

對於持有少於 32 ETH 的用戶，可以透過質押池（如 Lido、Rocket Pool）參與。

## Validator 的兩把鑰匙

每個 validator 持有兩組 [[BLS12-381]] 密鑰，分離「日常操作」和「資金控制」：

| 密鑰 | 用途 | 存儲方式 | 風險等級 |
|------|------|---------|---------|
| **Signing key** | 簽署 attestation、block proposal、RANDAO reveal | 熱存儲（必須線上） | 高（暴露即可被冒簽） |
| **Withdrawal key** | 控制質押金提款和退出 | 冷存儲（離線即可） | 低（只在提款時需要） |

這個分離設計的意義：即使 signing key 被竊，攻擊者可以讓你被 slash，但無法偷走你的質押金（需要 withdrawal key）。Pectra 升級（[[Validators|EIP-7002]]）進一步強化了 withdrawal key 的權力，讓持有 withdrawal credential 的人可以直接從 Execution Layer 觸發 validator 退出，不再依賴 signing key。

密鑰推導路徑遵循 EIP-2333/2334：
```
m / 12381 / 3600 / i / 0      -- signing key
m / 12381 / 3600 / i / 0 / 0  -- withdrawal key
```

## Proposer 與 Attester

Validator 在共識中扮演兩種角色：

**Proposer（提議者）**：
- 每個 [[Slot 與 Epoch|slot]] 隨機選出一位，負責組裝並廣播新區塊
- 從 Execution Layer 取得交易，打包成 Beacon Block
- 提交 [[RANDAO]] reveal，貢獻隨機數
- 成功出塊可獲得 proposer reward

**Attester（投票者）**：
- 每個 epoch 中，每位 validator 被分配到一個 slot 的一個 committee
- 投三票：head vote（鏈頭是哪個區塊）、source vote 和 target vote（給 [[Casper FFG]] 的 finality 投票）
- 投票結果透過 [[BLS Signatures]] 聚合，大幅降低網路和驗證開銷

大多數 validator 絕大部分時間都在當 attester。以 100 萬 validator 計算，每個 epoch（32 slot）每位 validator 只提交一次 attestation，但被選為 proposer 的機率大約每 5 天一次。

## Effective Balance

Validator 的投票權重不直接使用實際餘額，而是使用 **effective balance**——一個經過取整和滯後更新的值。

設計原因：如果每次餘額微幅變動（如收到 0.001 ETH 獎勵）都重新計算 committee 分配，[[Beacon Chain]] 的 state transition 開銷會大幅增加。Effective balance 以 1 ETH 為步進，只在實際餘額偏離超過閾值時才更新：

- 向下調整閾值：差距 > 0.25 ETH
- 向上調整閾值：差距 >= 1.25 ETH

Pectra 前上限 32 ETH（所有 validator 投票權重相同），Pectra 後上限 2048 ETH（大額質押者影響力更大，但也承擔更高的 slashing 風險）。

## 相關概念

- [[Validators]] - Validator 的完整生命週期和技術細節
- [[Beacon Chain]] - 管理 validator 註冊和 committee 分配
- [[Slashing]] - 違規 validator 的懲罰機制
- [[Attestation]] - Validator 的投票格式和流程
- [[Casper FFG]] - 利用 attestation 的 source/target vote 達成 finality
- [[RANDAO]] - Proposer 選擇的隨機數來源
- [[共識入門]] - 共識的基礎概念和雙層架構
- [[Slot 與 Epoch]] - Validator 操作的時間框架
