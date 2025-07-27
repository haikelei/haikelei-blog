---
author: Gary Lu
pubDatetime: 2024-11-25T15:00:00+08:00
modDatetime: 2024-11-25T15:00:00+08:00
title: Bitcoin Core Technology Analysis
slug: bitcoin-core-technology-analysis
featured: true
draft: false
tags:
  - blockchain
  - bitcoin
  - technology
description: Building upon blockchain fundamentals and key technologies, this article provides an in-depth analysis of Bitcoin's core technologies including blockchain structure, consensus protocol, mining, and transaction processes.
---

# Bitcoin Core Technology Analysis

## Introduction

Building upon blockchain fundamentals and key technologies covered in previous discussions, Bitcoin stands as the most exemplary application of blockchain technology. This article provides an in-depth analysis of Bitcoin's core technologies.

## Bitcoin System

Bitcoin is a digital currency invented by Satoshi Nakamoto in 2009, primarily designed to challenge centralized banking systems. Due to its sophisticated system design and security features, its value has risen rapidly. However, because it's not tied to real-world identities and offers strong anonymity, it has also been used for illegal transactions, money laundering, ransomware, and other malicious activities, causing some controversy.

As a decentralized blockchain system, everyone can access it and maintain a local node to participate in the Bitcoin network. This analysis will also demonstrate using the `Bitcoin Core` client to maintain a local node.

![bitcoin_network_nodes](https://image.pseudoyu.com/images/bitcoin_network_nodes.png)

Nodes are divided into full nodes and light nodes. Initially, all nodes were full nodes, but as data volume increased, Bitcoin clients running on mobile devices or tablets don't need to store the entire blockchain information. These are called `Simplified Payment Verification (SPV)` nodes, or light nodes.

The `Bitcoin Core` client is a full node. Full nodes remain online continuously, maintaining complete blockchain information. Because they maintain a complete `UTXO` set in memory, they verify transaction legitimacy by validating all blockchain blocks and transactions (from genesis block to latest block). They also decide which transactions get packaged into blocks, validate transactions through mining, determine which chain to continue mining on, and choose forks when equal-length forks occur. They also monitor blocks mined by other miners and verify their legitimacy.

Light nodes don't need to stay online continuously and don't need to retain the entire blockchain (which is massive). They only need to keep block headers for each block and only store blocks relevant to themselves rather than all on-chain transactions. Because they don't store complete information, they cannot verify most transaction legitimacy or the correctness of newly published blocks on the network - they can only verify blocks related to themselves. They can verify transaction existence through `Merkle Proof` but cannot confirm transaction non-existence. They can verify mining difficulty since it's stored in block headers.

> Here's an example illustrating transaction verification methods for full nodes versus light nodes:

To verify a transaction T located in block 300,000, a full node would examine all 300,000 blocks (back to the genesis block), building a complete `UTXO` database to ensure this transaction hasn't been spent. A light node would use `Merkle Path` to link all blocks related to transaction T, then wait for blocks 300,001 through 300,006 for confirmation to verify transaction legitimacy.

### Blockchain Structure

A blockchain is a data structure composed of sequentially linked blocks that can be stored in a single file or database. The `Bitcoin Client` uses Google's `LevelDB` database to store data. Each block points to the previous block - if any block is modified, all subsequent blocks are affected. Therefore, tampering with one block requires simultaneously tampering with all subsequent blocks, requiring massive computational power that often costs more than the potential gains, greatly ensuring security.

The blockchain structure includes several core components: `Block Size (4 bytes)`, `Block Header`, `Transaction Counter (1-9 bytes)`, and `Transaction`.

The blockchain's block header is 80 bytes, storing `Version (4 bytes)`, `Previous Block Hash (32 bytes)`, `Merkle Tree Root (32 bytes)`, `Timestamp (4 bytes)`, `Difficulty Target (4 bytes)`, and `Nonce (4 bytes)`.

Each block's hash value is calculated by double-hashing the block header: `SHA256(SHA256(Block Header))`. This isn't stored in the blockchain structure but is calculated by each node upon receiving the block, making it unique. Additionally, `Block Height` can serve as a block identifier.

#### Merkle Tree

The `Merkle Tree` is a crucial data structure in blockchain, primarily used to verify large datasets through hash algorithms (also using double-hashing: `SHA256(SHA256(Block Header))`). The structure is shown below:

![merkle_tree_example](https://image.pseudoyu.com/images/merkle_tree_example.png)

Using `Merkle Tree` allows quick verification that a transaction exists in a specific block (algorithm complexity of `LogN`). For example, to verify transaction K exists in a block, only a few nodes need to be accessed:

![merkle_proof_example](https://image.pseudoyu.com/images/merkle_proof_example.png)

Since Bitcoin networks contain numerous transactions, this method greatly improves efficiency:

![merkle_proof_efficiency](https://image.pseudoyu.com/images/merkle_proof_efficiency.png)

Since light nodes (like mobile Bitcoin wallets) don't store complete blockchain data, the `Merkle Tree` structure makes transaction lookup very convenient. Light nodes construct a `Bloom filter` to obtain transactions relevant to themselves:

1. First, initialize the Bloom filter as empty, get all addresses in the wallet, create a search pattern to match addresses related to transaction outputs, and add the search pattern to the Bloom filter
2. The Bloom filter is then sent to various nodes (via `filterload` message)
3. Nodes respond with a `merkleblock` message containing qualifying block headers and `Merkle Path` for qualifying transactions, plus a `tx` message containing filter results

During this process, light nodes use `Merkle Path` to link transactions with blocks and use block headers to form the blockchain, enabling verification that transactions exist in the blockchain.

Using Bloom filters returns results matching filter conditions but also produces false positives, returning many unrelated results. This also protects privacy when light nodes request related addresses from other nodes.

### Bitcoin Network

The Bitcoin system operates on a P2P peer-to-peer network where nodes are equal with no identity or permission differences. There's no centralized server and no network hierarchy.

Each node must maintain a collection of transactions waiting to be added to the chain. Each block is 1MB in size, requiring several seconds to propagate to most nodes. If a node monitors an A→B transaction, it writes it to the collection. If it simultaneously discovers an A→C double-spend attack, it won't write it. If it monitors the same A→B transaction or an A→C transaction from the same coin source, it will remove the A→B transaction from the collection.

### Bitcoin Consensus Protocol

As an open system where anyone can participate, Bitcoin needs to solve the threat of malicious nodes. The solution is a proof-of-work mechanism, essentially a computational power voting mechanism. When a new transaction occurs, the new data record is broadcast, the entire network executes the consensus algorithm - miners mine to verify records by solving for random numbers. The miner who first solves the puzzle gets recording rights, generates a new block, then broadcasts the new block. Other nodes verify and add it to the main chain after verification.

### Wallet

As a digital currency system, Bitcoin has its own wallet system, primarily composed of private keys, public keys, and wallet addresses.

> The wallet address generation process:

1. Use `ECDSA (Elliptic Curve Digital Signature Algorithm)` to generate the corresponding public key from the private key
2. Since public keys are long and difficult to input and remember, use `SHA256` and `RIPEMD160` algorithms to get a public key hash value
3. Finally, use `Base58Check` processing to get a readable wallet address

### Transaction Process

With a wallet (and assets), transactions can begin. Let's understand this process through a typical Bitcoin transaction:

A and B both have Bitcoin wallet addresses (generated using Bitcoin Client). Assume A wants to transfer 5 BTC to B. A needs B's wallet address, then uses their private key to sign the transaction `A→B transfer 5 BTC` (since only A knows A's private key, possessing the private key means owning wallet assets). Then this transaction is published - initiating transactions in the Bitcoin system requires paying small miner fees as transaction fees. Miners begin verifying transaction legitimacy, and after six confirmations, the transaction can be accepted by the Bitcoin ledger. The entire verification process takes about 10 minutes.

> Why do miners spend massive computational power to verify transactions?

Miners can receive block rewards and miner fees during verification. Block rewards halve every four years, so later incentives are mainly miner fees.

> Why does verification take 10 minutes?

Bitcoin isn't absolutely secure - new transactions are vulnerable to malicious attacks. Controlling mining difficulty to keep verification around 10 minutes largely prevents malicious attacks. This is only probabilistic security.

> How does Bitcoin avoid double spending?

Bitcoin uses a concept called `UTXO (Unspent Transaction Outputs)`. When a user receives a BTC transaction, it's recorded in `UTXO`.

In this example, if A wants to transfer 5 BTC to B, A's 5 BTC might come from two `UTXOs` (2 BTC + 3 BTC). When A transfers to B, miners need to verify whether these two `UTXOs` have been spent before this transaction. If detected as already spent, the transaction is invalid.

The following diagram illustrates multiple transaction flows and `UTXO` concepts:

![btc_utxo_example](https://image.pseudoyu.com/images/btc_utxo_example.png)

Additionally, `UTXO` has an important characteristic - indivisibility. If A has 20 BTC and wants to transfer 5 BTC to B, the transaction first takes 20 BTC as input, then produces two outputs: one transferring 5 BTC to B, and one returning the remaining 15 BTC to A. Therefore, A gains a new `UTXO` worth 15 BTC. If a single `UTXO` isn't enough for payment, multiple can be combined as input, but the total must exceed the transaction amount.

> How do miners verify transaction initiators have sufficient balance?

This seems simple - like checking balance sufficiency in payment apps. But Bitcoin uses a transaction-based ledger model without account concepts, so balance can't be directly queried. To know an account's remaining assets requires reviewing all previous transactions and finding all `UTXOs` to sum them.

### Transaction Model

> We've covered how transactions occur - what components make up Bitcoin transactions?

![blockchain_bitcoin_script_detail](https://image.pseudoyu.com/images/blockchain_bitcoin_script_detail.png)

As shown, the beginning part is `Version`, indicating version.

Then comes Input-related information: `Input Count` indicates quantity, `Input Info` indicates input content, the `Unlocking Script`, mainly used to verify input sources, input availability, and other input details.

- `Previous output hash` - All inputs can be traced back to an output. This points to the UTXO to be spent in this input, with its hash value stored in reverse order
- `Previous output index` - A transaction can have multiple `UTXOs` referenced by index numbers, first index is 0
- `Unlocking Script Size` - Byte size of the `Unlocking Script`
- `Unlocking Script` - Hash satisfying the `UTXO Unlocking Script`
- `Sequence Number` - Defaults to `ffffffff`

Next is Output-related information: `Output Count` indicates quantity, `Output Info` indicates output content, the `Locking Script`, mainly recording how much bitcoin is output, future spending conditions, and output details.

- `Amount` - Output bitcoin amount in Satoshis (smallest bitcoin unit), 10^8 Satoshis = 1 bitcoin
- `Locking Script Size` - Byte size of the Locking Script
- `Locking Script` - Locking Script hash specifying conditions that must be met to use this output

Finally is `Locktime`, indicating the earliest time/block a transaction can be added to the blockchain. If less than 500 million, it reads block height; if greater than 500 million, it reads timestamp.

### Bitcoin Script

Transactions mention `Unlocking script` and `Locking script` - what are Bitcoin scripts?

Bitcoin scripts are instruction lists recorded in each transaction that can verify transaction validity and bitcoin usability when executed. A typical script looks like:

```sh
<sig> <pubKey> OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG
```

Bitcoin scripts execute stack-based from left to right, using `Opcodes` to operate on data. In this script language, <> contains data to be pushed onto the stack, while OP\_-prefixed items without <> are operators (OP can be omitted). Scripts can also embed data permanently recorded on-chain (not exceeding 40 bytes) - this recorded data doesn't affect `UTXO`.

In transactions, `<sig> <pubKey>` is the `Unlocking script`, while `OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG` is the `Locking script`.

Compared to most programming languages, Bitcoin script is non-Turing complete, lacking loops or complex flow control. It's simple to execute with deterministic results regardless of execution location, doesn't save state, and scripts are mutually independent. Due to these characteristics, while Bitcoin scripts are relatively secure, they can't handle complex logic, making them unsuitable for complex business processes. `Ethereum`'s smart contracts achieved innovative breakthroughs in this area, spawning many decentralized applications.

### Mining

Mining was mentioned in the transaction process - let's examine it in detail.

Some nodes verify transactions to earn block rewards and miner fees for profit - this is called miner mining. Block rewards are created by `coinbase`, halving every four years, from 25 in 2009 to currently 6.25.

Mining is essentially a process of continuously trying random numbers to reach a set target value, like being less than a certain target value. This difficulty is artificially set to adjust verification time and improve security, not to solve mathematical problems.

Miners continuously try this value with low success rates but many attempts possible. Therefore, nodes with stronger computational power have proportional advantages and are more likely to solve puzzles.

> Why does mining difficulty need adjustment?

In Bitcoin systems, block times that are too short easily cause forks. If there are too many forks, it affects system consensus achievement, endangering system security. Bitcoin systems adjust difficulty to stabilize block speed around 10 minutes, preventing transaction tampering.

> How is mining difficulty adjusted?

The system adjusts the target threshold every 2016 blocks (about two weeks), stored in block headers. All network nodes must follow the new difficulty for mining. If malicious nodes don't adjust the target in their code, honest miners won't recognize it.

Target threshold = Target threshold \* (Actual time to generate 2016 blocks / Expected time to generate 2016 blocks)

When Bitcoin was first created, there were few miners and low mining difficulty - most used home computers (CPU) for direct mining. As more people joined the Bitcoin ecosystem, mining difficulty increased. People began using more powerful GPUs for mining, and specialized `ASIC (Application Specific Integrated Circuit)` dedicated mining chips and mining machines gradually emerged with market demand. Large mining pools combining massive network computational power for centralized mining have also appeared.

In large mining pool systems, `Pool Manager` acts as a full node, while numerous combined miners calculate hash values together, finally distributing profits through proof-of-work mechanisms. However, excessive computational power concentration can create centralization risks - if a large mining pool reaches over 51% of network computational power, it could roll back transactions or resist certain transactions.

### Forks

Bitcoin systems also experience situations where consensus isn't reached, called forks. Forks are mainly divided into two types: state forks, often deliberately performed by some nodes; and protocol forks, meaning disagreements about Bitcoin protocol.

Protocol forks can be divided into two types. One is hard forks - incompatible modifications to protocol parts, like adjusting Bitcoin block size from 1MB to 4MB. This fork type is permanent, forming two parallel developing chains from a certain point, like `Bitcoin Classic`, creating two currencies.

The other is soft forks - like adjusting Bitcoin block size from 1MB to 0.5MB. After adjustment, new nodes mine small blocks while old nodes mine large blocks. Soft forks are non-permanent. Typical examples include modifying coinbase content and forks generated by `P2SH (Pay to Script Hash)`.

## Bitcoin Core Client

`Bitcoin Core` is Bitcoin's implementation, also called `Bitcoin-QT` or `Satoshi-client`. You can connect to the Bitcoin network, verify blockchain, send and receive bitcoin through this client. It has three networks: `Mainnet`, `Testnet`, and `Regnet`, which can be switched between.

It provides a `Debug Console` for direct interaction with the Bitcoin blockchain. Common operations include:

> Blockchain

- getblockchaininfo: Returns various status information about blockchain processing
- getblockcount: Returns number of blocks in blockchain
- verifychain: Verifies blockchain database

> Hash

- getblockhash: Returns provided block hash value
- getnetworkhashps: Returns network hashes per second based on specified number of recent blocks
- getbestblockhash: Returns best block hash value

> Blocks

- getblock: Returns detailed block information
- getblockheader: Returns block header information
- generate: Immediately mines specified number of blocks to a wallet address

> Wallet

- getwalletinfo: Returns object containing various wallet status information
- listwallets: Returns currently loaded wallet list
- walletpassphrasechange: Changes wallet password

> Mempool

- getmempoolinfo: Returns detailed mempool activity status
- getrawmempool: Returns all transaction details in mempool
- getmempoolentry: Returns mempool data for given transaction

> Transaction

- getchaintxstats: Calculates statistics about total transaction count and rate in chain
- getrawtransaction: Returns raw transaction data
- listtransactions: Returns transaction list for given account

> Signature

- signrawtransaction: Signs raw transaction inputs
- signmessage: Signs message using address private key
- dumpprivkey: Gets private key

> Network

- getnetworkinfo: Returns P2P network status information
- getpeerinfo: Returns data for each connected network node
- getconnectioncount: Returns node connection count

> Mining

- getmininginfo: Returns object containing mining-related information
- getblocktemplate: Returns data needed to construct blocks
- prioritisetransaction: Accepts transactions into mined blocks with higher or lower priority

## Summary

This analysis covers Bitcoin's core technologies, mainly examining its fundamental principles and data models. Learning Bitcoin provides excellent understanding of blockchain design concepts and operating mechanisms. The next focus will be on Ethereum, known as Blockchain 2.0.
