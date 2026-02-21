---
tags: [ethereum, cryptography, zero-knowledge, zk-snark, proof-system]
aliases: [zkSNARKs, zk-SNARKs, Zero Knowledge Proofs, 零知識證明]
---

# zkSNARKs 支援

## 什麼是零知識證明？

想像這個情境：你要向某個網站證明你已滿 18 歲，但不想揭露你的生日或任何身份資訊。零知識證明讓這成為可能——你可以產生一份密碼學證明，證明「我持有的身份證上的出生日期在 18 年前」，驗證者可以確認這個陳述為真，但完全學不到你的生日是哪天。

zkSNARK（Zero-Knowledge Succinct Non-interactive Argument of Knowledge）是目前最實用的零知識證明系統。它的三個關鍵特性：

- **零知識**（Zero-Knowledge）：驗證者除了知道陳述為真，學不到任何額外資訊
- **簡潔**（Succinct）：proof 大小是常數（Groth16 只有 192 bytes），驗證時間也是常數，與原始計算的複雜度無關
- **非互動**（Non-interactive）：prover 只需傳送一份 proof，不需要和驗證者來回對話

在 Ethereum 上，zkSNARK 最重要的應用是 ZK Rollup：L2 在鏈下執行數千筆交易，產生一份 proof 證明所有交易都正確執行，然後只需在 L1 上驗證這份 ~280,000 gas 的 proof，而不是重新執行全部交易（可能需要數十億 gas）。

## 概述

Ethereum 透過 BN254 曲線的 [[Precompiled Contracts]]（ecAdd/ecMul/ecPairing）原生支援 Groth16 等 proof system 的鏈上驗證，為 ZK Rollup、隱私交易、身份證明等應用提供基礎設施。

## 從計算到證明：完整推導

### 算術電路（Arithmetic Circuit）

任何計算問題都可以表示為在有限域 $\mathbb{F}_p$ 上的算術電路：

- **Gate**：加法或乘法
- **Wire**：連接 gate 的值
- **Input wire**：public input（verifier 知道）和 private input（witness，只有 prover 知道）
- **Output wire**：計算結果

例如，證明「我知道 $x$ 使得 $x^3 + x + 5 = 35$」（答案 $x = 3$，但 prover 不想揭露 $x$）：

```
Wire assignment: x=3, w1=9, w2=27, w3=30, w4=35

Gate 1 (乘法): w1 = x * x        => 9 = 3 * 3
Gate 2 (乘法): w2 = w1 * x        => 27 = 9 * 3
Gate 3 (加法): w3 = w2 + x        => 30 = 27 + 3
Gate 4 (加法): w4 = w3 + 5        => 35 = 30 + 5
Assert: w4 == 35 (public input)
```

### R1CS（Rank-1 Constraint System）

電路中的每個乘法 gate 轉化為一個 R1CS 約束：

$$\vec{a}_i \cdot \vec{s} \times \vec{b}_i \cdot \vec{s} = \vec{c}_i \cdot \vec{s}$$

其中 $\vec{s} = [1, x, w_1, w_2, w_3, w_4]$ 是完整的 witness vector。

以 Gate 1（$w_1 = x \times x$）為例：
- $\vec{a}_1 = [0, 1, 0, 0, 0, 0]$ -- 選擇 $x$
- $\vec{b}_1 = [0, 1, 0, 0, 0, 0]$ -- 選擇 $x$
- $\vec{c}_1 = [0, 0, 1, 0, 0, 0]$ -- 選擇 $w_1$
- 驗證：$(0 \cdot 1 + 1 \cdot 3) \times (0 \cdot 1 + 1 \cdot 3) = 0 \cdot 1 + 0 \cdot 3 + 1 \cdot 9 = 9$

整個 R1CS 可以寫成矩陣形式：

$$(A \cdot \vec{s}) \circ (B \cdot \vec{s}) = C \cdot \vec{s}$$

其中 $\circ$ 是逐元素乘法（Hadamard product）。

### QAP（Quadratic Arithmetic Program）

R1CS 透過 Lagrange 插值轉化為 QAP：將矩陣的每一列轉成多項式。

定義多項式 $A_j(x), B_j(x), C_j(x)$，使得對每個約束 $i$：

$$A_j(r_i) = a_{i,j}, \quad B_j(r_i) = b_{i,j}, \quad C_j(r_i) = c_{i,j}$$

QAP 滿足條件：

$$\left(\sum_j s_j A_j(x)\right) \cdot \left(\sum_j s_j B_j(x)\right) - \left(\sum_j s_j C_j(x)\right) = H(x) \cdot T(x)$$

其中 $T(x) = \prod_i (x - r_i)$ 是 vanishing polynomial，$H(x)$ 是商多項式。

### Groth16

Groth16 是目前最廣泛使用的 zk-SNARK 系統，也是 Ethereum precompile 直接支援的方案。

**Trusted Setup**（per-circuit）：
1. 選擇 toxic waste $\tau, \alpha, \beta, \gamma, \delta$
2. 在 $\mathbb{G}_1, \mathbb{G}_2$ 上計算 SRS（Structured Reference String）
3. 銷毀 toxic waste

**Proof 結構**：
$$\pi = (A \in \mathbb{G}_1, B \in \mathbb{G}_2, C \in \mathbb{G}_1)$$

Proof 大小固定：192 bytes（BN254 上：$A$ 64B, $B$ 128B, $C$ 64B）。

**Verification**：
$$e(A, B) = e(\alpha, \beta) \cdot e(\sum_{i=0}^{l} x_i \cdot IC_i, \gamma) \cdot e(C, \delta)$$

其中：
- $e$ 是 bilinear pairing
- $x_0, ..., x_l$ 是 public input
- $IC_i$ 是 verification key 中的常數（$\mathbb{G}_1$ 點）
- $\alpha, \beta, \gamma, \delta$ 是 verification key 中的常數

驗證只需 3-4 次 pairing 運算，正好對應 ecPairing precompile。

### BN254 vs BLS12-381

Ethereum 的 precompile 使用 BN254 曲線（也稱 alt_bn128）：

| 特性 | BN254 | [[BLS12-381]] |
|------|-------|--------------|
| 安全等級 | ~100 bits | ~128 bits |
| $\mathbb{G}_1$ 元素大小 | 64 bytes | 96 bytes |
| $\mathbb{G}_2$ 元素大小 | 128 bytes | 192 bytes |
| Pairing cost | 較低 | 較高 |
| EVM 支援 | 有（0x06-0x08，Genesis） | 有（0x0B-0x13，Pectra 2025/5） |

BN254 的安全等級低於推薦的 128 bits（Kim-Barbulescu 攻擊），但目前仍被認為足夠安全。

### EIP-2537：BLS12-381 Precompile 對 ZK 的影響

EIP-2537 在 Pectra 升級（2025/5/7）正式上線，新增了 9 個 [[BLS12-381]] 曲線操作的 [[Precompiled Contracts]]（地址 0x0B-0x13），包括 G1/G2 的加法、乘法、multi-scalar multiplication、pairing 和 map-to-curve。

這對 ZK 生態系的影響是根本性的：

**直接影響**：
- ZK proof system 可以直接基於 BLS12-381（~128 bit 安全）而非 BN254（~100 bit 安全）
- PLONK、Groth16 等 proof 的鏈上驗證可以使用更安全的曲線
- 消除了 ZK Rollup 項目在安全性和 EVM 相容性之間的取捨

**跨層驗證**：
- 執行層現在可以驗證共識層的 [[BLS Signatures]]，因為兩者使用相同曲線
- 輕客戶端合約可以在鏈上驗證 Beacon Chain 的 validator 簽名
- 跨鏈橋可以更安全地驗證 Ethereum finality proof

**新的 proof 系統**：
- 基於 BLS12-381 的遞迴 SNARK 變得可行（Bandersnatch 嵌入 BLS12-381）
- 與 [[Verkle Trees]] 的 IPA proof（使用 Bandersnatch 曲線）整合更自然

### 在 EVM 上的驗證流程

```
1. 合約接收 proof (A, B, C) 和 public inputs
2. 計算 vk_x = IC[0] + sum(input[i] * IC[i+1]) using ecAdd + ecMul
3. 構造 pairing check:
   e(-A, B) * e(alpha, beta) * e(vk_x, gamma) * e(C, delta) == 1
4. 呼叫 ecPairing precompile (0x08) 驗證
5. 返回驗證結果
```

## 在 Ethereum 中的應用

### ZK Rollup

ZK Rollup 是 zkSNARK 在 Ethereum 上最重要的應用：

| 項目 | Proof System | 曲線 |
|------|-------------|------|
| zkSync Era | PLONK + FRI | BN254 |
| Scroll | zkEVM (Halo2) | BN254 |
| Polygon zkEVM | STARK + SNARK | BN254 (final wrapping) |
| Linea | PLONK | BN254 |

流程：
1. L2 sequencer 收集交易並執行
2. Prover 為批次交易生成 zk proof
3. 將 proof + public inputs 提交到 L1 verifier 合約
4. Verifier 合約用 precompile 驗證 proof
5. 驗證通過後更新 L1 上的 state root

### 隱私應用

- **Tornado Cash**（已制裁）：用 zk proof 斷開存款和提款的連結
- **Semaphore**：匿名投票/信號，用 Merkle tree membership proof
- **zkKYC**：證明滿足 KYC 條件而不揭露身份細節

### 身份與憑證

- **ZK identity**：證明年齡 > 18 而不揭露生日
- **ZK credentials**：證明持有某機構頒發的憑證
- **WorldID**：用 zk proof 證明是真人而不揭露身份

### Gas 成本

以 Groth16 為例（BN254）：

| 操作 | Gas |
|------|-----|
| 每個 public input 的 ecMul | 6000 |
| 每個 public input 的 ecAdd | 150 |
| 4-pair pairing check | 214,000 |
| **總計（10 public inputs）** | **~280,000 gas** |

相較之下，直接在 EVM 中驗證同樣的計算可能需要數百萬甚至數十億 gas。

## 程式碼範例

```solidity
// Groth16 verifier 合約（簡化版）
pragma solidity ^0.8.0;

contract Groth16Verifier {
    // Verification key（由 trusted setup 產生）
    struct VerifyingKey {
        uint256[2] alpha;     // G1
        uint256[2][2] beta;   // G2
        uint256[2][2] gamma;  // G2
        uint256[2][2] delta;  // G2
        uint256[2][] IC;      // G1 array, length = num_public_inputs + 1
    }

    struct Proof {
        uint256[2] A;         // G1
        uint256[2][2] B;      // G2
        uint256[2] C;         // G1
    }

    VerifyingKey private vk;

    function verify(
        Proof memory proof,
        uint256[] memory publicInputs
    ) external view returns (bool) {
        require(publicInputs.length + 1 == vk.IC.length, "Invalid inputs");

        // 計算 vk_x = IC[0] + sum(input[i] * IC[i+1])
        uint256[2] memory vk_x = vk.IC[0];
        for (uint256 i = 0; i < publicInputs.length; i++) {
            // ecMul: input[i] * IC[i+1]
            uint256[2] memory term = ecMul(vk.IC[i + 1], publicInputs[i]);
            // ecAdd: vk_x += term
            vk_x = ecAdd(vk_x, term);
        }

        // Pairing check:
        // e(A, B) == e(alpha, beta) * e(vk_x, gamma) * e(C, delta)
        // 等價於: e(-A, B) * e(alpha, beta) * e(vk_x, gamma) * e(C, delta) == 1
        return ecPairing(
            negate(proof.A), proof.B,
            vk.alpha, vk.beta,
            vk_x, vk.gamma,
            proof.C, vk.delta
        );
    }

    function ecAdd(uint256[2] memory p1, uint256[2] memory p2)
        internal view returns (uint256[2] memory r)
    {
        uint256[4] memory input = [p1[0], p1[1], p2[0], p2[1]];
        assembly {
            if iszero(staticcall(gas(), 0x06, input, 128, r, 64)) {
                revert(0, 0)
            }
        }
    }

    function ecMul(uint256[2] memory p, uint256 s)
        internal view returns (uint256[2] memory r)
    {
        uint256[3] memory input = [p[0], p[1], s];
        assembly {
            if iszero(staticcall(gas(), 0x07, input, 96, r, 64)) {
                revert(0, 0)
            }
        }
    }

    function ecPairing(
        uint256[2] memory a1, uint256[2][2] memory b1,
        uint256[2] memory a2, uint256[2][2] memory b2,
        uint256[2] memory a3, uint256[2][2] memory b3,
        uint256[2] memory a4, uint256[2][2] memory b4
    ) internal view returns (bool) {
        uint256[24] memory input;
        // Pack all 4 pairs (each: G1 point + G2 point = 192 bytes)
        // ... (assembly packing omitted for clarity)

        uint256[1] memory result;
        assembly {
            if iszero(staticcall(gas(), 0x08, input, 768, result, 32)) {
                revert(0, 0)
            }
        }
        return result[0] == 1;
    }

    function negate(uint256[2] memory p)
        internal pure returns (uint256[2] memory)
    {
        // BN254 的 p
        uint256 q = 21888242871839275222246405745257275088696311157297823662689037894645226208583;
        return [p[0], q - p[1]];
    }
}
```

```python
# 用 snarkjs 生成和驗證 Groth16 proof（概念流程）
"""
完整流程：
1. 寫 circom 電路
2. trusted setup (Powers of Tau + circuit-specific)
3. 生成 proof
4. 驗證 proof
5. 生成 Solidity verifier
"""

# circom 電路範例：證明知道 hash 的原像
# template HashPreimage() {
#     signal input preimage;
#     signal output hash;
#     component hasher = Poseidon(1);
#     hasher.inputs[0] <== preimage;
#     hash <== hasher.out;
# }

# 生成 proof（使用 snarkjs CLI）
# snarkjs groth16 prove circuit.zkey witness.wtns proof.json public.json

# 驗證 proof
# snarkjs groth16 verify verification_key.json public.json proof.json

# 生成 Solidity verifier
# snarkjs zkey export solidityverifier circuit.zkey verifier.sol

# Python 中組裝 calldata
def format_proof_for_evm(proof, public_inputs):
    """將 snarkjs 輸出轉換為 EVM calldata 格式。"""
    # Groth16 proof 有 A (G1), B (G2), C (G1)
    a = (int(proof["pi_a"][0]), int(proof["pi_a"][1]))
    b = (
        (int(proof["pi_b"][0][1]), int(proof["pi_b"][0][0])),  # 注意 G2 座標順序
        (int(proof["pi_b"][1][1]), int(proof["pi_b"][1][0])),
    )
    c = (int(proof["pi_c"][0]), int(proof["pi_c"][1]))

    inputs = [int(x) for x in public_inputs]

    return {
        "proof": {"A": a, "B": b, "C": c},
        "publicInputs": inputs,
    }
```

```javascript
// 用 snarkjs 在瀏覽器/Node.js 中生成 proof
const snarkjs = require("snarkjs");

async function generateAndVerifyProof() {
  // 載入 circuit
  const wasmPath = "circuit.wasm";
  const zkeyPath = "circuit.zkey";
  const vkeyPath = "verification_key.json";

  // Witness: private input
  const input = { preimage: "12345" };

  // 生成 proof
  const { proof, publicSignals } = await snarkjs.groth16.fullProve(
    input,
    wasmPath,
    zkeyPath
  );

  // 驗證 proof（離線）
  const vkey = JSON.parse(require("fs").readFileSync(vkeyPath));
  const valid = await snarkjs.groth16.verify(vkey, publicSignals, proof);

  // 生成 Solidity calldata
  const calldata = await snarkjs.groth16.exportSolidityCallData(
    proof,
    publicSignals
  );

  return { proof, publicSignals, valid, calldata };
}
```

## 相關概念

- [[Precompiled Contracts]] - ecAdd/ecMul/ecPairing precompile
- [[橢圓曲線密碼學]] - BN254 配對運算的數學基礎
- [[BLS12-381]] - 未來可能取代 BN254 的曲線（EIP-2537）
- [[KZG Commitments]] - 基於 pairing 的 polynomial commitment（PLONK 等使用）
- [[EIP-4844 Proto-Danksharding]] - Point evaluation precompile 支援 blob 驗證
- [[Keccak-256]] - 常與 zk proof 搭配使用的 hash（但 zk-unfriendly）
- [[交易廣播與驗證]] - ZK Rollup 的 proof 驗證是交易驗證的一部分
