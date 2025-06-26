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

### 2025.06.18
The Surge refers to a phase in Ethereum's development roadmap focused on scaling the network to handle a massive number of transactions per second (TPS), while maintaining decentralization and security.

Vitalik also discusses the scalability trilemma, which highlights the difficulty of simultaneously achieving decentralization, scalability, and security. This was long considered a hard tradeoff, but recent innovations like SNARKs and data availability sampling (DAS) have made significant progress in addressing it.

In summary, DAS allows nodes to verify that data is available without downloading all of it, data compression reduces the space each transaction takes on-chain, and Plasma improves scalability by posting only Merkle roots of its state to Ethereum, greatly reducing data requirements.

Also, Ethereum wants the L2 ecosystem to feel like one unified chain, not many separate blockchains. Solutions include chain-specific addresses, shared bridges, and standardized cross-chain transactions.

Note:

State channel: A small group of participants stake ETH on smart contract and only the final state is being recorded on level 1 chain, allowing extreme low fee but fixed within a fixed group of people and limited flexibility.

Plasma: running a separate blockchain (a “child chain”) and post only periodic summaries of its state to Ethereum.

Rollups: bundling large numbers of transactions and submit either fraud proofs or validity proofs to Ethereumsupport complex smart contracts and enable a wide range of decentralized applications.

### 2025.06.19
#### The scourge

This passage discusses the current centralization risks in Ethereum caused by PoS and the current block construction mechanism. Specifically, smaller stakers tend to join larger staking pools to maximize MEV (Maximal Extractable Value), which increases efficiency but risks centralization. If two dominant builders control around 80% of blocks and decide to censor transactions, inclusion could be delayed by an average of 9 slots. which would allow market manipulation.

#### Proposed Solutions:

1. FOCIL + APS:
   - Inclusion lists ensure key transactions can’t be censored.
   - Attester-Proposer Separation (APS) auctions off block production while proposers ensure inclusion.
2. BRAID (Multiple Concurrent Proposers):
   - Distributes the block building task to multiple proposers.
   - More decentralized but needs encrypted mempools for privacy.
3. Encrypted Mempools:
   - Protect user transactions from front-running.
   - Requires delay encryption or threshold decryption to ensure post-inclusion transparency.

### 2025.06.21
#### The Verge

In this passage, Vitalik discusses The Verge, which refers to making full verification of the Ethereum chain computationally affordable for marginal devices. Originally, The Verge focused on restructuring Ethereum's state representation into Verkle Trees, but it has since evolved into a broader goal: enabling efficient, low-cost verification for all users and devices.

There are two main approaches to achieving The Verge's goals: Verkle Trees and STARKed Binary Hash Trees. Verkle Trees are currently the most mature and time-efficient solution. However, they rely on elliptic curve cryptography, which is not considered post-quantum secure. On the other hand, STARKs are post-quantum safe but either suffer from longer proving times (if using conservative hash functions) or lack maturity and widespread trust (if using newer, aggressive hash functions).

| Approach                                     | Proof Size        | Security                    | Prover Time |
| -------------------------------------------- | ----------------- | --------------------------- | ----------- |
| Verkle Trees                                 | Data + 0.1–2 MB   | EC-based (not quantum-safe) | <1s         |
| STARK w/ Conservative Hashes (SHA256, BLAKE) | Data + 100–300 kB | Trusted                     | >10s        |
| STARK w/ Aggressive Hashes (Poseidon, Ajtai) | Data + 100–300 kB | Less mature                 | 1–2s        |

### 2025.06.22
#### The Purge
Today, I started reading the Purge, which is about reducing data storage burden on clients whileing preserving ETH's permanence of on-chain data. However, I haven't finished reading this passage. Hence, I'll summarize tmr. 

### 2025.06.23
請假

<!-- Content_END -->
