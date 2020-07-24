# 8. Memory Pool
## 8.1 Parameter List

|Parameter item | Parameter description|<div style="width:60pt">effect</div>
|:----:|:----:|:----:|
|pool.pending.maxcount | pending queue upper limit|3w
|pool.queued.maxcount|queued queue upper limit|6w
|pool.clear.days|transaction expiration time in queued and pending|2hours
|min.procedurefee|minimum handling charge for transactions entering memory pool|0.002wdc
|wisdom.ipc-config.queued_to_pending_cycle|write cycle from queued to pending|5seconds
|wisdom.ipc-config.clear-cycle|cleaning cycle from queued to pending|1minutes
|pool.queuedtopending.maxcount|maximum number of transactions per time from queued to pending|5000

## 8.2 Memory Queue
&#160;&#160;&#160;&#160;&#160;&#160;The transaction memory queue is divided into queued and pending

&#160;&#160;&#160;&#160;&#160;&#160;The structure of queue storage is TransPool

|<div style="width:50pt">Number</div> |<div style="width:80pt">Field</div>|<div style="width:120pt">Length/Type</div>|Explanation
|:----:|:----:|:----:|---
|1|transaction|Encapsulating objects|Transaction object
|2|state|4bytes|Transaction status, 0 is to be confirmed, 1 to be referenced, 2 to be acknowledged
|3|datetime|8bytes|Time to enter pending, millisecond
|4|height|8bytes|Block height. Only when the status is 1 and 2, there will be block height

&#160;&#160;&#160;&#160;&#160;&#160;**queued queue：**

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;The queued queue is a transaction queue waiting to be packed. When the node RPC receives the transaction, it will verify the transaction. If the correct transaction is verified, it will enter the queued queue, and the upper limit of the queued queue will be 60000.
The same account address sends two identical nonce transactions. Only if the transaction type is the same, the queued transaction will be covered, and the last queued transaction will cover the forward queued transaction.

&#160;&#160;&#160;&#160;&#160;&#160;**pendin queue：**

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;The pending queue is a transaction queue to be packed. Transactions are written from queued to pending every 5 seconds. Transactions will be checked again before entering pending. Those that fail the verification will be directly deleted in queued. The maximum number of pending transactions written each time is 5000. The transaction status of the transactions entering pending is 0 by default, and the corresponding transactions will be deleted by queued.

&#160;&#160;&#160;&#160;&#160;&#160; &#160;&#160;&#160;&#160;&#160;&#160;The transactions packaged by the miner node are all obtained from the pending queue. When the node is packaged, the transaction will be verified. If the verification is successful, it will be packed into the block, and the unsuccessful transaction will be directly deleted in the pending.After the package is successful, the corresponding transaction status will be changed to 1. Only when the number of block confirmations is satisfied and the DB is successfully written, the status is changed to 2 and wait for clearing.

## 8.3 Priority Principle
&#160;&#160;&#160;&#160;&#160;&#160;In memory queues, there is a principle that transactions are always sorted according to nonce. Therefore, different accounts in the queue are sorted in ascending order according to nonce, that is, from small to large. In this way, transactions with small nonce will be packed first, which conforms to the nonce mechanism.

## 8.4 Local Storage
&#160;&#160;&#160;&#160;&#160;&#160;In order to prevent the unexpected termination of the node or other reasons causing the node to stop, the node will regularly store the unpackaged transactions in queued and pending, and serialize them in the K-V database regularly every 10 minutes.When the node starts, it initializes the last serialized transaction in the transaction memory pool, and the miner node gets the correct transaction package from it.

&#160;&#160;&#160;&#160;&#160;&#160;When the node starts, it first obtains the serialized unpacked transactions from the K-V database and deserializes them to the queued and pending transaction memory pools respectively. The transactions that pass the verification will be packed normally, and those that fail to verify will be deleted immediately.
