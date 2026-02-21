---
tags: [ethereum, data-structure, verkle, statelessness, trie]
aliases: [Verkle Trees, Verkle Trie, VKT]
---

# Verkle Trees

## 為什麼需要 Verkle Trees？

Ethereum 的全局狀態（所有帳戶的 balance、nonce、合約 storage）目前超過 100GB，且持續增長。這帶來一個根本問題：運行全節點需要高效能 SSD 和大量記憶體，而且同步時間越來越長。更重要的是，每個區塊的驗證者都必須在本地擁有完整的狀態副本，才能確認交易結果是否正確。

Stateless client 是解決方案：讓驗證者不需要存儲完整狀態，只需要區塊附帶的「見證資料」（witness）就能驗證狀態轉換。但在目前的 Merkle Patricia Trie 結構下，一筆交易的 witness（從葉到根的所有 sibling hash）可能達到數 KB，一個區塊的總 witness 可以達到數 MB，頻寬壓力巨大。

Verkle Trees 透過 polynomial commitment 取代 hash-based commitment，將 proof 大小從 $O(k \log n)$ 壓縮到 $O(k)$。一個區塊的 witness 從數 MB 縮減到數百 KB，使得 stateless client 在實際頻寬限制下變得可行。

## 概述

Verkle Trees（名稱來自 Vector commitment + Merkle）是 Ethereum 計劃用來取代 [[Merkle Patricia Trie]] 的新資料結構。核心改進是用 polynomial commitment（如 IPA 或 [[KZG Commitments]]）取代 hash-based commitment，使得 proof 大小從 $O(k \log n)$ 縮減到 $O(k)$（$k$ 是查詢的 key 數量），為 stateless client 鋪路。

## 核心原理

### Merkle Tree 的瓶頸

在 [[Merkle Patricia Trie]] 中，證明某個 key 的存在需要提供從葉到根的所有 sibling hash。對於 branching factor $b$ 和深度 $d$ 的樹：

$$\text{proof size} = O(d \times b) = O(\log_b n \times b)$$

如果 $b = 16$（MPT 的 branching factor），每層需要 15 個 sibling hash（每個 32 bytes）。整個 state proof 可能達數 MB。

### Verkle Tree 的結構

Verkle Tree 用 polynomial commitment 取代 hash：

- 每個內部節點有最多 256 個子節點（$b = 256$）
- 節點的 commitment 是子節點值的 polynomial commitment
- 子節點值 $[v_0, v_1, ..., v_{255}]$ 構成多項式 $p(x)$，使得 $p(i) = v_i$
- 節點的 commitment $C = \text{Commit}(p)$

### Proof 大小比較

| 結構 | Branching Factor | 深度 | Proof Size（單 key） |
|------|-----------------|------|---------------------|
| Merkle Patricia Trie | 16 | ~24 | ~3.5 KB |
| Binary Merkle Tree | 2 | ~32 | ~1 KB |
| Verkle Tree | 256 | ~4 | ~150 bytes |

Verkle 的 proof 如此小是因為：polynomial commitment opening 的大小與多項式 degree 無關，每層只需一個 opening proof（~32-48 bytes），而不需要所有 sibling。

### 為什麼選 IPA 而非 KZG？

Verkle Tree 的每個節點需要一個 polynomial commitment，理論上 KZG 和 IPA 都能勝任。Ethereum 最終選擇 IPA（Inner Product Argument），以下是技術上的考量：

| 特性 | IPA（Ethereum 選擇） | [[KZG Commitments|KZG]] |
|------|---------------------|-----|
| Trusted setup | **不需要** | 需要（KZG Ceremony） |
| Proof 大小 | 稍大（~log n 的 group elements） | 固定 48 bytes |
| 驗證時間 | 較慢（$O(n)$ multi-scalar multiplication） | 較快（2 pairings） |
| 曲線 | Bandersnatch（嵌入 BLS12-381） | [[BLS12-381]] |
| SNARK 內部驗證 | 高效（Bandersnatch 嵌入 BLS12-381） | 需 pairing（更昂貴） |
| 量子安全 | 否（同 DLP） | 否（同 DLP） |

選擇 IPA 的核心理由：

1. **消除信任假設**：KZG 需要 trusted setup ceremony（2023 年的 ceremony 有 141,000 人參與）。雖然安全假設只需一個誠實參與者，但這仍然是一個額外的信任要求。IPA 完全不需要 trusted setup。
2. **SNARK 友好**：Bandersnatch 曲線嵌入 BLS12-381 的標量域，使得 Verkle proof 可以高效地在 SNARK 電路中驗證——這對未來的 zkEVM 和遞迴 proof 非常重要。
3. **Multi-proof 合併**：Verkle Tree 的一大優勢是多個 key 的 proof 可以合併為單一 proof，IPA 的 multi-opening 在此場景下表現良好。

### Bandersnatch 曲線

Bandersnatch 是嵌在 [[BLS12-381]] 標量域中的 twisted Edwards 曲線：

$$-5x^2 + y^2 = 1 + dx^2y^2$$

特點：
- 標量域與 BLS12-381 的基域相同，方便 SNARK 內部驗證
- 支援高效的 multi-scalar multiplication
- GLV endomorphism 加速

### 樹的結構細節

Ethereum 的 Verkle Tree 設計（EIP-6800）：

**Key 結構**（32 bytes）：
```
[stem: 31 bytes][suffix: 1 byte]
```

- **Stem**：前 31 bytes，決定在樹中的路徑
- **Suffix**：最後 1 byte（0-255），對應葉節點中的 slot

**節點類型**：

1. **Inner node**：256 個子節點的 commitment
2. **Extension node**：包含 stem 和兩個 commitment（C1 和 C2）
   - C1：suffix 0-127 的值的 commitment
   - C2：suffix 128-255 的值的 commitment

**地址空間映射**：
帳戶的各種資料（version、balance、nonce、code hash、storage slots）被映射到不同的 suffix：

| Suffix | 資料 |
|--------|------|
| 0 | Version |
| 1 | Balance |
| 2 | Nonce |
| 3 | Code hash |
| 4 | Code size |
| 64-127 | Code chunks (前 128 chunks) |
| 128-255 | 保留 |

Storage slots 透過特定的 hash function 映射到獨立的 stem。

### 多重 Proof 合併

Verkle Tree 的一大優勢：多個 key 的 proof 可以高效合併。

給定要證明的 key 集合 $\{k_1, k_2, ..., k_m\}$，它們的路徑可能共享中間節點。合併 proof 只需要：

1. 收集所有涉及的節點
2. 對每個節點產生一個 multi-opening proof
3. 用隨機線性組合壓縮為單一 proof

最終 proof 大小約 $O(d)$（樹的深度），與 key 數量幾乎無關（只要 key 共享路徑）。

### State Transition Witness

有了 Verkle proof，block 的 witness 包含：

1. 所有被讀取/修改的 key-value pair
2. 一個 Verkle multi-proof 證明這些值在 state trie 中

Block verifier 不需要完整的 state，只需要 witness 就能驗證 state transition。這就是 **stateless client**。

## 在 Ethereum 中的應用

### Stateless Ethereum

Verkle Trees 是 Stateless Ethereum roadmap 的核心：

- 全節點不需要存儲完整的 state（目前 > 100GB）
- Block producer 在區塊中附帶 witness
- Verifier 用 witness 驗證 [[狀態轉換]]
- 大幅降低節點硬體需求

### Migration 策略

從 MPT 遷移到 Verkle Tree 的過渡方案：

1. **Overlay approach**：新的寫入進 Verkle Tree，舊的保留在 MPT
2. 讀取時先查 Verkle，miss 時 fallback 到 MPT
3. 逐步將 MPT 資料遷移到 Verkle
4. 遷移完成後停用 MPT

### 時程

Verkle Trees 目標在 Fusaka 之後的升級上線（暫稱 Osaka/Amsterdam，時間待定）。Pectra（2025 Q2，已上線）和 Fusaka（預計 2026）都未納入 Verkle，優先處理了 blob 擴容和其他改進。Verkle 的 devnet 測試仍在進行中，正式上線時程取決於測試進度。

目前狀態（截至 2026 初）：
- 多個客戶端（Geth、Nethermind、Besu）正在積極實作 Verkle 支援
- Devnet 測試持續進行中
- EIP-6800（Verkle state tree）和相關 EIP 仍在迭代
- Gas 計費重新設計（witness size 影響 gas cost）尚未定案

主要挑戰：

- Gas 計費重新設計（witness size 影響 gas cost）
- 客戶端實作的效能優化，特別是 IPA proof 的計算效率
- MPT 到 Verkle 的遷移期相容性
- 遷移期間節點需要同時維護兩套資料結構的儲存成本

## 程式碼範例

```python
# Verkle Tree 節點結構（簡化版）
from dataclasses import dataclass
from typing import Optional, List

@dataclass(frozen=True)
class InnerNode:
    """內部節點：256 個子節點的 commitment。"""
    commitment: bytes  # IPA commitment
    children: tuple    # 256 個子節點（可為 None）

@dataclass(frozen=True)
class ExtensionNode:
    """延伸節點：一個 stem 對應 256 個值。"""
    stem: bytes        # 31 bytes
    commitment: bytes  # 整體 commitment
    c1: bytes          # suffix 0-127 的 commitment
    c2: bytes          # suffix 128-255 的 commitment
    values: tuple      # 256 個值（可為 None）

def get_tree_key(address: bytes, tree_index: int, sub_index: int) -> bytes:
    """計算帳戶資料在 Verkle Tree 中的 key。"""
    # stem = hash(address || tree_index)[0:31]
    stem = pedersen_hash(address + tree_index.to_bytes(32, 'big'))[:31]
    # key = stem || sub_index
    return stem + bytes([sub_index])

# 帳戶欄位映射
BASIC_DATA_LEAF_KEY = 0
CODE_HASH_LEAF_KEY = 1

def get_balance_key(address: bytes) -> bytes:
    return get_tree_key(address, 0, BASIC_DATA_LEAF_KEY)

def get_nonce_key(address: bytes) -> bytes:
    return get_tree_key(address, 0, BASIC_DATA_LEAF_KEY)  # packed in same slot

def get_code_hash_key(address: bytes) -> bytes:
    return get_tree_key(address, 0, CODE_HASH_LEAF_KEY)
```

```javascript
// Verkle proof 驗證（概念）
function verifyVerkleProof(root, proof, keys, values) {
  // proof 包含：
  // - commitments: 路徑上所有涉及的節點 commitment
  // - multiproof: 合併的 IPA multi-opening proof

  const { commitments, multiproof } = proof;

  // 1. 重建路徑上的期望 commitment
  const expectedCommitments = rebuildCommitments(keys, values, commitments);

  // 2. 驗證 root 一致
  if (expectedCommitments[0] !== root) {
    return false;
  }

  // 3. 驗證 IPA multi-proof
  return verifyIPAMultiProof(commitments, multiproof, keys, values);
}

// Witness 大小估算
function estimateWitnessSize(numKeys, treeDepth) {
  // 每個 key-value pair: 32 + 32 = 64 bytes
  const kvSize = numKeys * 64;

  // Multi-proof: 大約 depth * 32 bytes + 常數
  const proofSize = treeDepth * 32 + 128;

  // 路徑上的 commitments
  const commitmentSize = treeDepth * numKeys * 32;  // 上界，實際會共享

  return {
    kvSize,
    proofSize,
    commitmentSize,
    total: kvSize + proofSize + commitmentSize,
    merkleEquivalent: numKeys * 24 * 32,  // MPT 的估計大小
  };
}
```

## 相關概念

- [[Merkle Patricia Trie]] - Verkle Trees 要取代的現有資料結構
- [[KZG Commitments]] - 一種 polynomial commitment scheme（Verkle 選用 IPA）
- [[State Trie]] - 現有的全局狀態樹
- [[Storage Trie]] - 現有的合約存儲樹
- [[區塊 Header]] - stateRoot 將指向 Verkle Tree root
- [[橢圓曲線密碼學]] - IPA 和 KZG 的數學基礎
- [[BLS12-381]] - Bandersnatch 曲線嵌入的目標曲線
- [[EIP-4844 Proto-Danksharding]] - 另一個朝向擴展性的 EIP
- [[狀態轉換]] - Verkle witness 讓 stateless client 能驗證狀態轉換
