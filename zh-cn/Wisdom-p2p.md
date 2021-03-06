# 十、P2P网络处理
## 10.1 网络拓扑
&#160;&#160;&#160;&#160;&#160;&#160;公链组成网络是一种全分布式的拓扑结构，节点与节点之间的传输过程更接近“泛洪算法”，但为了防止泛洪过大，TTL大小限制为8，即：交易从某个节点产生，接着广播到临近节点，临近节点一传十十传百，直至传播到全网。每个节点都有自身的唯一标识，该标识遵循Kademlia协议，协议会保证网络处于连通状态。

## 10.2 如何连接到网络
&#160;&#160;&#160;&#160;&#160;&#160;首先，节点需要开启节点发现功能，如果不开启节点发现，只能和官方种子节点连接，不能加入到网络中，开启后再填写节点公网IP地址。

&#160;&#160;&#160;&#160;&#160;&#160;新的节点在启动时，都先连接官方默认的四个种子节点，种子节点为：wisdom://47.74.183.249:9585,wisdom://47.74.216.251:9585,wisdom://47.96.67.155.:9585,wisdom://47.74.86.106:9585,随着节点的运行，节点会连接到更多的邻居节点，根据Kademlia协议，优先与自己距离合适的节点。
## 10.3 节点发现机制
&#160;&#160;&#160;&#160;&#160;&#160;每个节点启动后都拥有节点的椭圆曲线密钥对，默认是Ed25519,Ed25519的公钥长度是32字节私钥长度是32字节。节点在P2P网络中的唯一标识符就是自己的公钥，这个公钥信息以十六进制编码在节点的URI中。例如：

 &#160;&#160;&#160;&#160;&#160;&#160;wisdom://03a5acb1faa4dfe70f8e038e297de499cb258cc00afda2822e27291ed180013bd8@192.168.1.3:9585

&#160;&#160;&#160;&#160;&#160;&#160;其他节点若想在启动时连接这个 节点，可以在种子节点列表里把这个URI加入其中，如果URI中不包含公钥，例如：wisdom://192.168.1.3:9585,节点会在建立连接时获取对方的公钥。

&#160;&#160;&#160;&#160;&#160;&#160;节点发现基于kademlia协议，在此协议中，每个节点都有自己的唯一标识，节点之间可以通过这个标识计算出距离，这个距离和ip地址物理地址无关。根据不同的距离，节点会将其他节点分到不同的桶当中，Ed25519的公钥长度是32个字节，256个bit,那桶的数量就有256个。kademlia协议可以保证当节点的邻居节点均匀分布到不同的桶当中时，可以最大化网络的连通性。

&#160;&#160;&#160;&#160;&#160;&#160;每个节点每隔15秒会向其他节点发送一个PING消息，其它节点在收到PING消息后会回复PONG消息，节点A收到节点B的任何类型的消息都会增加节点B的分值，这个分值的半衰期同样是15秒，在A和B节点持续保持连接的状态下，各自的分值始终都是大于0的。

&#160;&#160;&#160;&#160;&#160;&#160;当节点B主动断开与A的连接后，B的分值会快速衰减至0，当B的分值为0后，节点A认为节点B已长时间处于离线状态，将节点B从邻居节点列表中删除。
## 10.4 节点同步机制
&#160;&#160;&#160;&#160;&#160;&#160;节点与节点之间，定时随机选取一个邻居节点，发送自己的状态信息，状态信息中包含了当区块的最高高度，如果发现该邻居节点的最新区块高度高于自己，节点会向该区块请求到这最新高度下的区块，最后该邻居节点接收到请求后，发送一定数量的区块给节点，节点接收到区块后进行区块校验，此协议能够保证所有节点和邻居节点之间区块高度的同步。如果发现该邻居节点区块高度小于或等于自己，就断开连接，等待下个定时任务再发节点状态信息。

&#160;&#160;&#160;&#160;&#160;&#160;而最新的区块，则有矿工节点在出块后广播给所有的邻居节点，而其他邻居节点收到最新区块后继续广播给自己的邻居节点，让最新的区块可以很快的广播并覆盖到整个网络。
## 10.5 网络参数

|参数项| 参数说明|作用
|:----:|:----:|---
|<div style="width:145pt">p2p.mode</div> | p2p通讯模式|grpc,rest已禁用
|p2p.packet-cacheSize| lru缓存大小，防止p2p广播事务过量泛洪|256
|p2p.address|节点地址|设置自己的节点地址 格式为wisdom://ip地址:端口
|p2p.bootstraps|官方种子节点地址|格式和节点地址一致，用逗号隔开，默认是wisdom://47.74.183.249:9585,wisdom://47.74.216.251:9585,wisdom://47.96.67.155:9585,wisdom://47.74.86.106:9585
|p2p.enable-discovery|节点发现|true开启（默认），false关闭
|p2p.max-blocks-per-transfer|最大的区块传输数量|256
|p2p.enable-message-log|消息日志|true开启，false关闭（默认）