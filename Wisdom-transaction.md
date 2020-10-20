# 5. Transaction Structure
## 5.1 Transaction Type

| Number | Transaction type|Explaination
| :----:|:----:|:----:
| 1 | 0x00|coinbase transaction
| 2 | 0x01|WDC transfer
| 3 | 0x02|vote
| 4 | 0x03|certificate storage transaction（deprecated）
| 5 | 0x07|rule deployment（deprecated）
| 6 | 0x08|rule call（deprecated）
| 7 | 0x09|apply for incubation
| 8 | 0x0a|gain benefits
| 9 | 0x0b|gain share income
| 10 | 0x0c|withdraw principal
| 11 | 0x0d|withdraw  vote
| 12 | 0x0e|mortgage
| 13 | 0x0f|withdraw mortgage
| 14 | 0x10 | wasm contract deployment
| 15 | 0x11 | wasm call contract

##  5.2 Transaction Structure

| <div style="width:60pt">Number</div> | Field name|Effect|Type
| :----:|:----:|---|:----:|
|1| <div style="width:60pt">version</div>|transaction version|<div style="width:90pt">1byte</div>
| 2 | txHash |transaction hash|32byteSHA3-256
| 3 |type|transaction type|1byte
| 4 |nonce|prevent replay attacks|unsigned 64 bit t
| 5|from|issuer public key|32byte
| 6|gasPrice|service charge unit price, the actual service charge is the unit price multiplied by Gas value|unsigned 64 bit
| 7|amount|transfer amount, according to different transaction types, the transfer amount can represent ordinary transfer, incubation principal, interest extraction, shared income extraction, etc|unsigned 64 bit
| 8|signature|signature of signer|byte[]
| 9|to|The receiver's public key hash, if it is certificate storage and rule deployment, fill in all zeros; if it is a vote, fill in the target's public key hash; if it is a rule call, fill in the public key hash of rule programming|20byteHash160
| 10|payloadLen|byte length|4byte
| 11|payload|Byte array, which can store certificate data, incubation script and certificate data; |byte[]

##  5.3 About payload
&#160;&#160;&#160;&#160;&#160;&#160;For different transaction types, the storage data in payload is different, but the storage type is the same, which is byte array. Except for coinbase transaction, WDC transfer, voting and mortgage, there is no payload. The following is a description of payloads for other types of transactions：

&#160;&#160;&#160;&#160;&#160;&#160;Certificate of deposit: storing the certificate data, the encoding format is officially recommended to use UTF-8, otherwise the data will be confused when viewed on the official browser；

&#160;&#160;&#160;&#160;&#160;&#160;Rule deployment: stores the rule deployment data, and its payload is divided into two sections. The first segment represents the rule type, 1 byte, and the second segment is the rule deployment data in RLP encoding format；

&#160;&#160;&#160;&#160;&#160;&#160;Rule call: stores the rule call data, and its payload is divided into two sections. The first segment represents the type of calling rule, 1 byte, and the second segment is rule deployment data, RLP code lattice；

&#160;&#160;&#160;&#160;&#160;&#160;Apply for Incubation: store incubation data, defined by Protobuf, in the following format. The transaction has been disabled;

&#160;&#160;&#160;&#160;&#160;&#160;{

&#160;&#160;&#160;&#160;&#160;&#160; &#160;&#160;&#160;&#160;&#160;&#160;   "txid": //When applying for hatching, it is blank. If it is added later, fill in 0 when it is blank

&#160;&#160;&#160;&#160;&#160;&#160; &#160;&#160;&#160;&#160;&#160;&#160;    "sharepubkeyhash":Hexadecimal hash value

&#160;&#160;&#160;&#160;&#160;&#160; &#160;&#160;&#160;&#160;&#160;&#160;    "type"：120,//value give a byte

&#160;&#160;&#160;&#160;&#160;&#160;}

&#160;&#160;&#160;&#160;&#160;&#160;Obtain interest income / share income / withdraw principal: deposit application incubation transaction hash, 32 bytes；

&#160;&#160;&#160;&#160;&#160;&#160;Withdraw vote: store the voting transaction hash, 32 bytes；

&#160;&#160;&#160;&#160;&#160;&#160;Withdraw Mortgage: store the mortgage transaction hash, 32 bytes.

&#160;&#160;&#160;&#160;&#160;&#160;Contract deployment：

```python
    payload = rlp(gaslimit, bytecode, args, abi) 
    # payload is the RLP encoded result of the gasLimit (integer), WebAssembly bytecode, constructor parameters (optional), and ABI in series
```

&#160;&#160;&#160;&#160;&#160;&#160;Contract call

```python
    payload = rlp(gaslimit, method, args) 
    # payload is the result of the gasLimit (integer), the method name of the calling contract, the parameter of the calling contract in series, and the RLP encoding
```