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

<!-- Content_END -->
