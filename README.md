# Merkle Trees

Merkle Trees are a fundamental concept in blockchain technology.

## What is a Merkle Tree?

A merkle tree is a type of hash tree in which each leaf node is labeled with the cryptographic hash of a data block, and each non-leaf node is labeled with the cryptographic hash of its child nodes' labels. The majority of hash tree implementations are binary (each node has two child nodes), but they can also have many more child nodes.

A typical Merkle Tree looks something like this:
![](https://upload.wikimedia.org/wikipedia/commons/thumb/9/95/Hash_Tree.svg/1920px-Hash_Tree.svg.png)

## Simple Example

Let's say we have 4 transactions: "Transaction A", B, C and D. All of them are executed in the same block. Each of these transactions is going to get hashed. Let's call those hashes "Hash A", B, C, and D respectively.

The following would be the resulting Merkle Tree of these transactions:

![](https://i.imgur.com/QeUy35i.jpg)

## Verifying Validity using the Merkle Root

When these transactions get rolled up into a block, the block header would contain the Merkle Root, Hash ABCD. All miners have a copy of all transactions so far, and therefore all the transaction hashes. Any miner can rebuild the merkle tree on demand, which means that every miner can independently arrive at the same merkle root for the same set of transactions.

This allows any miner to verify a fraudulent transaction. Let's say someone tries to introduce a false transactions instead of Transaction D. Let's call this Transaction E. Because this transaction is different from Transaction D, the hash is going to be different as well. The hash of Transaction E is Hash E. The has of C and E together is Hash CE, which is different from Hash CD. When Hash AB and CE are hashed together, you get Hash ABCE. Since hash ABCE is different from Hash ABCD, we can conclude that Transaction E is fraudulent.

![](https://i.imgur.com/QNaIOvk.jpg)

A miner can recompute the Merkle Root in their own block and try to publish that version to the blockchain, but since every other miner has a different Merkle Root, the fraudulent miner is easily rejected.

## Hash Function

To hash Transaction A into Hash A, a one-way cryptographic hash function is used. Once hashed, Hash A cannot be easily turned into Transaction A; the hash is not reversible.

Each blockchain uses different hash functions, but they all have the same properties in common.

#### Deterministic

The same input always has the same output

#### Computationally Efficient

The hash calculation is fast.

#### Cannot be Reversed Engineered

Given a resulting hash, it is near impossible to determine the input.

#### Collision Resistant

Two different inputs never generate the same output.

#### Pre-Image Resistant

For essentially all pre-specified outputs, it is computationally infeasible to find any input that hashes to that output. For example: given `y`, it is difficult to find an `x` such that `h(x) = y`

## Benefits of Merkle Trees in Blockchains

Merkle Trees allow for quick verification of data integrity.

The disk space used up is very little compared to the entire set of transactions. The Merkle Root is included in the block header for this reason.

If you have two different sets of transactions, verifying they are the same with a Merkle Tree is faster then verifying each and every single individual transaction to each other. Once can verify that a a block has not been modified with only knowing the Merkle Root.

## Use cases outside of blockchain

Merkle Trees aren't just used in blockchain applications. Some popular applications that use Merkle Trees are:

- [IPFS](https://en.wikipedia.org/wiki/InterPlanetary_File_System)
- [Git](https://github.com)
- Distributed databases such as [AWS DynamoDB](https://aws.amazon.com/dynamodb) and [Apache Cassandra](https://cassandra.apache.org/_/index.html) use Merkle trees to control discrepancies
