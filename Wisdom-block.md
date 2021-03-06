# 4. Block Structure
## 4.1 Block Header

|<div style="width:60pt">Number</div> | Parameter | Effect | Type
|:----:|:----:|---|:----:
|1  | <div style="width:110pt">nVersion</div> |Block version number. The default value is 0|<div style="width:100pt">Unsigned 32-bit integer</div>
|2  | hashPrevBlock |Hash value of the previous block|32byte
|3  |hashMerkleRoot|Transaction merkle root|32byte
|4 | hashMerkleState|Merkle Tree root of account status|32byte
|5 |hashMerklelncubate|Merkle root of incubating affairs(Referenced. All fill in 0)|32byte
|6 |nHeight|Block height|Unsigned 32-bit integer
|7 |nTime|Block timestamp era, the unit is seconds|Unsigned 32-bit integer
|8 |nBits|Target difficulty value|32byte
|9 |nNonce|Random number of completed workload proof|32byte
|10 |blockNotice|Witness phrases are converted into byte arrays by using UTF-8 encoding, and are also deserialized by using UTF-8 when reading contents. Note that this field does not participate in the hash calculation of the block header, and is not within the scope of verification. Now, only genesis block is used, and other blocks display null|32byte

## 4.2 Block Body
&#160;&#160;&#160;&#160;&#160;&#160;The block body contains multiple transactions, which can be divided into two types：
        
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;1.The coinbase transaction is a transaction automatically generated by miner node, which contains public key hash of blocker and miner reward. In the block body, it always ranks first, and there is only one；


&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;2.Other transactions, which are broadcast transactions in the public chain, are constructed by users and broadcast in the p2p network. The miner node obtains the transaction in the transaction memory pool and packs it into the block when output the block;

&#160;&#160;&#160;&#160;&#160;&#160;If there are no other transactions when the miner node is packaged, there are only coinbase transactions in the block body, and the block is referred to as an empty block.

##  4.3 Block Validation
&#160;&#160;&#160;&#160;&#160;&#160;1.Verify  the hash value of block header, hash value of previous block, and whether the version number and height match；

&#160;&#160;&#160;&#160;&#160;&#160;2.Verify the version number of the block header and the size of the whole block；

&#160;&#160;&#160;&#160;&#160;&#160;3.Verify whether the random number in the block meets the workload proof；

&#160;&#160;&#160;&#160;&#160;&#160;4.Verify whether the coinbase transaction of the block is a legal miner；

&#160;&#160;&#160;&#160;&#160;&#160;5.Verify whether the Merkle root in the block header is legal or not；

&#160;&#160;&#160;&#160;&#160;&#160;6.Verify whether the transaction format in the block body is legal and whether the account status is normal.。

## 4.4 Merkle Tree
&#160;&#160;&#160;&#160;&#160;&#160;The Merkle Tree usually contains the underlying (transaction) database of the block body, the root hash value of the block header (that is, the Merkle root), and all branches along the underlying block data to the root hash. Generally, the operation process of Merkle Tree is to hash the block data in groups, and insert the generated hash value into the Merkle Tree, so recursively  until only the last root hash value is left and recorded as the Merkle root of the block header.

&#160;&#160;&#160;&#160;&#160;&#160;In the block body, there are transaction Merkle root, account state Merkle Tree root and incubated transaction Merkle root.
