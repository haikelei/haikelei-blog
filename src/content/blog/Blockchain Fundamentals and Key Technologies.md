---
author: Gary Lu
pubDatetime: 2024-11-23T00:00:00+08:00
modDatetime: 2024-11-23T00:00:00+08:00
title: Blockchain Fundamentals and Key Technologies
slug: blockchain-fundamentals
featured: true
draft: false
tags:
  - Blockchain
  - DeFi
  - Cryptography
description: This article systematically reviews the fundamentals of blockchain and DeFi, covering cryptographic principles, basic blockchain concepts, typical application scenarios, types of blockchains, core technical frameworks, as well as security and privacy issues. It is suitable for readers interested in blockchain technology and its applications.
---

# Blockchain Fundamentals and Key Technologies

## Preface

Recently, while taking the _Decentralized Finance (DeFi) Infrastructure_ course on Coursera (part of the _Decentralized Finance (DeFi): The Future of Finance_ Specialization), I've gained a more systematic understanding of DeFi and its underlying blockchain foundations. Combined with my prior knowledge of blockchain technologies, I've realized the complexity and practical value of the DeFi ecosystem. I plan to organize a series of articles to sort out knowledge about DeFi, blockchain, cryptocurrencies, and more. If there are any errors or omissions, please feel free to exchange and correct them.

## Cryptographic Principles in Blockchain

Blockchain is closely linked to cryptography. As mentioned in the Coursera course, core technologies such as public-private key encryption, digital signatures, and hashing are widely used in blockchains like Bitcoin and Ethereum. Many consensus algorithms are also based on complex cryptographic concepts. Therefore, understanding several core cryptographic concepts first can help deepen the understanding of their applications in the blockchain and DeFi systems.

### Hash Functions

A hash function is a method that converts arbitrary-length source data into a fixed-length output value through a series of algorithms. Although the concept is simple, its unique characteristics make it widely used in various fields.

You can experience the working principle of hash functions through relevant demos (taking `SHA256` as an example)!

- **One-way irreversibility**: It is easy to perform a hash operation on an input x to get the value H(x), but if given a value H(x), it is almost impossible to reverse-engineer the value of x. This feature protects the source data well.
- **Collision resistance**: Given a value x and another value y, if x is not equal to y, then H(x) is almost impossible to be equal to H(y). It is not completely impossible, but the probability is very low. Therefore, the hash value of a piece of data is almost unique, which can be well used in scenarios such as identity verification.
- **Unpredictability of hash calculation**: It is difficult to derive the hash value based on existing conditions, but it is easy to verify whether it is correct. This mechanism is mainly used in the `PoW` mining mechanism mentioned in the course.

### Encryption/Decryption

Encryption mechanisms are mainly divided into two types: symmetric encryption and asymmetric encryption.

- **Symmetric encryption**: Both parties use the same key for information encryption and decryption, which is convenient and efficient. However, there is a great risk in key distribution. If the key is distributed through the network or other means, it is easy to leak, leading to information leakage.
- **Asymmetric encryption (public-private key encryption)**: Each person generates a pair of keys through an algorithm, called a public key and a private key. If A wants to send information to B, A can encrypt the file with B's public key and send the encrypted information to B. During this process, even if the information is intercepted or leaked, the source file will not be exposed, so it can be transmitted in any way. When B receives the encrypted file, B uses his own private key to decrypt it to obtain the file content. B's private key is not transmitted through any channel and is only known to himself, so it has extremely high security.

In practical applications, asymmetric encryption of large files is inefficient, so a combined mechanism is generally adopted: Suppose A wants to send a large file D to B, first encrypt the file D with a key K using symmetric encryption, and then encrypt the key K with B's public key using asymmetric encryption. A sends the encrypted key K and file D to B. Even if they are intercepted or leaked during transmission, the key K cannot be obtained without B's private key, and thus the file D cannot be accessed. After receiving the encrypted file and key, B first uses his private key to decrypt to get the key K, and then uses the key K to decrypt the file D to obtain the file content.

### Digital Signatures

Digital signatures are another application of asymmetric encryption. As mentioned earlier, each person has a pair of generated public and private keys. In encryption/decryption applications, public keys are used for encryption and private keys for decryption, while the digital signature mechanism is just the opposite. Suppose a file holder encrypts the file with his private key, and others can decrypt it with his public key. If the result is obtained, the ownership of the file can be proved.

A typical application of the digital signature mechanism is in the Bitcoin blockchain network. As mentioned in the Coursera course, users use private keys to prove their ownership of Bitcoin and sign transactions. Others can use public keys to verify whether the transaction is legal. The entire process does not need to expose their private keys, ensuring the security of assets.

## Basic Concepts of Blockchain

As introduced in the Coursera course, from the earliest barter economies to specie currency (backed by gold, etc.), fiat currency, and electronic transfers, the way humans record and exchange value has been evolving. Traditional centralized digital bookkeeping often relies on the credibility of certain organizations, which has trust risks. Blockchain technology is essentially a distributed ledger technology. A group of people jointly maintain a decentralized database and conduct joint bookkeeping through a consensus mechanism.

Blockchain makes it easy to trace historical records, and due to the existence of a decentralized trust mechanism, it is almost impossible to tamper with (or the cost of tampering is much higher than the gain). Compared with traditional databases, blockchain only has two operations: addition and query. All operation history records are accurately stored in the ledger and immutable, with high transparency and security. Of course, the cost is that all nodes must reach a consensus through some mechanisms (so the efficiency is low and not suitable for real-time operations), and each node must permanently store historical records, which takes up a lot of storage space.

## DeFi Application Scenarios and Blockchain Application Value

The Coursera course emphasizes that DeFi is designed to solve a series of problems in the current financial system. To judge whether a business is suitable for adopting blockchain or DeFi solutions, we can refer to the following needs:

- Need a shared database with multi-party participation
- There is a lack of trust between participating parties
- Existing businesses rely on one or more trusted institutions
- Have business needs for encrypted authentication
- There is an urgent need to integrate data into different databases and meet business digitization and consistency requirements
- There are unified rules for system participants
- Multi-party decision-making is transparent
- Need objective and unchangeable records
- Handle non-real-time business

In fact, in many application scenarios, enterprises need to balance decentralization and efficiency, and sometimes many complex businesses have different requirements for transparency and rules. Therefore, based on complex commercial needs, there are solutions such as "consortium chains" that can better integrate with existing systems to meet business needs.

## Types of Blockchains

There are different types of blockchains, mainly including private chains, public chains, and consortium chains.

- **Private chain**: It is mainly applied in a specific field or only runs in a certain enterprise. It is mainly used to solve trust problems, such as cross-department collaboration, and generally does not require external institutions to access data.
- **Public chain**: It involves open transactions and is often used in businesses that require transaction/data openness, such as authentication, traceability, and finance. For example, Bitcoin, Ethereum, and `EOS` mentioned in the course.
- **Consortium chain**: The most prominent feature is that nodes need to verify permissions to participate in the blockchain network, and authentication is generally associated with their real roles. Therefore, consortium chains also have centralized attributes, but their efficiency, scalability, and transaction privacy are greatly improved, meeting the needs of enterprise-level applications. The most widely used one is `Hyperledger Fabric`. It is worth mentioning that consortium chains often do not need tokens as incentives. Instead, each participating node is used as an accounting node, and the economic benefits brought by cross-department business collaboration through blockchain mechanisms are used as internal incentives, which is a healthier way more in line with enterprise applications.

In the long run, public chains and consortium chains will gradually converge in technology. Even for the same business, data that needs to be trusted can be placed on public chains, while some industry data and private data can be placed on consortium chains, and transaction privacy can be ensured through permission management.

## Basic Blockchain Framework

What are the components of a blockchain? As mentioned in the Coursera course, the core components include blocks, blockchains, P2P networks, and consensus mechanisms.

### Blocks

A blockchain is an ecosystem composed of blocks. Each block contains the hash value of the previous block, a timestamp, `Merkle Root`, `Nonce`, and block data. The block size of Bitcoin is 1 MB. You can access relevant demos to experience the generation process of a block.

Since each block contains the hash value of the previous block, according to the hash characteristics mentioned earlier, even a very small change will result in a completely different hash value, so it is easy to detect whether a block has been tampered with. The `Nonce` value is mainly used to adjust the mining difficulty, which can control the time to about 10 minutes to ensure security.

### Blockchain

All blocks are connected in series to form a blockchain, which is a ledger that stores all transaction history records in the network. Because each block contains the hash information of the previous block (for example, the Bitcoin system hashes the header of the previous block twice), changes in transactions will cause the blockchain to break. There are some good demos that can well demonstrate this process!

### P2P Network

A P2P network is a distributed network used to share information and resources between different users. It is a distributed network where everyone in the network can get a copy of information and has access rights. A centralized network means that everyone is connected to one (or a group of) centralized networks; a decentralized network has multiple such central networks, but no single network can have all the information.

### Consensus Mechanisms

A blockchain network is composed of multiple network nodes, and each node stores a copy of information. How do they reach an agreement on transactions? That is, as independent nodes, they need a mechanism to ensure mutual trust, which is the consensus mechanism.

Common consensus mechanisms include `PoW (Proof of Work)`, `PoS (Proof of Stake)`, `DPoS (Delegated Proof of Stake)`, `DBFT (Delegated Byzantine Fault Tolerance)`, etc.

- Bitcoin mainly uses the Proof of Work mechanism, which increases the cost of malicious nodes' misbehavior through computational power competition. By dynamically adjusting the mining difficulty, the transaction time is controlled at about 10 minutes (6 confirmations). However, as Bitcoin mining becomes more popular, it consumes more resources and causes environmental damage; some mining pools have a lot of resources, which may also cause some centralization risks.
- The Proof of Stake mechanism reaches a consensus through voting by equity (usually tokens) holders. This mechanism does not require a lot of computational power competition like Proof of Work, but it also has some risks, called the `Nothing at Stake` problem. Many equity holders will bet on all blocks to profit from them. To solve this problem, the system sets some rules, such as setting some punishment mechanisms for users who create blocks on multiple chains at the same time or create blocks on wrong chains. Ethereum is currently transitioning to this consensus mechanism.
- `EOS` uses Delegated Proof of Stake, selecting some representative nodes to vote. This method aims to optimize the efficiency and results of community voting but brings some centralization risks.
- The `DBFT` consensus mechanism reaches a consensus by assigning different roles to nodes, which can greatly reduce overhead and avoid forks, but there is also the risk of core roles misbehaving.

## Blockchain Security and Privacy

### Security

As a relatively new technology, blockchain also has many security risks, such as attacks on digital currency exchanges, smart contract vulnerabilities, attacks on consensus protocols, attacks on network traffic (Internet ISP), and uploading malicious data. Famous cases include the Mt.Gox incident and the Ethereum DAO incident. Therefore, the security risks of blockchain are also an important research direction of blockchain.

Risk analysis can be carried out from the perspectives of protocols, encryption schemes, applications, program development, and systems to improve the security of blockchain applications. For example, in the Ethereum blockchain, analysis can be carried out on the `Solidity` programming language, `EVM`, and the blockchain itself.

For example, a low-cost attack in smart contracts is to identify operations with low `Gas` fees in the Ethereum network and execute them repeatedly to disrupt the entire network.

For security issues, building a general code detector to check for malicious code will be a more general solution.

### Privacy

When talking about blockchain concepts, we mentioned that one of its important features is related to privacy. However, as pointed out in the Coursera course, in public blockchains, everyone can see the transaction details and historical records on the chain. This feature is mainly used in supply chain links such as food and pharmaceuticals, but for some financial scenarios, such as personal account balances and transaction information, it is easy to cause some privacy risks.

What technologies can be applied to privacy protection of high-value and sensitive information?

- At the hardware level, a trusted execution environment can be adopted, and some secure hardware, such as `Intel SGX`, can be used to greatly ensure privacy; the network can use multi-path forwarding to avoid inferring real identities from node IP addresses.
- At the technical level:
  - Coin mixing technology can mix many transactions, making it difficult to find the corresponding transaction sender and receiver.
  - Blind signature technology can ensure that third parties cannot connect the two parties involved in the transaction.
  - Ring signatures are used to ensure the anonymity of transaction signatures.
  - Zero-knowledge proof can be applied to one party (prover) to prove to the other party (verifier) that a statement is correct without revealing any information other than that the statement is correct.
  - Homomorphic encryption can protect the original data. Given E(x) and E(y), it is easy to calculate some encrypted function values (homomorphic operations) about x and y.
  - Attribute-based encryption (`Attribute-based Encryption, ABE`) adds some attributes/roles to each node to realize permission control, thereby protecting privacy.

It should be noted that even if a transaction generates multiple inputs and outputs, the addresses of these inputs and outputs may be linked by others; in addition, address accounts may also be linked to real identities in the real world.

## Conclusion

The above is a sort of basic knowledge of blockchain and DeFi-related foundations learned from the Coursera course. It mainly involves concept and principle levels. Subsequent articles will update the analysis and thinking on typical applications such as Bitcoin, Ethereum, and `Hyperledger Fabric`, and explore popular technologies such as IPFS, cross-chain, and NFT. Stay tuned!
