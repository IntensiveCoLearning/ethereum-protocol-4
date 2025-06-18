---
timezone: UTC-7

---


# Matseoi Zau

1. 自我介绍
   大家好，我是一位UNC at Chapel Hill在讀的CS and Math學生。對加密學，區塊鏈，以及科技對社會的改變有興趣。希望可以通過這個共學，更好地瞭解以太坊。
2. 你认为你会完成本次残酷学习吗？
   可以。
3. 你的联系方式（推荐 Telegram）
   TG: @matseoi

## Notes

<!-- Content_START -->

Study Plan:

- Finish reading Vitalik's 6 passages on the future of ethreum protocol in the first half week 6.16~6.21
- Then, read the EPFsg Wiki and the Geth 

### 2025.06.17

Today, I read the first passages on the future of ETH protocol. The first passage is about the merge and the second passage is about the surge. 

#### The Merge

In the first passage, Vitalik discusses the pursuable future of more decentralized staking, reducing the time to finality and improving resilience against network attacks and failures.

To promote more decentralized staking, Vitalik proposes lowering the minimum validator stake from 32 ETH to 1 ETH, making solo staking more accessible and viable.

The goal of reducing finality time is to enable block finalization within each 12-second slot, rather than waiting approximately 15 minutes as is currently the case.

##### Pathways to Single‑Slot Finality (SSF)

- Brute-force signature aggregation – Use advanced schemes (e.g., ZK-SNARKs) to handle millions of validator signatures per slot.

- Orbit committees – Select medium-sized committees that retain economic security while allowing low-overhead validation.

- Two-tier staking – Have a high-deposit tier (for finality) and a lower-deposit tier (delegating or contributing in lighter ways).

- Maintain status quo – Stay with current model, foregoing SSF and staking democratization.

A mix of these proposals are also proposed by Vitalik.

##### Secret Leader Election

Single Secret Leader Election (SSLE) mechanisms is proposed to hide which validator will propose until the block appears. They use cryptographic shuffling or ring signatures but add complexity. Alternatives like network-layer defenses are also being considered

Note:

What is Single Slot Finality?

While finality is a crypto-economic terms pointing to the iead that the blockchain is economically costly to alter, SSF refers to that blocks getting proposed and finalized in the same slot. 

### 2025.07.12

<!-- Content_END -->
