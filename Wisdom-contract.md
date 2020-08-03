# 12. Programmable Verification Rules
## 12.1 Rule Type
&#160;&#160;&#160;&#160;&#160;&#160;Because the program function in the public chain is clearly specified and the format is strictly defined, the program composition here is actually more of a template format definition, and the function is clear.

&#160;&#160;&#160;&#160;&#160;&#160;Five rule definitions are supported:

&#160;&#160;&#160;&#160;&#160;&#160;1、Asset definition-ASSET

&#160;&#160;&#160;&#160;&#160;&#160;2、Multi-signature-MULTISIGN

&#160;&#160;&#160;&#160;&#160;&#160;3、Hash time lock payment-HASHTIMELOCK

&#160;&#160;&#160;&#160;&#160;&#160;4、Hash height lock payment-HASHHEIGHTLOCK
## 12.2 Rule Deployment
&#160;&#160;&#160;&#160;&#160;&#160;All the above rule types have a fixed rule definition format, and the node core is supported by source level hard coding. Therefore, users only need to fill in their own attribute values and dynamic values in the rules.

&#160;&#160;&#160;&#160;&#160;&#160;The process of deploying rules, that is, writing users' own rules to blocks, is completed through a special transaction. The transaction format is as follows:

|<div style="width:45pt">Number</div> |<div style="width:60pt">Field name</div>|Effect|<div style="width:95pt">type</div>
| ---- |:----:|:----:|:----:|
|1| version|transaction version|1byte|
|2| txHash|transaction hash|32bytes SHA3-256
|3|type|transaction type|1byte
|4|nonce|prevent replay attack|unsigned 64 bit
|5|from|issuer public key|32bytes
|6|gasPrice|Service charge unit price, the actual service charge is the unit price multiplied by Gas value|unsigned 64 bit
|7|amount|fill in 0|unsigned 64 bit
|8|signature|signature of signer|byte[]
|9|to|fill in all 0|20bytes Hash160
|10|payloadLen|byte length|4bytes
|11|payload|regular program in byte format|byte[]

&#160;&#160;&#160;&#160;&#160;&#160;<font color=red>Note Item 9 To, fill in 0 when deploying</font>

&#160;&#160;&#160;&#160;&#160;&#160;After the transaction is written to the block ledger, the corresponding transaction hash can return an address according to the generation logic of the address, which is equivalent to the contract address. In this way, the contract address, like the account address, is in a unified account model.

&#160;&#160;&#160;&#160;&#160;&#160;In order to distinguish the contract address from the general account address, the contract address prefix is set to two upper case WR characters.
## 12.3 Rule Call
&#160;&#160;&#160;&#160;&#160;&#160;Each type of rule contains several rule statements. The rule statements can be called by signature outside. The transaction format is as follows:

|<div style="width:45pt">Number</div> |<div style="width:60pt">Field name</div>|Effect|<div style="width:95pt">type</div>
:----:|:----:|:----:|:----:
1| version|transaction version|1byte
2| txHash|transaction hash|32bytes SHA3-256
3|type|transaction type|1byte
4|nonce|prevent replay attack|unsigned 64 bit
5|from|issuer public key|32bytes
6|gasPrice|Service charge unit price, the actual service charge is the unit price multiplied by Gas value|unsigned 64 bit
7|amount|fill in 0|unsigned 64 bit
8|signature|signature of signer|byte[]
9|to|160 hash value of the contract address|20bytes Hash160
10|payloadLen|byte length|4bytes
11|payload|calling rules in byte format|byte[]

&#160;&#160;&#160;&#160;&#160;&#160; <font color=red>Note that the contract address of Item 9 is consistent with the description in rule deployment</font>

## 12.4 Asset Definition
### 12.4.1 Asset Rules
Grammatical format

```json
{
    "ASSET_RULE":{
        "code":"WDCC",
        "offering":10000,
        "totalamount":10000,
        "create":"xxxx",
        "owner":"yyyy",
        "allowincrease":"0",
        "info":{}    //300bytes/nullable
    }
}
```

&#160;&#160;&#160;&#160;&#160;&#160;The above is the rule program defined for assets, and supports the following rule function calls:

|Number| Rule function|Notes
|:----:|---|:----:|
|1|```"changeowner":{ "newowner":"ddddd" }```|Change owner public key
|2|```"transfer":{ "from":"Public key", "to":"Public key hash", "value":"50" }```|Forwarding assets
|3|```"increased":{ "amount":500 }```|whether to issue additional or not

## 12.4.2  Rule Attribute
1. code，Asset code

&#160;&#160;&#160;&#160;&#160;&#160;test points：

&#160;&#160;&#160;&#160;&#160;&#160;1) The asset code must be greater than or equal to 3 characters and less than or equal to 12 characters

&#160;&#160;&#160;&#160;&#160;&#160;2) It cannot contain special characters such as space / soft carriage return

&#160;&#160;&#160;&#160;&#160;&#160;3) Must be all English characters

&#160;&#160;&#160;&#160;&#160;&#160;4) Characters must be all uppercase

&#160;&#160;&#160;&#160;&#160;&#160;5) It can't be WDC

&#160;&#160;&#160;&#160;&#160;&#160;6) Cannot duplicate conflict with existing code

&#160;&#160;&#160;&#160;&#160;&#160;Byte length：6

2. offering，Initial issuance amount

&#160;&#160;&#160;&#160;&#160;&#160;test points：

&#160;&#160;&#160;&#160;&#160;&#160;1) Must be an integer

&#160;&#160;&#160;&#160;&#160;&#160;2) The value range is in the range of  long type

&#160;&#160;&#160;&#160;&#160;&#160;3) Cannot be less than or equal to 0

&#160;&#160;&#160;&#160;&#160;&#160;Byte length：8

3. totalamount，Total issue amount

&#160;&#160;&#160;&#160;&#160;&#160;test points:

&#160;&#160;&#160;&#160;&#160;&#160;1) Must be an integer

&#160;&#160;&#160;&#160;&#160;&#160;2) The value range is in the range of  long type

&#160;&#160;&#160;&#160;&#160;&#160;3) Cannot be less than or equal to 0

&#160;&#160;&#160;&#160;&#160;&#160;4) By default, it is the same as "initial issue amount" during deployment

&#160;&#160;&#160;&#160;&#160;&#160;Byte length：8

4. create，Public key of rule Creator

5. owner，Public key hash of the rule owner

6. allowincrease，Whether additional issue is allowed or not: 1 means allowed, 0 means not allowed

7. By default, the precision remains the same as the default precision of the main chain, and the maximum decimal point is 8 digits

## 12.4.3 General Verification
1) When calling a rule function, it is necessary to confirm whether the rule function is included in the corresponding rule program to prevent miscalling, such as calling the function of asset rule for multiple signature rules;

2) The calling syntax of a rule function must be legal and cannot contain redundant content;

3) Call data is also uses RLP coding.

## 12.4.4 Asset Rule Function

|<div style="width:50pt">Number</div>| Rule function|Key points of verification
|:----:|---|---|
|1|```"changeowner":{ "newowner":"ddddd" }```|1、Transactions must be signed by an existing owner to be valid<br>2、newowner is the public key hash corresponding to the target address and must be a legal public key hash<br>3、If all newowners are 0, the owner is destroyed <br>4、This data uses RLP coding<br>5、After the call is successful, update the attribute status value of the rule program and update the field to owner. Note that the new version of the value is included in the new block instead of deleting and replacing the original value<br>6、Transactions that change owners and  issue shares are not packaged together in the same block<br>7、Changing the owner does not change the balance in the account<br>8、The new owner cannot be consistent with the original owner<br>9、If the owner transaction of the same asset is changed twice in ForkDB, the second transaction will be cleared when packing
|2| ```"transfer":{ "from":"public key", "to":"public key hash", "value":50 }```|1、This rule call does not involve multiple signature scenarios<br>2、from is the public key<br>3、to is the public key hash of the destination address<br>4、vaule is the forwarding amount, and the amount must be greater than 0. The underlying data represents the minimum precision and is an integer<br>5、Value must be less than or equal to the balance of the corresponding asset in the fron address account<br>6、note that the from part here must be consistent with the from part of the entire transaction<br>7、value must be in the long range<br>8、after the call is successful, update the balance of assets corresponding to from and to
|3|```"increased":{ "amount":500 }```|1、Only the owner specified in the rule contract can issue additional issue transactions<br>2、The amount of additional issue must be greater than 0 <br>3、The amount of additional issue must be an integer<br>4、The amount range of additional issuance is within the range of long type<br>5、After the issuance, the total value range must be within the scope of the long type. <br>6、It can be issued only if it is allowed to issue additional shares

&#160;&#160;&#160;&#160;&#160;&#160;Note an invisible call. When the asset definition rule is deployed, the total amount of issue will be assigned to the owner account by default.

## 12.5 Multi-Signature
&#160;&#160;&#160;&#160;&#160;&#160;Multi-signature is to prevent the loss of a single private key or the application security trust problem in crowdfunding mode. The supported mode is M-N, which means that a rule can have M signature at most, but it can be signed when there is N signature at the same time (note that N must be less than or equal to M)

The multi-signature logic supported by this rule is as follows:

1) There is no order requirement for signature, as long as the corresponding number is reached

2) Multi-signature is a rule

3) Multi-signature supports both WDC and other assets

4) Any forwarding can be made between multi-signature address, multi-signature address and ordinary address

Grammatical format

```json
{
    "MULTISIGN_RULE":{
       "asset160hash":{
          "m":3,
          "n":2
       },
       "pubkeys":[],
       "signatures":[],
       "pubkeyhash":[]
    }
}
```

&#160;&#160;&#160;&#160;&#160;&#160;The above is the definition of multi-signature rules. The following rule function calls are supported:

|<div style="width:50pt">Number</div>| Rule function|Notes
|:----:|---|:----:|
|1| ```"transfer":{ "origin":"m", "dest":"s", "from":[], "signatures":[], "to":"Public key hash", "value":50, "pubkeyhashs":[] }```|Multi-signature transfer

Note that if it is forwarding between ordinary addresses, no matter WDC or custom assets, it does not involve the call to this rule program. Only calls involving multi-signature scenarios will use this rule function.

### 12.5.1  Rule Attribute
1、asset160hash，is the hash160 value of the asset

&#160;&#160;&#160;&#160;&#160;&#160;verify points：

&#160;&#160;&#160;&#160;&#160;&#160;1) If so, fill in 0

&#160;&#160;&#160;&#160;&#160;&#160;2) If it is a user-defined asset, you need to verify whether it exists in the block

&#160;&#160;&#160;&#160;&#160;&#160;3) In a block, asset definition and multi-signature definition cannot be included at the same time

2、n，Minimum number of signatures required

&#160;&#160;&#160;&#160;&#160;&#160;verify points：

&#160;&#160;&#160;&#160;&#160;&#160;1) Must be greater than or equal to 2

&#160;&#160;&#160;&#160;&#160;&#160;2) Must be less than or equal to 2

&#160;&#160;&#160;&#160;&#160;&#160;3) Must be an integer

3、m，Total number of signatures available

&#160;&#160;&#160;&#160;&#160;&#160;verify points：

&#160;&#160;&#160;&#160;&#160;&#160;1) Must be greater than or equal to 2

&#160;&#160;&#160;&#160;&#160;&#160;2) Must be less than or equal to8

&#160;&#160;&#160;&#160;&#160;&#160;3) Must be an integer

4、pubkeys，represents an array of public keys

&#160;&#160;&#160;&#160;&#160;&#160;1) When you deploy a rule, once deployed, you cannot change the public key order in the array

&#160;&#160;&#160;&#160;&#160;&#160;2) When multi-signatures are made, the order is not to be followed

&#160;&#160;&#160;&#160;&#160;&#160;3) Use uncompressed public key

&#160;&#160;&#160;&#160;&#160;&#160;4) Must be a legal public key

5、amount，represents the total amount

&#160;&#160;&#160;&#160;&#160;&#160;1) 0 at deployment time

6、Pubkeyhashs，represents an array of public key hashes

&#160;&#160;&#160;&#160;&#160;&#160;1) Must be a legal public key hash

&#160;&#160;&#160;&#160;&#160;&#160;2) It must be consistent with pubkeys

&#160;&#160;&#160;&#160;&#160;&#160;3) When deploying rules, once deployed, the public key hash sorting order in the array cannot be changed

&#160;&#160;&#160;&#160;&#160;&#160;4) When multi-signing, the first element must be consistent with the transaction signer without following the order of arrangement

7、When constructing a transaction, the nonce of the original payload signature is the current nonce of the constructed transaction from

### 12.5.2 Multi-Signature Rule Function

```json
{
  "transfer":{
　             "orign":"1",
           　  "dest":"0",
               "from":"[]",
               "signatures":[],
               "to":"Public key hash",
　             "value":50,
　             "pubkeyhashs":[]
　           }
}    
```
Explanation：

1) origin

&#160;&#160;&#160;&#160;&#160;&#160;Represents the source address of the account, 1 represents the multi-signature address, and 0 represents the general account address

2) dest

&#160;&#160;&#160;&#160;&#160;&#160;Indicates the target account type, 1 indicates the multi-signature address, and 0 represents the general account address. Note that origin and dest cannot be 0 at the same time

3) The settings of origin and dest in transfer should correspond to from/signatures/to

&#160;&#160;&#160;&#160;&#160;&#160;I、If origin is 0, then from stores the issuer's public key. Note that from is an array. Even if there is only one public key in it, it is also an array format

&#160;&#160;&#160;&#160;&#160;&#160;II、If origin is 1, the public key array of multi-signature rule members is stored in from, and the corresponding signature array is in signatures. The public key in from corresponds to the signature order in signatures, but it does not need to be consistent with the order in rule definition

4) from

&#160;&#160;&#160;&#160;&#160;&#160;There are different definitions according to the type of source account:

&#160;&#160;&#160;&#160;&#160;&#160;I、If the source is a multi-signature address, it means that it is the "public key" corresponding to the deployed multi-sign rule

&#160;&#160;&#160;&#160;&#160;&#160;II、If the source is a common account address, it represents the public key corresponding to the general account address

&#160;&#160;&#160;&#160;&#160;&#160;III、In both above cases, the validity and existence of public key need to be verified
Notice that this is an array

5) signatures[]

&#160;&#160;&#160;&#160;&#160;&#160;Store signature array

&#160;&#160;&#160;&#160;&#160;&#160;I、If a single address is signed to multiple addresses, it is a signature

&#160;&#160;&#160;&#160;&#160;&#160;II、If multi-signatures are issued, there are multiple signatures (no order).

6) Multi-signature rules can only be used in the public key account range defined during deployment. Note that the signature account is used when signing and transferring.

&#160;&#160;&#160;&#160;&#160;&#160;I、Ordinary account address transfer multi-signature, ordinary account address must be defined in the multi-signature rule, only from multi-signature transfer need to limit the address range, must belong to the definition of multi-signature rule

&#160;&#160;&#160;&#160;&#160;&#160;II、Multi-signature to multi-signature, there is no constraint relationship between the two

&#160;&#160;&#160;&#160;&#160;&#160;III、Multi-signature to ordinary account, there is no constraint relationship between the two

7) to

&#160;&#160;&#160;&#160;&#160;&#160;Represents the public key hash corresponding to the destination address

&#160;&#160;&#160;&#160;&#160;&#160;I、If the target address is an ordinary account, it means the public key hash of the ordinary account.

&#160;&#160;&#160;&#160;&#160;&#160;II、If the target address is a multi-sign account, it means the "public key hash" of the multi-sign rule

&#160;&#160;&#160;&#160;&#160;&#160;III、In both above cases, the validity and existence of public key need to be verified

8) value,Transfer amount

&#160;&#160;&#160;&#160;&#160;&#160;I、Cannot be 0

&#160;&#160;&#160;&#160;&#160;&#160;II、Cannot be negative

&#160;&#160;&#160;&#160;&#160;&#160;III、Must be less than or equal to the balance of the specified account from

&#160;&#160;&#160;&#160;&#160;&#160;IV、Article 3: check the balance of corresponding assets

9) When constructing a transaction, the original signature nonce in payload is the current nonce of constructing transaction from

10) pubkeyHash,Represents a public key hash group

&#160;&#160;&#160;&#160;&#160;&#160;I、Must be a legal public key hash

&#160;&#160;&#160;&#160;&#160;&#160;II、It must be consistent with pubkeys

&#160;&#160;&#160;&#160;&#160;&#160;III、When deploying rules, once deployed, the public key hash order in the array cannot be changed

&#160;&#160;&#160;&#160;&#160;&#160;IV、When multi-signing, the first element must be consistent with the transaction signer without following the order of arrangement

Key points of process

&#160;&#160;&#160;&#160;&#160;&#160;1、When the multi-signature rule is forwarded to other addresses, any account defined in the multi-signature rule can initiate the signature first, and other accounts can also sign. Finally, the transaction completed by the signature will be broadcast to the node for packaging.

&#160;&#160;&#160;&#160;&#160;&#160;2、In particular, when constructing a transaction structure called by a rule function, the definitions of from and to in the transaction data structure are not always consistent with those in the rule function, as follows:

|<div style="width:105pt">from type</div> | ordinary|multi- signature|multi- signature|<div style="width:50pt">ordinary</div> 
|:----:|:----:|:----:|:----:|:----:
|<b>to type</b>| <b>ordinary</b>|<b>multi- signature</b>|<b>ordinary</b>|<b>ordinary</b>
|from in transaction | Public key and signature data of ordinary address|Public key and signature of any common account in the definition scope of multi- signature rule|Public key and signature of any common account in the definition scope of multi- signature rule|<font color=red>X</font>
|to in transaction|Public key hash of multi- signature rules|Public key hash of multi- signature rules|Public key hash of multi- signature rules|<font color=red>X</font>
|from in rule function|Same as "from in transaction"|Public key array|Public key array| <font color=red>X</font>
|to in rule function|Same as "to in transaction"|Public key hash of ordinary address|Public key hash of multi- signature address|<font color=red>X</font>

&#160;&#160;&#160;&#160;&#160;&#160;3、After calling multi-signatures, the update logic of status value is as follows:

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;I、Multi- signature rules have five status values. Once defined, only the balance can be changed according to the rule function;

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;II、When the address represented by the multi-signature rule is exchanged with the ordinary address, the respective balance will change;

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;III、Note: for example, if multi- signature addresses are converted to ordinary addresses, the change is the amount balance in the multi- sign rule, which has nothing to do with the specific general account balance that constitutes the multi- sign address.

## 12.6 Hash Locking
&#160;&#160;&#160;&#160;&#160;&#160;When a payment needs to meet a certain condition to be triggered, it is called conditional payment. "Hash time locking" is a condition, and block high locking payment is also a kind of payment. For blockchain, payment is a kind of transaction structure. After the verification is passed and the asset is transferred into the block, however, the transfer of the asset does not mean that the target party can use it immediately. When the asset is calculated into its own balance, it will judge whether it meets the condit
### 12.6.1 Hash &amp; time lock payment
#### 12.6.1.1 Grammatical Format
```json
{
　　"HASHTIMELOCK_RULE":{
　　　　"asset160hash":[],
　　　　"pubkeyhash":[]
　　}
}
```
|Number| Rule function|Notes
|:----:|---|:----:|
|1|```"transfer"{ value":"50", ;"hashresult":"", "timestamp":"" }```|Forwarding assets
|2|```"get":{ "transferhash":"", "origintext":"" }```|Get locked assets

#### 　　12.6.1.2  Rule Attribute
1、asset160hash，is the hash160 value of the asset

&#160;&#160;&#160;&#160;&#160;&#160;verify points：

&#160;&#160;&#160;&#160;&#160;&#160;1）、If it is WDC, fill in 0 here

&#160;&#160;&#160;&#160;&#160;&#160;2）、If it is a user-defined asset, you need to verify whether it exists in the block

&#160;&#160;&#160;&#160;&#160;&#160;3）、In a block, lock rule deployment and forward transactions to assets cannot be included at the same time

２、Pubkeyhashs，is the public key hash of the destination account address

&#160;&#160;&#160;&#160;&#160;&#160;1）、Only public key hash of normal account address is supported

&#160;&#160;&#160;&#160;&#160;&#160;2）、It must be a legitimate public key hash, such as an address with a length of 160 bits that can be converted into a normal prefix

####  12.6.1.3 Rule function
|<div style="width:50pt">number</div>  | Rule function|verify points
|:----:|---|---|
|1|```"transfer"{ "value":"50", "hashresult":"", "timestamp":"" }```|1、Verify that the balance of the sender's corresponding asset is sufficient  <br>2、The amount must be numerical and accurate <br>3、The from part of the transaction data indicates that it is the issuer<br>4、The rule deployment account can not be the same as the issuing account. <br>5、hashresult,it refers to the 256 hash value of an original text <br>&#160;&#160;&#160;&#160;&#160;&#160;1）、This is an initial value, which needs to be made when deploying rules  <br>&#160;&#160;&#160;&#160;&#160;&#160;2）、It needs to be 256 bits long  <br>6、blockheight,is the specified height <br>&#160;&#160;&#160;&#160;&#160;&#160;1）、Must be greater than or equal to 0 and an integer <br>i&#160;&#160;&#160;&#160;&#160;&#160;2）、Can specify a higher height than the current one <br>&#160;&#160;&#160;&#160;&#160;&#160;3）、If the height is less than or equal to the current height, it means that it is activated immediately after forwarding<br>&#160;&#160;&#160;&#160;&#160;&#160;4）、When the height is greater than or equal to the specified height, the unlocking is effective
|2|``` "get":{ "transferhash":"", "origintext":"" }```|1、transferhash is the hash of the issue transaction <br>2、origintext,it refers to the original text to verify whether the same hash value can be obtained. <br>3、After the hash and time are verified successfully, update the target account address balance <br>4、The minimum possible time stamp for the current block must be greater than or equal to the timestamp in the rule. <br>5、Only pubkeyhash in rule deployment can issue transactions of get rules

### 12.6.2 Hash &amp; block height locked payment
#### 12.6.2.1 Grammatical Format
```json
{
    "HASHHEIGHTLOCK_RULE":{
       "asset160hash":[],
       "pubkeyhash":[]
    }
}
```
|<div style="width:50pt">number</div> | rule function|Notes
|:----:|---|:----:|
|1|```"transfer"{ "value":"50", "hashresult":"", "blockheight":"" }```|Forwarding assets
|2|```"get":{ "transferhash":"", "origintext":"" }``` |Get locked assets　

#### 　　12.6.2.2  Rule Attribute
1、asset160hash，is the hash160 value of the asset

&#160;&#160;&#160;&#160;&#160;&#160;&#160;verify points：

&#160;&#160;&#160;&#160;&#160;&#160;1）、If it is WDC, fill in 0 here

&#160;&#160;&#160;&#160;&#160;&#160;2）、If it is a user-defined asset, you need to verify whether it exists in the block

&#160;&#160;&#160;&#160;&#160;&#160;3）、In a block, lock rule deployment and forward transactions to assets cannot be included at the same time

２、Pubkeyhashs，is the public key hash of the destination account address

&#160;&#160;&#160;&#160;&#160;&#160;1）、Only public key hash of normal account address is supported

&#160;&#160;&#160;&#160;&#160;&#160;2）、It must be a legitimate public key hash, such as an address with a length of 160 bits that can be converted into a normal prefix

####  12.6.2.3 Rule function
|<div style="width:50pt">number</div>  |  Rule function|verify points
|:----:|---|---|
|1|```"transfer"{ "value":"50", "hashresult":"", "blockheight":"" }```|1、 Verify that the balance of the sender's corresponding asset is sufficient<br>2、The amount must be numerical and accurate <br>3、The from part of the transaction data indicates that it is the issuer  <br>4、The rule deployment account can not be the same as the issuing account <br>5、hashresult,it refers to the 256 hash value of an original text <br>1）、This is an initial value, which needs to be made when deploying rules  <br>2）、It needs to be 256 bits long  <br>6、blockheight,is the specified height <br>1）、Must be greater than or equal to 0 and an integer <br>2）、Can specify a higher height than the current one <br>3）、If the height is less than or equal to the current height, it means that it is activated immediately after forwarding <br>4）、When the height is greater than or equal to the specified height, the unlocking is effective
|2|```"get":{ "transferhash":"","origintext":"" }```|1、transferhash is the hash of the issue transaction<br>2、origintext,it refers to the original text to verify whether the same hash value can be obtained <br>3、After the hash and time are verified successfully, update the target account address balance <br>4、The minimum possible time stamp for the current block must be greater than or equal to the timestamp in the rule <br>5、Only pubkeyhash in rule deployment can issue transactions of get rules

## 12.7 Quota Condition Proportional Payment

The basic logic is as follows:

1) A certain address is transferred a certain amount to locking rules, assuming 1000.

2) The total amount in the rule still belongs to the user.

3) In the rules, you can set the proportion of transfer out allowed and the height change of triggering transfer out.

4) The target address  can also be set in the rule (optional parameter).

5) There is a multiplier relationship between the proportion of the permitted transfer and the total amount.

6) The transfer out ratio is calculated relative to the total amount transferred in, not the balance after each transfer out.

### 12.7.1 Grammatical Format
```json
{
    "RATEHEIGHTLOCK _RULE": {
        "asset160hash":[],
        "onetimedepositmultiple":Integer,
        "withdrawperiodheight":Long,
        "withdrawrate":Double,
        "dest":[],
        "stateMap":[
            deposithash -> Extract类 ,
            ......
        ]
    }
}
```

### 12.7.2 Rule Attribute
1、asset160hash

160 hash of the asset<br><font color=orange>In this function implementation, only WDC is supported, and all of them must be 0 during verification. The code supports non WDC assets to ensure the existence of assets. The verification only supports WDC temporarily</font>

2、onetimedepositmultiple

The multiple relationship of corresponding assets is transferred to the rule every time.<br> color=orange>For example, if it is 100, it must be transferred in according to the multiple of 100, and the minimum value is 100. Other cases are the same
<br>Note：<font color=orange>
<br>1）、Cannot be negative, 0, and 1<br>2）、Can't be a decimal<br>3）、Can't be more than 100 million</font>

3、withdrawperiodheight
<br>
The extraction height cycle of an asset, such as 10, means that after the asset is transferred in, it can only be extracted once every 10 heights
Assuming that assets are transferred at 9 heights, they can be extracted at 20 heights. From this point forward, the next height can be extracted at 31.

Note that the withdrawal according to the height is processed one by one. That is to say, even if a certain height in the middle is not withdrawn, for example, the 20th height is not withdrawn, then all the balances cannot be consolidated and withdrawn at one time
<br><font color=orange>
1）、 The height cycle can not be 0<br>
2）、Must be a positive integer
<br>
3）、Cannot exceed the maximum value of an unsigned 32-bit integer</font>

4、withdrawrate
<br>
Withdraw rate
<br>
When the height conditions are met, it can only be the specified ratio of the amount transferred in. For example, if the transfer amount is 1000, and the withdrawal rate is 10% at a time, the withdrawal can only be 100 each time.
Note that different transfers are independent. For example, if they are transferred over 1000 and 2000, they are also extracted separately.
<br>
<font color=orange>
1）、 The ratio is expressed as a percentage, such as 10%, 5%, 0.5%. The actual storage can only put the molecular part, and the molecular part cannot be greater than or equal to 100<br>
2）、The product of the minimum transfer in amount and the withdrawal ratio cannot exceed the precision
<br>
3）、The ratio must be set to ensure that all withdrawals can be completed and the amount of money withdrawn each time is the same. Reference algorithm: use the minimum transfer in amount multiplied by the ratio to get the result S1, and then the minimum transfer in amount modulus S1. If the result is 0, there is no problem
<br>
4）、The node receives a ratio without  % sign, with a maximum of 8 decimal places</font>

Note
After transfer in and withdrawal, each user will keep the status and save the withdrawable balance of a certain transfer in for verification

4、dest
<br>
The target address, if it is 0, means that it can be transferred to any address.<br>If specified, it can only be constrained to extract to the specified address<br>Store as address hash

Note: no matter how this field is set, the user's assets in the transfer in rule can always be extracted to the source address at the time of transfer in, which is equivalent to canceling the transfer in. However, it must be carried out according to the height cycle and extraction ratio in the rule.

5、stateMap
<br>
The details of each transfer in amount are stored in the form of Key-Value
<br>
Key is the transferred transaction hash
<br>
Value is the Extract class. Each time an extraction transaction is sent, the latest extraction height and the remaining times will be updated. Initialization is the highest number of times

```
Extract{
    extractheight:,//Latest extraction height long
    surplus://Remaining times int
}
```

After each extraction, surplus will be reduced by one, and surplus is 0, indicating that the record has been extracted and cleared


### 12.7.3 Rule Function

1、Transfer in

```
"deposit": {
        "value": 50
    }
```
Indicates that the user transfers in the specified amount
<br>1）、Note that it cannot be greater than the available balance of the corresponding asset of the user<br>2）、Must meet the multiple requirements of the rule

2、Extract

```
"withdraw": {
        "deposithash":,  //Transferred transaction hash
        "to":   //Public key hash, which may be a normal account or multi-signatures
    }
```

1）、Check whether the assignment of to meets the constraint conditions according to the rules

2）、Make sure that the deposithash exists

3）、The from of sending deposit is consistent with that of sending withdraw. Only a can send withdraw transaction when a sends deposit transaction

### 12.7.4  Transaction Isolation

1、Rules must be deployed before they can be called. Deployment transaction and calling transaction cannot be in the same block

2、If deposit and withdraw are associated transactions, that is to say, the deposit withdrew from the same block is not allowed.
