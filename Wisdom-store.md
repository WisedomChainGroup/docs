# 7. Storage Structure
## 7.1 Block Storage
&#160;&#160;&#160;&#160;&#160;&#160;The block is stored in PostgreSQL relational database with version number of 10, and its data tables are header, transaction and transaction_ index.

&#160;&#160;&#160;&#160;&#160;&#160;Header table, record block header information

|Number | Field|Type|Explanation|
|:----:|:----:|:----:|:----:|
|1|block_hash|bytea|block hash
|2|block_notice|bytea|notes
|3|is_canonical|boolean|Whether it is the main chain block or not. The default is yes
|4|created_at|bigint|Creation time
|5|hash_merkle_incubate|bytea|Incubation state Merkle root
|6|hash_merkle_root|bytea|Merkle root
|7|hash_merkle_state|bytea|Merkle state
|8|hash_prev_block|bytea|Previous block hash
|9|height|bigint|height
|10|nbits|bytea|Target difficulty
|11|nonce|bytea|Serial number
|12|total_weight|bigint|Total block weight
|13|version|smallint|Version number

&#160;&#160;&#160;&#160;&#160;&#160;transaction table to record transaction details

|Number | Field|Type|Explanation
|:----:|:----:|:----:|:----:|
|1|tx_hash| bytea|Transaction hash
|2|amount|bigint|balance
|3|from|bytea|Public key of transaction from
|4|gas_price|bigint|gas unit Price
|5|nonce|bigint|Serial number
|6|payload|bytea|Different transactions store different data
|7|signature|bytea|Transaction signature
|8|to|bytae|Public key hash of transaction to
|9|type|smallint|Transaction type
|10|version|smallint|Version number

&#160;&#160;&#160;&#160;&#160;&#160;transaction_index table, association table of transactions and blocks

|Number | Field|Type|Explanation
|:----:|:----:|:----:|:----:|
|1|block_hash|bytea|block hash
|2|tx_hash|bytea|Transaction hash
|3|tx_index|integer|Transaction number in block

## 7.2 State Storage
&#160;&#160;&#160;&#160;&#160;&#160;The collection of accounts constitutes the world state, and accounts are organized in the form of Merkle Patricia Tree (MPT) and stored in the K-V database.

&#160;&#160;&#160;&#160;&#160;&#160;Each account is located in the leaf node of the tree, and the organization of the tree is hashed in series according to the arrangement order, and finally the world state is obtained by layer by layer hash. When a single account is changed, the upper hash value of its branch will be changed until the hash value of the root node is affected. The hash value of the root node is called the status tree, which will be stored in the local storage. The state of the world changes with the progress of the blockchain, and the value of the state tree also changes.

## 7.3 Retrieval Structure

![State tree](img/search.png)

 &#160;&#160;&#160;&#160;&#160;&#160;Prefix Tree is a well-known data structure for storing ordered strings. The main advantage of MPT over ordinary prefix tree is its reduced storage. If we search in the Patricia Tree, we will search the letters of the string in turn until we get a complete path (in the correct order). If a null pointer is encountered before all the letters in the string (the target) are retrieved, the string is said to be not in the prefix tree. On the other hand, if we reach a leaf node (branch end node) with the retrieval, then the path represents the target string, which can be considered as in the prefix tree.

&#160;&#160;&#160;&#160;&#160;&#160;For example, in the above figure, we can find "dog" through the string [3,15,3,13,4,10], and the hexadecimal encoding form of cat is [3,15,3,13,4,10], so we think there is a key value mapping from cat to dog in this tree.
