# 9. Consensus Mechanism
## 9.1 Consensus Process
&#160;&#160;&#160;&#160;&#160;&#160;All nodes need to mortgage 100000 WDC to run for miner node first. All WDC addresses with mortgage of more than 100000 will be sorted according to the voting rights and interests. Only the top 15 addresses with voting rights can become miners. Each era (one era = 120 blocks), the top 15 miners' nodes will be reordered according to the current voting rights. The list of miners in this era will not be reordered until the beginning of the next era, and the maximum number of miners' nodes can be 15.

&#160;&#160;&#160;&#160;&#160;&#160;Before the block height of 285600, the average interval between blocks is 30 seconds. After this height, the average interval of blocks is adjusted to 10 seconds. The average time of block production in the current era is 10 * 120 = 1200 seconds = 20 minutes.

&#160;&#160;&#160;&#160;&#160;&#160;The block sequence is based on the miners list. When a miner comes out block, the miners select legitimate transactions from the transactional memory pool, pack them into blocks, and broadcast them through the p2p protocol to the network. In addition, the miners also need to complete a certain workload proof. Miners need to solve hash calculation problems within 30 seconds. Find a random number to meet the block hash value less than the current era's difficulty, and the difficulty value is adjusted once every era.

&#160;&#160;&#160;&#160;&#160;&#160;Each miner has an interval of up to 30 seconds to output block. If the miner fails to produce a block within 30 seconds, it will skip the miner directly and arrange the next miner output block in sequence.
## 9.2 Cryptographic Algorithm
&#160;&#160;&#160;&#160;&#160;&#160;The hash algorithms used in consensus are：

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;WHIRLPOOL

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;RIPEMD-256

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;BLAKE2B-256

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;SHA3-256

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;KECCAK-256

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;SKEIN256-256
## 9.3 Witness

&#160;&#160;&#160;&#160;&#160;&#160;The witness needs to meet the following conditions

&#160;&#160;&#160;&#160;&#160;&#160;1. Mortgage at least 100 thousand of WDC

&#160;&#160;&#160;&#160;&#160;&#160;2. Voting rights rank among the top 15.

&#160;&#160;&#160;&#160;&#160;&#160;3. The number of votes can not be 0.
## 9.4 Standardization of Main Chain
&#160;&#160;&#160;&#160;&#160;&#160;After receiving the block, the node will write it to ForkDB for temporary storage. When the block receives confirmation from 2/3 miners, that is, after the block receives a certain number of legal blocks, the blocker of this part of the block meets the miners in the 2/3 miner list in the era, which is the 2/3 miner confirmation. Then the block is permanently acknowledged and written to disk.

&#160;&#160;&#160;&#160;&#160;&#160;The node has a certain temporary bifurcation fault-tolerant ability (the possibility is relatively low), but if two nodes have hard bifurcation, that is, the two nodes have different confirmed blocks at the same height, it is necessary to manually delete the hard forked blocks and keep the branch of one of the nodes. Or run the operation and maintenance tools provided by the WDC official, and automatically detect whether the node is bifurcated. If the node bifurcates, it will automatically repair the branch with the highest reserved height.
## 9.5 Punishment Mechanism
&#160;&#160;&#160;&#160;&#160;&#160;In an era, the miner did not output a block, and the miner's address would be permanently blackened and could no longer be a miner. If the node wants to become a miner node again, it needs to withdraw the mortgage on the address of the black miner, withdraw the vote, regenerate a new miner's address and transfer it to a new miner's address. If the node meets the requirements of miners in the next era, it can continue to output block.
