# 10. P2P Network Processing
## 10.1 Network Topology
&#160;&#160;&#160;&#160;&#160;&#160;The public chain network is a fully distributed topology, and the transmission process between nodes is closer to the "flooding algorithm". However, in order to prevent excessive flooding, the size of TTL is limited to 8, that is, transactions are generated from a certain node, then broadcast to the adjacent nodes, and the neighboring nodes transmit ten to one hundred until they spread to the whole network. Each node has its own unique identity, which follows the Kademlia protocol, which will ensure that the network is connected.

## 10.2 How to connect to the network
&#160;&#160;&#160;&#160;&#160;&#160;First of all, the node needs to turn on the node discovery function. If the node discovery is not enabled, it can only connect with the official seed node and cannot join the network. After the node discovery is enabled, the public IP address of the node can be filled in.

&#160;&#160;&#160;&#160;&#160;&#160;When a new node is started, it first connects to the four official default seed nodes. The seed nodes are:wisdom://47.74.183.249:9585,wisdom://47.74.216.251:9585,wisdom://47.96.67.155.:9585,wisdom://47.74.86.106:9585,with the node running, the node will connect to more neighbor nodes. According to Kademlia protocol, the node with proper distance from itself is preferred.

## 10.3 Node Discovery Mechanism
&#160;&#160;&#160;&#160;&#160;&#160;Each node has its own elliptic curve key pair after starting. The default is Ed25519. The public key length of Ed25519 is 32 bytes, and the private key length is 32 bytes. The unique identifier of a node in P2P network is its public key, which is encoded in the node's URI in hexadecimal. For example:

 &#160;&#160;&#160;&#160;&#160;&#160;wisdom://03a5acb1faa4dfe70f8e038e297de499cb258cc00afda2822e27291ed180013bd8@192.168.1.3:9585

&#160;&#160;&#160;&#160;&#160;&#160;If other nodes want to connect to this node at startup, they can add this URI to the list of seed nodes. If the URI does not contain a public key, for example: wisdom://192.168.1.3:9585, the node will obtain the public key of the other party when establishing the connection.

&#160;&#160;&#160;&#160;&#160;&#160;Node discovery is based on kademlia protocol. In this protocol, each node has its own unique identity. The distance between nodes can be calculated by this identifier, which is independent of the physical address of IP address. According to different distances, nodes will divide other nodes into different buckets. The public key length of Ed25519 is 32 bytes and 256 bits, and the amount of buckets is 256. Kademlia protocol can ensure that when the neighbors of nodes are evenly distributed among different buckets, it can maximize the network connectivity.

&#160;&#160;&#160;&#160;&#160;&#160;Each node will send a PING message to other nodes every 15 seconds, and other nodes will reply PONG message after receiving the PING message. Node A will increase the score of node B when receiving any type of message from node B. The half-life of this score is also 15 seconds. When nodes A and B are continuously connected, their scores are always greater than 0.

&#160;&#160;&#160;&#160;&#160;&#160;When node B takes the initiative to disconnect from A, the score of B will quickly decay to 0. When the score of B is 0, node A thinks that node B has been offline for a long time, and will delete node B from the list of neighbor nodes.
## 10.4 Node Synchronization Mechanism
&#160;&#160;&#160;&#160;&#160;&#160;The node selects a neighbor node randomly and sends its own status information. The status information contains the highest height of the current block. If the latest block height of the neighbor node is higher than itself, the node will request the block under the latest height from the block. Finally, the neighbor node receives the request and sends a certain number of blocks to the node,the node checks the block after receiving the block.This protocol can ensure the block height synchronization between all nodes and neighbor nodes. If the block height of the neighbor node is less than or equal to itself, it will disconnect and wait for the next scheduled task to send the node status information again.

&#160;&#160;&#160;&#160;&#160;&#160;For the latest block, there are miner nodes broadcast to all neighbor nodes after the block is out, while other neighbor nodes continue to broadcast to their neighbors after receiving the latest block, so that the latest block can quickly broadcast and cover the whole network.
## 10.5 Network Parameters

|Parameter item|  <div style="width:155pt">Parameter description</div> |Effect
|:----:|:----:|---
|<div style="width:145pt">p2p.mode</div> | p2p communication mode|grpc,rest disabled
|p2p.packet-cacheSize| lru cache size to prevent excessive flooding of p2p broadcast transactions|256
|p2p.address|node address|set own node address The format is wisdom://ip address: port
|p2p.bootstraps|official seed node address|The format is consistent with the node address, separated by commas. The default is:wisdom://47.74.183.249:9585,wisdom://47.74.216.251:9585,wisdom://47.96.67.155:9585,wisdom://47.74.86.106:9585
|p2p.enable-discovery|node discovery|true is on (default), false is off
|p2p.max-blocks-per-transfer|maximum number of block transfers|256
|p2p.enable-message-log|message log|true is on, false is off (default)
