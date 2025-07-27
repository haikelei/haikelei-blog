---
author: Gary Lu
pubDatetime: 2024-11-27T15:00:00+08:00
modDatetime: 2024-11-27T15:00:00+08:00
title: Ethereum Core Technology Analysis
slug: ethereum-core-technology-analysis
featured: true
draft: false
tags:
  - blockchain
  - ethereum
  - smart-contracts
  - web3
description: An in-depth analysis of Ethereum's core technologies, exploring smart contracts, decentralized applications, and the revolutionary concept of programmable blockchain known as "world computer".
---

# Ethereum Core Technology Analysis

## Introduction

Bitcoin, as a decentralized digital currency, has been extremely successful. However, it is limited by Bitcoin's scripting language (non-Turing complete, capable of handling only simple logic) and cannot process complex business logic. Ethereum introduced smart contracts, allowing the decentralized concept to be applied to richer application scenarios, earning it the title of "Blockchain 2.0". As a programmer with over 10 years of experience, I want to share my understanding of Ethereum's core technologies and explore how this platform revolutionized the blockchain space.

## Ethereum System

In January 2014, Russian developer Vitalik Buterin published the Ethereum whitepaper and formed a team, aiming to create a blockchain platform that integrates a more general scripting language. One team member, Dr. Gavin Wood, published a yellow paper covering the technical aspects of the Ethereum Virtual Machine (EVM), marking the birth of Ethereum.

![ethereum_overview](https://image.pseudoyu.com/images/ethereum_overview.png)

Simply put, Ethereum is an open-source decentralized system that uses blockchain to store system state changes, hence it's also called the "world computer". It supports developers in deploying and running immutable programs called smart contracts on the blockchain, enabling support for a wide range of application scenarios. It uses the digital currency Ether to measure system resource consumption and incentivize more people to participate in building the Ethereum system.

### Decentralized Applications (DApp)

In a narrow sense, a DApp is an application that integrates a user interface, supports smart contracts, and runs on the Ethereum blockchain.

![ethereum_architecture](https://image.pseudoyu.com/images/ethereum_architecture.png)

As shown in the diagram above, Ethereum application instances are deployed on the blockchain network (smart contracts run in the blockchain virtual machine), while web programs only need to make RPC remote calls to the blockchain network through Web3.js. This way, users can access decentralized service applications through browsers (DApp browsers or plugin tools like MetaMask).

### Ledger

The Ethereum blockchain is a decentralized ledger (database) where all transactions in the network are stored in the blockchain. All nodes must maintain a local copy of the data and ensure the credibility of every transaction. All transactions are public and immutable, and all nodes in the network can view and verify them.

### Accounts

When we need to log into a website or system (like email), we typically need an account and password, with the password stored in encrypted form in a centralized database. However, Ethereum is a decentralized system - how are accounts generated?

Similar to the Bitcoin system:

1. First, generate a private key known only to yourself, let's call it `sk`, and use ECDSA (Elliptic Curve Digital Signature Algorithm) to generate the corresponding public key `pk`
2. Use the `keccak256` algorithm to hash the public key `pk`
3. Take the last 160 bits as the Ethereum address

The user's private key and address together form an Ethereum account, which can store balance, initiate transactions, etc. (Bitcoin's balance is calculated by computing all UTXOs, rather than being stored in an account like Ethereum).

Actually, Ethereum has two types of accounts. The type generated above is called Externally Owned Accounts (EOA), which are accounts owned by regular users, mainly used to send/receive Ether tokens or send transactions to smart contracts (i.e., calling smart contracts).

The other type is Contract Accounts. Unlike external accounts, these accounts don't have corresponding private keys but are generated when contracts are deployed, storing smart contract code. It's worth noting that contract accounts must be called by external accounts or other contracts to send or receive Ether; they cannot actively execute transactions themselves.

### Wallets

Software/plugins that store and manage Ethereum accounts are called wallets, providing functions such as transaction signing and balance management. Wallet generation mainly has two methods: non-deterministic random generation or generation based on random seeds.

### Gas

Operations on the Ethereum network also require "transaction fees" called Gas. Deploying smart contracts on the blockchain and transfers require consuming a certain amount of Gas. This is also an incentive mechanism to encourage miners to participate in Ethereum network construction, making the entire network more secure and reliable.

Each transaction can set the corresponding Gas amount and Gas price. Setting higher Gas fees often means miners will process your transaction faster. However, to prevent transactions from consuming large amounts of Gas fees through multiple executions, you can set limits through Gas Limit. Gas-related information can be queried through the [Ethereum Gas Tracker](https://etherscan.io/gastracker) tool.

```
If START_GAS * GAS_PRICE > caller.balance, halt
Deduct START_GAS * GAS_PRICE from caller.balance
Set GAS = START_GAS
Run code, deducting from GAS
For negative values, add to GAS_REFUND
After termination, add GAS_REFUND to caller.balance
```

### Smart Contracts

As mentioned above, the Ethereum blockchain not only stores transaction information but also stores and executes smart contract code.

Smart contracts control application and transaction logic. Smart contracts in the Ethereum system use the proprietary Solidity language, with syntax similar to JavaScript. Besides this, there are other programming languages like Vyper and Bamboo. Smart contract code is compiled into bytecode and deployed to the blockchain. Once on-chain, it cannot be edited. The EVM serves as a smart contract execution environment that can guarantee the determinism of execution results.

#### Smart Contract Example: Crowdfunding

Let me imagine a more complex scenario. Suppose I want to crowdfund $10,000 to develop a new product. Using existing crowdfunding platforms requires paying substantial fees, and it's difficult to solve trust issues. So, we can solve this problem through a crowdfunding DApp.

First, let's set some rules for crowdfunding:

1. Each person who wants to participate in crowdfunding can donate between $10-$10,000
2. If the target amount is reached, the funds will be sent to me (the crowdfunding initiator) through smart contracts
3. If the target is not reached within a certain time (like 1 month), the crowdfunding funds will be returned to the crowdfunding users
4. Some rules can also be set, such as after one week, if the target amount is not reached, users can apply for refunds

Because these crowdfunding terms are implemented through smart contracts and deployed on a public blockchain, even the initiator cannot tamper with the terms, and anyone can view them, solving the trust problem.

### Transactions

What does a typical transaction look like in Ethereum?

1. Developers deploy smart contracts to the blockchain
2. DApp instantiates contracts, passing in corresponding values to execute contracts
3. DApp digitally signs the transaction
4. Local verification of the transaction
5. Broadcast the transaction to the network
6. Miner nodes receive the transaction and verify it
7. Miner nodes confirm trusted blocks and broadcast them to the network
8. Local nodes synchronize with the network, receiving new blocks

### Architecture

![ethereum_architecture_simple](https://image.pseudoyu.com/images/ethereum_architecture_simple.png)

Ethereum adopts an "Order - Execute - Validate - Update State" system architecture. In this architecture, when a new transaction occurs, miners perform Proof of Work (PoW) calculations; after verification is completed, blocks are broadcast to the network through the gossip protocol; other nodes in the network receive new blocks and also verify them; finally, they are committed to the blockchain, updating the state.

Specifically, the Ethereum system has core components like consensus layer, data layer, and application layer, with interaction logic as follows:

![ethereum_architecture_concrete](https://image.pseudoyu.com/images/ethereum_architecture_concrete.png)

As shown in the diagram above, Ethereum data consists of Transaction Root and State Root. Transaction Root is a tree composed of all transactions, containing From, To, Data, Value, Gas Limit, and Gas Price; while State Root is a tree composed of all accounts, containing Address, Code, Storage, Balance, and Nonce.

## Summary

This covers my analysis of Ethereum's core technologies. The introduction of smart contracts has brought more possibilities to blockchain applications, but there are still many security, privacy, and efficiency issues to consider. For complex enterprise-level application scenarios, consortium blockchains are better choices. In my experience working with blockchain technologies over the past decade, Ethereum's programmable nature has opened up entirely new possibilities for decentralized applications, laying the foundation for the Web3 ecosystem we see today.
