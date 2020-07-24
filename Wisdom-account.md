# 6. Account Model
## 6.1 Account Structure
&#160;&#160;&#160;&#160;&#160;&#160;The AccountState account object is the abstract encapsulation of the address. For the account address, it is only a simple representation, and it can't carry more data. In the actual process, we need a more advanced account function, which can carry some data that is convenient for statistics.

&#160;&#160;&#160;&#160;&#160;&#160;Basic structure of the AccountState object：

|<div style="width:60pt">Number</div>|<div style="width:70pt">Field</div>|Length/Type|Explanation
|:----:|:----:|:----:|---|
|1| account|Encapsulating objects|Basic information of account
|2|interestMap|ByteArrayMap|Apply for incubation customization Map
|3|ShareMap|ByteArrayMap|Sharing incubation customization Map
|4|type|4bytes|Account type, cannot be empty
|5|Contract|byte[]|Byte data of rule programming (RLP encoding), if the type is not rule programming, here is 0 byte
|6|Tokensnap|ByteArrayMap|Store contract token customize Map. If the type is rule programming, here is empty Map


&#160;&#160;&#160;&#160;&#160;&#160;Account is the basic structure of account basic information object：

|<div style="width:60pt">Number</div> | Field|Field/Type|Explanation
|:----:|:----:|:----:|---|
|1 | blockHeight|8bytes|blockHeight
|2|pubkeyHash|20bytes|Public key hash of the address
|3|nonce|8bytes|Unsigned 64 bit, prevent replay attack from 1
|4|balance|8bytes|WDC balance
|5|incubatecost|8bytes|Incubation principal
|6|mortgage|8bytes|Mortgage amount
|7|vote|8bytes|Number of votes obtained


&#160;&#160;&#160;&#160;&#160;&#160;Incubator is the basic structure of incubation object. In the AccountState object, custom ByteArrayMap is used to store incubation information. Key is the transaction hash applied for incubation, and Value is the Incubator object.

|Number| Field|Field/Type|Explanation
|:----:|:----:|:----:|---|
|~~1~~|~~id~~|~~36bytes~~|~~Primary key ID，Disabled~~
|2|share_pubkeyhash|20bytes|Public key hash of recommender. If there is no recommender, fill in 0
|3|pubkeyhash|20bytes|Incubator public key hash
|4|txid_issue|32bytes|Apply for incubation transaction hash
|5|height|8bytes|block height
|6|cost|8bytes|Incubation principal
|7|interest_ammount|8bytes|Interest balance
|8|share_amount|8bytes|Share balance
|9|last_blockheight_interest|8bytes|Last height of withdrawal interest
|10|last_blockheight_share|8bytes|Last height of withdrawal share income

## 6.2 State Representation
The status of an AccountState account is type and int type, respectively:

&#160;&#160;&#160;&#160;&#160;&#160;0 is a normal address；

&#160;&#160;&#160;&#160;&#160;&#160;1 is the contract token address；

&#160;&#160;&#160;&#160;&#160;&#160;2 is the contract multi signature address；

&#160;&#160;&#160;&#160;&#160;&#160;3 is the contract lock time hash address；

&#160;&#160;&#160;&#160;&#160;&#160;4 is the contract lock height hash address.

## 6.3 nonce mechanism

&#160;&#160;&#160;&#160;&#160;&#160;Nonce exists to prevent nodes from being attacked by replay. The nonce of each account starts from 1. Each account sends a transaction and writes it into the block. Nonce needs nonce + 1. Nonce cannot be repeated, cannot be negative, and can only be a positive integer.

&#160;&#160;&#160;&#160;&#160;&#160;For example, if account A wants to send a transfer transaction, the node RPC is used to check that the current account nonce is 9, and then account A needs to send a transfer transaction with nonce of 10. Nonce cannot be < = 9. If a transaction with nonce < = 9 is sent, the node will reject the transaction;

&#160;&#160;&#160;&#160;&#160;&#160;If account A sends a transaction with nonce = 11, this phenomenon is called nonce skip. Nodes can normally receive and package nonce skips into blocks, but nodes will have a penalty mechanism for nonce skip.Under normal circumstances, the node will be immediately packed if it receives the qualified transaction. However, for the nonce skip transaction, the packet will be delayed. If the current public chain miner node is 15 block miners, it needs to delay 15 * 7 = 105 blocks before being packed into the block, that is, delay packing penalty.
