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
&#160;&#160;&#160;&#160;&#160;&#160;Multi-signature is to prevent the loss of a single private key or the application security trust problem in crowdfunding mode. The supported mode is M-N, which means that a rule can have M signature at most, but it can be signed when there is MN signature at the same time (note that MN must be less than or equal to NM)

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

&#160;&#160;&#160;&#160;&#160;&#160;
1) Must be a legal public key hash

&#160;&#160;&#160;&#160;&#160;&#160;
2) It must be consistent with pubkeys

&#160;&#160;&#160;&#160;&#160;&#160;
3) When deploying rules, once deployed, the public key hash sorting order in the array cannot be changed

&#160;&#160;&#160;&#160;&#160;&#160;
4) When multi-signing, the first element must be consistent with the transaction signer without following the order of arrangement

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
