# 八、内存池
## 8.1 参数列表

|参数项 | 参数说明|作用
|:----:|:----:|:----:|
|pool.pending.maxcount | pending队列上限|3w
|pool.queued.maxcount| queued队列上限|6w
|pool.clear.days|queued和pending中事务过期时间|2小时
|min.procedurefee|事务进入内存池的最低手续费|0.002wdc
|wisdom.ipc-config.queued_to_pending_cycle|queued到pending的写入周期|5秒
|wisdom.ipc-config.clear-cycle|queued到pending的清理周期|1分钟
|pool.queuedtopending.maxcount|每次queued到pending最大事务数|5000

## 8.2 内存队列
&#160;&#160;&#160;&#160;&#160;&#160;事务内存队列分为queued和pending

&#160;&#160;&#160;&#160;&#160;&#160;其中队列存储的结构是TransPool

|编号|字段|长度/类型|说明
|:----:|:----:|:----:|---
|1|transaction|封装对象|事务对象
|2|state|4字节|事务状态，0是待确认，1是被引用，2是已确认
|3|datetime|8字节|进入pending的时间，毫秒
|4|height|8字节|区块高度，只有状态是1和2时，才会有区块高度

&#160;&#160;&#160;&#160;&#160;&#160;**queued队列：**

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;queued队列是等待打包的事务队列，当节点RPC接收事务时，会对事务进行校验，校验正确的事务，会进入到queued队列，queued队列上限60000。
同一个账户地址发送两条相同nonce事务，只有事务类型相同，queued中才会被覆盖，后进queued会覆盖前进queued的事务。

&#160;&#160;&#160;&#160;&#160;&#160;**pending队列：**

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;pending队列是准备打包的事务队列，每隔5s从queued到pending写入事务，事务在进入pending前会再次进行校验，校验不通过的，会在queued
中直接删除，每次写入pending事务最大5000条，进入到pending的事务，事务状态默认是0，queued也会删除相应的事务。

&#160;&#160;&#160;&#160;&#160;&#160; &#160;&#160;&#160;&#160;&#160;&#160;矿工节点打包的事务，都是从pending列队中获取，节点在打包时还会校验事务，校验成功的会被打包进区块，不成功的事务在pending中会直接被删
除，打包成功后，相应的事务状态会修改成1，只有满足区块确认数后写入db成功的，状态修改成2后等待清除。

## 8.3 优先原则
&#160;&#160;&#160;&#160;&#160;&#160;在内存队列中一直遵循一个原则，事务排序永远都是根据nonce来排序的，所以队列中不同账户都是按照的nonce升序的，也就是从小到大，这样打包时会优先打包nonce小的事务，符合nonce机制。

## 8.4 本地存储
&#160;&#160;&#160;&#160;&#160;&#160;为了防止节点意外终止进程或其他原因导致节点停止，节点会定时存储queued和pending中未打包事务，每10分钟定时序列化在K-V型数据库中。

&#160;&#160;&#160;&#160;&#160;&#160;而节点在启动时，会把最后一次序列化的事务初始化在事务内存池中，矿工节点再从中获取正确的事务打包。