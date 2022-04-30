# Merkle Trees

Merkle Trees are a fundamental concept in blockchain technology.

## What is a Merkle Tree?

A merkle tree is a type of hash tree in which each leaf node is labeled with the cryptographic hash of a data block, and each non-leaf node is labeled with the cryptographic hash of its child nodes' labels. The majority of hash tree implementations are binary (each node has two child nodes), but they can also have many more child nodes.

A typical Merkle Tree looks something like this:
![](https://upload.wikimedia.org/wikipedia/commons/thumb/9/95/Hash_Tree.svg/1920px-Hash_Tree.svg.png)

(Reference from [using-merkle-trees-for-nft-whitelists](https://medium.com/@ItsCuzzo/using-merkle-trees-for-nft-whitelists-523b58ada3f9))

## Simple Example

Let's say we have 4 transactions: "Transaction A", B, C and D. All of them are executed in the same block. Each of these transactions is going to get hashed. Let's call those hashes "Hash A", B, C, and D respectively.

The following would be the resulting Merkle Tree of these transactions:

![](https://i.imgur.com/QeUy35i.jpg)

## Verifying Validity using the Merkle Root

When these transactions get rolled up into a block, the block header would contain the Merkle Root, Hash ABCD. All miners have a copy of all transactions so far, and therefore all the transaction hashes. Any miner can rebuild the merkle tree on demand, which means that every miner can independently arrive at the same merkle root for the same set of transactions.

This allows any miner to verify a fraudulent transaction. Let's say someone tries to introduce a false transactions instead of Transaction D. Let's call this Transaction E. Because this transaction is different from Transaction D, the hash is going to be different as well. The hash of Transaction E is Hash E. The Hash of C and E together is Hash CE, which is different from Hash CD. When Hash AB and CE are hashed together, you get Hash ABCE. Since hash ABCE is different from Hash ABCD, we can conclude that Transaction E is fraudulent.

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


## Build 

We will try to use Merkle trees for Whitelisting addressees for a NFT Collection.

The reason why merkle trees will play an important role here is because we will not add each address individually to the whitelist but will only save the root of the merkle tree within our contract

So now when the user will try to mint using their whitelist spot, they will submit a proof which will verify that they are indeed are a part of the whitelist.

These proofs are called `Merkle Proofs`

## TODO Explain Merkle proofs here

Lets see how all this works

- To setup a Hardhat project, Open up a terminal and execute these commands

  ```bash
  npm init --yes
  npm install --save-dev hardhat
  ```
  
- If you are on a Windows machine, please do this extra step and install these libraries as well :)

  ```bash
  npm install --save-dev @nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers
  ```

- In the same directory where you installed Hardhat run:

  ```bash
  npx hardhat
  ```

  - Select `Create a basic sample project`
  - Press enter for the already specified `Hardhat Project root`
  - Press enter for the question on if you want to add a `.gitignore`
  - Press enter for `Do you want to install this sample project's dependencies with npm (@nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers)?`

Now your hardhat folder is set-up.

We also need to install some extra dependecies to execute everything. So again in your terminal pointing to root dirctory execute the following command:

```bash
npm install @openzeppelin/contracts keccak256 merkletreejs
```

Now start by creating a file inside your `contracts` folder named `Whitelist.sol` and add the following lines of code to it

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";

contract Whitelist {

    bytes32 public merkleRoot;

    constructor(bytes32 _merkleRoot) {
        merkleRoot = _merkleRoot;
    }

    function checkInWhitelist(bytes32[] calldata proof, uint64 maxAllowanceToMint) view public returns(bool) {
        bytes32 leaf = keccak256(abi.encode(msg.sender,maxAllowanceToMint));
        bool verified = MerkleProof.verify(proof, merkleRoot, leaf);
        return verified;
    }
    
}
```

Whats exactly happening here? So as we mentioned we are not storing the address of each user in the contract, instead we are only storing the root of the merkle tree. It gets initialized in the contructor.

We also have another function `checkInWhitelist` which takes in a `proof` and `maxAllowanceToMint`. 
`maxAllowanceToMint` is a variable which keeps track of the number of NFT's a given address can mint.

The hash of the leaf node on which this address exists can be computed by first converting 

# TODO Complete the exaplanation

Next lets write a test which can help determine if the code in our contract actually works.

Inside your `test` folder create a new file `merkle-root.js` and add the following lines of code to it

```javascript
const { expect } = require("chai");
const { ethers } = require("hardhat");
const keccak256 = require("keccak256");
const { MerkleTree } = require("merkletreejs");

describe("Check if merkle root is working", function () {
  it("Should be able to verify if the a given address is in whitelist or not", async function () {
    const [owner, addr1, addr2, addr3, addr4, addr5] =
      await ethers.getSigners();
    
    const list = [
      encodeLeaf(owner.address, 2),
      encodeLeaf(addr1.address, 2),
      encodeLeaf(addr2.address, 2),
      encodeLeaf(addr3.address, 2),
      encodeLeaf(addr4.address, 2),
      encodeLeaf(addr5.address, 2),
    ];

    const merkleTree = new MerkleTree(list, keccak256, {
      hashLeaves: true,
      sortPairs: true,
    });
    const root = merkleTree.getHexRoot();

    const whitelist = await ethers.getContractFactory("Whitelist");
    const Whitelist = await whitelist.deploy(root);
    await Whitelist.deployed();

    const leaf = keccak256(list[0]);
    const proof = merkleTree.getHexProof(leaf);

    let verified = await Whitelist.checkInWhitelist(proof, 2);
    expect(verified).to.equal(true);
    verified = await Whitelist.checkInWhitelist([], 2);
    expect(verified).to.equal(false);
  });
});

function encodeLeaf(address, spots) {
  return ethers.utils.defaultAbiCoder.encode(
    ["address", "uint64"],
    [address, spots]
  );
}
```


To run the test, execute the following command from the root of the directory:

```bash
npx hardhat test
```


If all your tests pass, you have successfully learnt what a merkle tree is and how it can be used for whitelisting 🥳 🥳 🥳

Hope you had fun!!

Cheers 🥂


# Contributors
**This module was built in collaboration with [Hypotenuse Labs](https://hypotenuse.ca/)**
