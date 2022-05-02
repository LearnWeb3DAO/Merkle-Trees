# Merkle Trees

Merkle Trees are a fundamental concept in blockchain technology.

## What is a Merkle Tree?

A Merkle tree is a type of hash tree in which each leaf node is labeled with the cryptographic hash of a data block, and each non-leaf node is labeled with the cryptographic hash of its child nodes' labels. The majority of hash tree implementations are binary (each node has two child nodes), but they can also have many more child nodes.

A typical Merkle Tree looks something like this:
![](https://upload.wikimedia.org/wikipedia/commons/thumb/9/95/Hash_Tree.svg/1920px-Hash_Tree.svg.png)

(Reference from [using-merkle-trees-for-nft-whitelists](https://medium.com/@ItsCuzzo/using-merkle-trees-for-nft-whitelists-523b58ada3f9))

## Simple Example

Let's say we have 4 transactions: "Transaction A", B, C and D. All of them are executed in the same block. Each of these transactions is going to get hashed. Let's call those hashes "Hash A", B, C, and D respectively.

The following would be the resulting Merkle Tree of these transactions:

![](https://i.imgur.com/QeUy35i.jpg)

## Verifying Validity using the Merkle Root

When these transactions get rolled up into a block, the block header would contain the Merkle Root, Hash ABCD. All miners have a copy of all transactions so far, and therefore all the transaction hashes. Any miner can rebuild the Merkle tree on-demand, which means that every miner can independently arrive at the same Merkle root for the same set of transactions.

This allows any miner to verify a fraudulent transaction. Let's say someone tries to introduce a false transaction instead of Transaction D. Let's call this Transaction E. Because this transaction is different from Transaction D, the hash is going to be different as well. The hash of Transaction E is Hash E. The Hash of C and E together is Hash CE, which is different from Hash CD. When Hash AB and CE are hashed together, you get Hash ABCE. Since hash ABCE is different from Hash ABCD, we can conclude that Transaction E is fraudulent.

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

If you have two different sets of transactions, verifying they are the same with a Merkle Tree is faster than verifying each and every single individual transaction to each other. One can verify that a block has not been modified by only knowing the Merkle Root.

## Use cases outside of the blockchain

Merkle Trees aren't just used in blockchain applications. Some popular applications that use Merkle Trees are:

- [IPFS](https://en.wikipedia.org/wiki/InterPlanetary_File_System)
- [Git](https://github.com)
- Distributed databases such as [AWS DynamoDB](https://aws.amazon.com/dynamodb) and [Apache Cassandra](https://cassandra.apache.org/_/index.html) use Merkle trees to control discrepancies


## Build 

We will try to use Merkle trees for Whitelisting addressees for an NFT Collection.

The reason why Merkle trees will play an important role here is that we will not add each address individually to the whitelist but will only save the root of the Merkle tree within our contract. This will help save gas for us as we don't have to manage to add each address to the whitelist.

So now when the user will try to mint using their whitelist spot, they will submit proof which will verify that they are indeed a part of the whitelist.

These proofs are called `Merkle Proofs` 

Imagine you have to prove that a value `K` exists in the data. Now you can create `H[k]` by hashing k, in our example, we will use `keccak256` algorithm for hashing.

A verifier who only has the root hash `r` can be given a `K` and an associated Merkle proof which convinces them that H[K] is at a leaf node and was used at that leaf node to compute the root hash `r`.

If a Merkle proof says that K was the at a given leaf node and was used to generate `r`, no attacker can come up with another Merkle proof that says that `K` was actually at a different leaf node. So essentially an attacker cannot come up with a Merkle root `r` and two values of leaf nodes for K. So K can only have one unique leaf node for which the root r can be generated.

![](https://i.imgur.com/XsxMA0b.png)

(Referenced [Merkle Proofs Explained](https://medium.com/crypto-0-nite/merkle-proofs-explained-6dd429623dc5))


To check that the proof provided was indeed valid, all the verifier has to do is put H[K] at a given node and provided only a part of the tree, the verifier will parse the part of the tree and check if the root hash `r` is actually derived and validate the proof.

Again it is important to remember that only one given combination of nodes can generate this unique root `r` because the Merkle tree is a  `collision-resistant hash function` which means it is a hash function that given two inputs is almost impossible to produce the same output.

For our given example, we only need to provide the following nodes to be able to prove that H[K] actually exists in our nodes:


![](https://i.imgur.com/nDe4iYS.png)


Let's see how all this works practically for our whitelist example.

- To set up a Hardhat project, Open up a terminal and execute these commands

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

We also need to install some extra dependencies to execute everything. So again in your terminal pointing to root directory execute the following command:

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

What's exactly happening here? So as we mentioned we are not storing the address of each user in the contract, instead, we are only storing the root of the merkle tree which gets initialized in the constructor.

We also have another function `checkInWhitelist` which takes in a `proof` and `maxAllowanceToMint`. 
`maxAllowanceToMint` is a variable that keeps track of the number of NFT's a given address can mint.

The hash of the leaf node on which this address exists can be computed by first encoding the address of the sender and the `maxAllowanceToMint` into bytes string which further gets passed down to the `keccak256` hash function which requires the hash string to generate the hash.

Now we use the Openzeppelin's MerkleProof contract to verify that the proof sent by the user is indeed valid. Note how Openzeppelin performs the verification on a high level is similar to the verification of the Merkle proof we talked about earlier in the tutorial.

Next, let's write a test that can help determine if the code in our contract actually works.

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

Here we first get some signers using hardhat's extended ethers package for testing.

Then we create a list of nodes which are all converted into byte strings using the `ethers.utils.defaultAbiCoder.encode`

Using the `MerkleTree` class from `merkletreejs` we input our list, specify our hashing function which is going to be `keccak256`, and set the sorting of nodes to `true`

After we create the `Merkle Tree`, we get its root by calling the `getHexRoot` function. We use this root to deploy our `Whitelist` contract.

After our contract is verified, we can call our `checkInWhitelist` by providing the proof. So now here we will check that `(owner.address, 2)` exists in our dataset. To generate the proof, we hash the encoded value of `(owner.address, 2)` and generate a proof using 
`getHexProof` function from `merkletreejs` library.

This proof is then sent in `checkInWhitelist` as an argument which further returns a value of true to signify that `(owner.address, 2)` exists.

To run the test, execute the following command from the root of the directory:

```bash
npx hardhat test
```


If all your tests pass, you have successfully learned what a Merkle tree is and how it can be used for whitelisting 🥳 🥳 🥳

Hope you had fun!!

Cheers 🥂


# Contributors
**This module was built in collaboration with [Hypotenuse Labs](https://hypotenuse.ca/)**
