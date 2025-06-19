---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# fffuuuming

1. 自我介绍：資工碩班，想踏入web3
2. 你认为你会完成本次残酷学习吗？盡力
3. 你的联系方式（推荐 Telegram）@fffuuuming

[HACKMD LINK](https://hackmd.io/@BKuxYlL0T6iT9rROj2QldQ/H1zMMB3Qll)
###### tags:`intensive-colearning`
## Notes

<!-- Content_START -->
### 2025.06.16
[Ethereum State Trie Architecture Explained](https://medium.com/@eiki1212/ethereum-state-trie-architecture-explained-a30237009d4e)
---
- **World State Trie (Global State Trie)**: A global state constantly updated by transaction executions
    - Stores the state of all Ethereum accounts, including `EOA` & contracts
    - Each account is stored as a key-value pair: 
        - **Key** = `Keccak-256(address)`
        - **Value** = RLP-encoded account object below
            ```
            [
              nonce,
              balance,
              storageRoot,  <- pointer to the account's storage trie
              codeHash
            ]
            ```
- **Account Storage Trie**: Store contract’s key-value storage (i.e., its state variables)
    - **Key** = `Keccak-256(storage slot)`
	- **Value** = 32-byte data stored at that slot

Relationship between World State Trie  and Account Storage Trie:
```
World State Trie (global)
│
├── Keccak(addr_1) → [nonce, balance, ⛔, ⛔]            (EOA)
│
├── Keccak(addr_2) → [nonce, balance, root_2, codeHash] (Contract)
│                         │
│                         └── Account Storage Trie
│                             ├── Keccak(slot_0) → value_0
│                             ├── Keccak(slot_1) → value_1
│                             └── ...
│
├── ...
└── Keccak(addr_n) → [nonce, balance, root_n, codeHash]
```
- **Transaction Trie**: records transactions in Ethereum
- **Transaction Receipt Trie(Receipt Trie)**: 
#### More Reference
- [Modified Merkle Patricia Trie — How Ethereum saves a state](https://medium.com/codechain/modified-merkle-patricia-trie-how-ethereum-saves-a-state-e6d7555078dd)
### 2025.06.17
[Consensus Layer Overview](https://epf.wiki/#/wiki/CL/overview?id=consensus-layer-overview)
---
**Goal**: 
Achieve reliable coordination of distributed Ethereum nodes over unreliable infrastructure (internet, misconfigured software, malicious actors...), where each node maintains a copy of Ethereum’s state; while consensus ensures all match exactly and update in the same order

**Byzantine Fault Tolerance (BFT)**:
Ability of a distributed system to function correctly even if some nodes behave arbitrarily or maliciously (Byzantine faults), which is essential for decentralized networks where trust among nodes is not assumed.

**Consensus protocol**
- **PoW / PoS**: not a protocol themselves but a **Sybil resistance mechanisms** (placing a cost on participating in the protocol, prevents attackers from overwhelming the system cheaply) that enable consensus protocols
    - PoW → weight = total computational work
    - PoS → weight = total stake supporting the chain

**Ethereum’s Consensus Protocol: Gasper**
Combination of
- LMD GHOST: fork choice rule (decides the head of the chain)
- Casper FFG: finality gadget (helps finalize blocks)

Block payload:
- Execution Layer: list of transactions.
- Pre-Merge Beacon Chain: attestations only.
- Post-Merge: includes execution payload and attestations.
- Post-EIP-4844 (Deneb): includes data blob commitments.
---
**The Merge (Paris Hard Fork)**: The day Ethereum switched to **PoS**

Ethereum was originally **PoW**, but for preparation for **PoS**, it launched a new parallel chain called **Beacon Chain** for running PoS logic only — **no user transactions**. After the Merge, Beacon Chain became the main engine of Ethereum’s consensus (**Consensus Layer**), while the original Ethereum chain became the **Execution Layer**

**Terminal Total Difficulty (TTD)**:

---

#### Beacon Chain Introduction
- **Validators**: responsible for **block proposal** and **attestation**
    - `32 ETH` per validator to participate
        - Each validator is run by a validator client, which interacts with the beacon node that tracks the Beacon Chain.
    - Can be either proposer or one of committee
    - Attestations help finalize blocks and reach agreement on the chain head
    - Rewarded for honesty and penalized for faults or malicious behavior (e.g., slashing).
- **Committees**: A group of ≥128 validators per slot at least
    - Randomly assigned for security
    - Committees attest to block proposals and help finalize chain state
- **Attestations**: validator’s vote on the chain state (specifically, the head block).
    - Votes are weighted by validators' stake and broadcasted in addition to blocks, recorded in the chain
    - Follow the **LMD-GHOST** fork choice rule to determine the head of the Beacon Chain
- **Slot**: A chance for a block to be added to the Beacon Chain every 30 seconds. Can be empty. Each slot is assigned with
    - 1 block proposer
    - Committees of validators for attestation
![截圖 2025-06-17 晚上8.36.29](https://hackmd.io/_uploads/S1MDZk1Nee.png)
- **Epoch**: 32 slots → 6.4 minutes (384 seconds).
![截圖 2025-06-17 晚上8.35.36](https://hackmd.io/_uploads/rk4EbyJNeg.png)
- **Randomness**: Beacon Chain emits publicly verifiable randomness like a “randomness beacon"
    - **[RANDAO](https://inevitableeth.com/home/ethereum/network/consensus/randao)** (Randomness DAO) + **[VDF](https://inevitableeth.com/home/ethereum/upgrades/consensus-updates/vdf)** (Verifiable Delay Function)
    - At every epoch, a pseudorandom process **RANDAO** selects proposers for each slot, and shuffles validators to committees.
![截圖 2025-06-17 晚上8.34.18](https://hackmd.io/_uploads/B1DZ-yJVgl.png)

Example:
![截圖 2025-06-17 晚上8.38.38](https://hackmd.io/_uploads/BkhR-JkVee.png)
- `Slot 1`: Block proposed, 2 validators attest, 1 is offline.
- `Slot 2`: Block proposed, 1 validator attests to previous block (missed latest).
- `Slot 3`: All committee validators agree on the same head (via LMD-GHOST).

The Beacon Chain maintains:
- A validator registry
- Validator states (active, exited, slashed, etc.)
- Attestations (votes)
### 2025.06.18
[Consensus Layer Overview](https://epf.wiki/#/wiki/CL/overview?id=consensus-layer-overview)
---
#### [EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) (Proto-Danksharding)
Introduces a data availability layer to Ethereum, allowing for the **temporary storage** of arbitrary data on the blockchain: **`Blob`**, which provide cheap data availability for L2s like rollups (reducing gas costs).
- **Retention**: Each `blob` is Retained for 4096 epochs (~18 days), new blocks keep adding `blobs`, while older ones are deleted.
- **Storage Impact**: Nodes need to prepare extra disk usage for 
    ```
    131,928 bytes/blob * 4096 epochs * 32 blocks/epoch * 3~6 blobs/block 
    ≈ 52 ~ 104 GB total temporary storage needed
    ```
- **Cryptographic Integrity**
    - Each blob is committed via **[KZG commitments](https://epf.wiki/#/wiki/Cryptography/kzg)** for **integrity verification** & Compatibility with **proposer-builder separation**
    - Needs a trusted setup, done via the **[KZG Ceremony](https://github.com/ethereum/kzg-ceremony)**
#### Checkpoints (EBB): Used for finalizing the blockchain.
- One per epoch: block at the **first slot of an epoch**, or the **latest prior block** if empty
- A block becomes a checkpoint if it **receives attestations from a majority of validators**
- A block is considered **final** when it **is included in two-thirds of the most recent checkpoint attestations**, ensuring it cannot be reverted
![截圖 2025-06-18 下午2.00.03](https://hackmd.io/_uploads/S1gPLR1Vgl.png)
$\mathrm{EBB(n, k)}$: the j-th epoch boundary block of B, to be the block with the highest slot less than or equal to jC in chain(B)
Epoch Boundary Block at slot number n, serving as the checkpoint of epoch k, so $\mathrm{EBB(n, k)}$ = `slot 180`

Validators cast two types of votes:
- **LMD GHOST votes** for `blocks`
- **Casper FFG votes** for `checkpoints`: includeing a source checkpoint from a previous epoch and a target checkpoint from the current epoch. (e.g. a validator in Epoch 1 might vote for a source checkpoint at the genesis block and a target checkpoint at Slot 64)

Note that only validators assigned to a slot cast LMD GHOST votes, while all validators cast FFG votes for epoch checkpoints.
#### Justification & Finality
- **Supermajority**: ⅔ of the total validator balance
- **justified**: Once a checkpoint receives a supermajority
- **finalized**: the subsequent epoch's checkpoint also achieves justification
### 2025.06.19
[Execution Layer Specification](https://epf.wiki/#/wiki/EL/el-specs?id=execution-layer-specification)
---
EL focus on executing the **state transition function** (STF) with two questions:
- Is it possible to append the block to the end of the blockchain?
- How does the state change as a result?

Simplified Overview:
![截圖 2025-06-15 晚上9.30.42](https://hackmd.io/_uploads/SkaMsH2mee.png)
$$\sigma_{t+1} \equiv \Pi(\sigma_t, B)$$
- $B$ : **current [block](https://github.com/ethereum/execution-specs/blob/0f9e4345b60d36c23fffaa69f70cf9cdb975f4ba/src/ethereum/shanghai/fork_types.py#L217)** that is being sent to the execution layer for processing.
- $\sigma_t$ , $\sigma_{t+1}$: **state of the [blockchain](https://github.com/ethereum/execution-specs/blob/0f9e4345b60d36c23fffaa69f70cf9cdb975f4ba/src/ethereum/shanghai/fork.py#L73)** before / after applying the current block
- $\Pi$ : [block level state transition function](https://github.com/ethereum/execution-specs/blob/0f9e4345b60d36c23fffaa69f70cf9cdb975f4ba/src/ethereum/shanghai/fork.py#L145)

前情提要：**collapse function**: an operation that reduces world state into a single hash, to store in the block header
- Takes a set of account/storage entries.
- Builds a Merkle Patricia Trie (MPT)
- Computes the root hash, e.g., `stateRoot`

預計之後會先轉往 Geth 的架構及實現閱讀
<!-- Content_END -->
