# 九、共识机制
## 9.1 共识流程
&#160;&#160;&#160;&#160;&#160;&#160;全节点成为矿工节点首先需要抵押10万的WDC才能竞选矿工节点，所有抵押超过10万WDC的地址会按照被投票的权益进行排序，只有投票权益前15名的地址才能成为矿工。每到一个纪元（一纪元=120个区块），以当前投票权益重新排序出前15名矿工节点出块，该纪元内矿工列表不会再重新排序，直到下一个纪元开始才会重新排序，而矿工节点最多只能15个。

&#160;&#160;&#160;&#160;&#160;&#160;区块高度在285600之前，出块平均间隔是30秒，在此高度后，出块平均间隔调整为10秒，现一个纪元的出块平均时间是10*120=1200秒=20分钟。

&#160;&#160;&#160;&#160;&#160;&#160;出块顺序是根据矿工列表出块，当轮到某个矿工出块时，矿工从事务内存池中筛选出合法的事务，打包进区块，通过p2p协议广播到网络中。除此之外，矿工还需要完成一定的工作量证明，矿工需要在30秒内解决哈希值计算难题，找到一个随机数满足区块的哈希值小于当前纪元的难度值，难度值每过一个纪元调整一次。

&#160;&#160;&#160;&#160;&#160;&#160;每个矿工出块间隔最多30秒，如果该矿工在30秒内没有成功出块，会直接跳过该矿工出块，按照顺序安排下一个矿工出块。
## 9.2 密码算法
&#160;&#160;&#160;&#160;&#160;&#160;共识用到的哈希算法有：

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;WHIRLPOOL

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;RIPEMD-256

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;BLAKE2B-256

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;SHA3-256

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;KECCAK-256

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;SKEIN256-256
## 9.3 见证者

&#160;&#160;&#160;&#160;&#160;&#160;见证者需要满足的条件有

&#160;&#160;&#160;&#160;&#160;&#160;1、抵押至少10万的WDC

&#160;&#160;&#160;&#160;&#160;&#160;2、投票权益排在前15名

&#160;&#160;&#160;&#160;&#160;&#160;3、投票数不能为0
## 9.4 主链规范化
&#160;&#160;&#160;&#160;&#160;&#160;节点收到区块后会写入到ForkDB中临时保存，当区块收到2/3矿工确认后，也就是在这个区块后又接收到了一定数量的合法区块，这部分的区块的出块者满足该纪元中2/3矿工列表的矿工，这就是2/3矿工确认。那该区块被永久确认，写入磁盘。

&#160;&#160;&#160;&#160;&#160;&#160;节点具有一定的临时分叉容错能力（可能性比较低），但是如果两个节点发生了硬分叉，也就是两个节点在同一高度上存在不同的被确认的区块，此时需要手动删除硬分叉的区块，并保留其中一个节点的分支。或者运行WDC官方提供的运维工具，会自动检测节点是否分叉，如果节点出现分叉，会自动修复保留高度最高的那条分支。
## 9.5 惩罚机制
&#160;&#160;&#160;&#160;&#160;&#160;在一个纪元内，矿工没有出一个区块，该矿工地址会被永久拉黑，无法再成为矿工。若想该节点再次成为矿工节点，需要把被拉黑矿工地址上的抵押撤回，把投票也撤回，重新生成一个新的矿工地址，转移到新的矿工地址上，下个纪元满足矿工要求，就可以继续出块了。