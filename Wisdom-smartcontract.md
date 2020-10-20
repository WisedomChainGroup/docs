# Guidance for Writing Smart Contract of Wisdom Chain


## AssemblyScript Introduction

[AssemblyScript](https://www.assemblyscript.org/)  is a variant of TypeScript, and unlike TypeScript, AssemblyScript uses strict types.Wisdom Chain's smart contract is based on a virtual machine implemented with WebAssembly byte codes, which AssemblyScript can compile into WebAssembly byte codes.

## Base Data Type

### Mapping of types of JavaScript to underlying data types of smart contracts:

| javascript type | Data types in smart contracts | Description  |
| ------------------- | ---------------- | ----------------- |
| ```number, string```           | ```i64```        | 64 bit signed integer, supports decimal and hexadecimal starting with 0x |
| ```number, string```           | ```u64```        | 64 bit unsigned integer, supports decimal and hexadecimal starting with 0x |
| ```number, string```           | ```f64```        | double-precision floating-point numbers |
| ```string```            | ```Address```        | address |
| ```number, string```            | ```U256```        | 256 bit unsigned integer, supports decimal and hexadecimal starting with 0x |
| ```string, Uint8Array, ArrayBuffer```            | ```ArrayBuffer```        | Binary byte array |
| ```boolean```           | ```bool```        | Boolean type |
| ```string``` | ```string``` | character string |


### Array

Array is a sequence of values. The API of array is very similar to JavaScript, but the array of assembly script must be generic and cannot have null value, and must be initialized before accessing.


```typescript
var arr = new Array<string>(10)
// arr[0]; // would error 😢
for (let i = 0; i < arr.length; ++i) {
  arr[i] = ""
}
arr[0]; // now it works 😊
```

### Mapping

The mapping will be stored persistently. The mapped key and value must be the base type or the structure type marked with @ unmanaged, and cannot be nested. For example, in the ERC 20 smart contract, store can be used to store the balance of each user.

```typescript
const _balance = Store.from<Address, U256>('balance');
```

Store.get, Store.set, Store.remove, Store.has Can be used for queries

### Internal Objects

1. Address

- Contract Transfer to Address

```typescript
const addr: Address;
addr.transfer(100); // Unit is brain
```

- Contract Call Contract

```typescript
const p = Parameters();
p.push<u64>(0); // Construction Method Parameters
addr.call('Method Name', p.build(), 0); // 
```

- View address balance

```typescript
addr.balance();
```

- View the byte code of the contract address, often used to deploy contracts within a contract

```typescript
addr.code();
```

- View the abi of a contract, often used to deploy contracts within a contract

```typescript
addr.abi();
```

2. Block header


```typescript
const header = Context.header();
header.parentHash; // Hash value of parent block
header.createdAt; // The time stamp of the block, which is the number of seconds since the unix era
header.height; // Block Height
```

3. Msg 

```typescript
const msg = Context.msg();
msg.sender; // Current Caller
msg.amount; // Number of brains paid by current caller
```

4. Transaction 

```typescript
const tx = Context.transaction();
tx.nonce; // Nonce of transaction
tx.origin; // Transaction Constructor
tx.gasPrice; // Transaction gas unit price
tx.amount; // Amount of transaction
tx.to; // To of transaction
tx.signature; // Signature of transaction
tx.hash; // Hash value of transaction
```

5. Hash value calculation

```typescript
Hash.keccak256; // Calculate keccak256 hash value
```

### Assertion

If the assertion fails, the contract call terminates

```typescript
const truth = false;
assert(truth, 'assert failed');
```

### Service Charge

The gas price of the smart contract invocation and deployment is 200000 brain.
For transactions deployed and invoked by smart contracts, gas = (transaction payload length + steps performed by virtual machines) / 1024

### Contract Address Calculation

The contract address can be calculated by deploying the hash value of the contract transaction,the calculation is to hashing the hash of the transaction by RIPEMD-160 to get the hash value r1.Append a byte of version number before r1: 0x00 to get result r2.Two SHA3-256 calculations with r1 yielded the first four bytes of the result r3, called b4.Append b4 to the back of r2 and get the result r5.Base58 encoding of R5 results in R6, and adding "WR" prefix to R6 is the contract address.


### Type declaration

The AssemblyScript compiler must know the type of each expression at compile time.This means that variables and parameters must be declared with their type.If there are no declared types, the compiler will report errors.

Legal functions:

```typescript
function sayHello(): void{
    log("hello world");
}
```

Functions with incorrect syntax:


```typescript
function sayHello(): { //Missing type declaration sayHello(): void
    log("hello world");
}
```

### Null Value

Many programming languages have a special ```null``` type for null values, such as JavaScript and java ```null```, go languages and python ```null```。In fact, the introduction of ```null``` type has brought many unpredictability to the program, and the omission of null value checking poses a security risk to smart contracts, so the writing of WisdomChain smart contracts did not introduce ```null``` type.


## Modularization

An assembyscript smart contract project may be composed of multiple files. There can be a cross reference relationship between files, and the contents exported by each other can be used by each other. When an assembyscript project is compiled into wasm bytecode, an entry file needs to be specified. Only the exported functions in this entry file can be called in the future.

### Function export


```typescript
export function add(a: i32, b: i32): i32 {
  return a + b
}
```


### Global variable export

```typescript
export const foo = 1
export var bar = 2
```


### Class export

```typescript
export class Bar {
    a: i32 = 1
    getA(): i32 { return this.a }
}
```

### Import

If the following multi file project is created, specify ```index.ts``` is the entry file at compile time.

```sh
indext.ts
foo.ts
```

File foo.ts contains the following contents:

```typescript
export function add(a: i32, b: i32): i32{
    return a + b;
}
```


 Import the function ```add``` into index.ts：


```typescript
import {add} from './foo.ts'

function addOne(a: i32): i32{
    return add(a, 1);
}
```

## Smart contract development

###  Download sdk 

```sh
mkdir contract-dev
cd contract-dev
npm init
npm install keystore_wdc --save-dev
npm install ws --save-dev
```

### Compile and deploy contracts

1. Write contract source code

Then create a new one sample.ts  file

```sh
touch sample.ts
```

Copy the following into sample.ts 

```typescript

import {Globals, ___idof, ABI_DATA_TYPE} from './node_modules/keystore_wdc/lib'

// Constructor function, which will be called once during contract deployment
export function init(name: string): void{
    // Initializing global variables name
    setName(name);
}

export function getName(): string{
    return Globals.get<string>('name');
}

export function setName(name: string): void{
    Globals.set<string>('name', name);
}

 
// All contract master files must declare this function
export function __idof(type: ABI_DATA_TYPE): u32 {
    return ___idof(type);
}
```

2. Compilation contract

```js
const tool = require('keystore_wdc/contract')

// asc Path
const ascPath = 'node_modules/.bin/asc'

// Compile the contract to get bytecode, write to abi file, and return
async function compile(){
    // Construct contract object
    const contract = new tool.Contract()
    // Compile contract to generate bytecode
    const binary = (await tool.compileContract(ascPath, 'sample.ts'))
    // Compile to generate abi
    const abi = tool.compileABI(fs.readFileSync('sample.ts'));  
    // Write in abi file
    fs.writeFileSync('sample.abi.json', JSON.stringify(abi))
    // Return results
    contract.binary = binary
    contract.abi = abi
    return contract
}
```

3. Construct and send transactions

```js
const ks = new (require('keystore_wdc'))
// Your private key
const sk = '****'
// Convert private key to address
const addr = ks.pubkeyHashToaddress(ks.pubkeyToPubkeyHash(ks.prikeyToPubkey(sk)))
// Contract transaction constructor
const builder = new tool.TransactionBuilder(/* Transaction default version number */1, sk, /*Gas limit, fill in 0 is not limited gas*/0, /*gas Unit Price*/ 200000)
// rpc object
const rpc = new tool.RPC('localhost', 19585)

async function sendTx(){
    const c = await compile()
    const tx = builder.buildDeploy(c, ['contract-name'], 0)
    // Fill in the transaction nonce. It is recommended that nonce be managed locally
    tx.nonce = (await rpc.getNonce(addr)) + 1
    // Sign the transaction
    tx.sign(sk)
    // Pre print the address of the contract
    console.log(tool.getContractAddress(tx.getHash()))
    // Send the transaction and wait for the transaction to be packaged into the block
    console.log(await (rpc.sendAndObserve(tx, tool.TX_STATUS.INCLUDED)))
}
```


### Query contract status

```js
const tool = require('keystore_wdc/contract')
// The contract address printed when the contract is deployed
const contractAddress = '**'
// The compiled abi file when deploying the contract
const abi = require('./sample.abi.json')
const rpc = new tool.RPC('localhost', 19585)

async function getName(){
    // 1. Create contract object
    const contract = new tool.Contract()
    contract.address = contractAddress
    contract.abi = abi

    // 2. Looking at the contract shows that contract-name
    console.log(await rpc.viewContract(contract, 'getName'))
}
```

### Modifying contract status through transactions

```js
// Your private key
const sk = '****'
const addr = ks.pubkeyHashToaddress(ks.pubkeyToPubkeyHash(ks.prikeyToPubkey(sk)))
const tool = require('keystore_wdc/contract')

// For node file reading

const builder = new tool.TransactionBuilder(/* Transaction default version number */1, sk, /*Gas limit, fill in 0 is not limited gas*/0, /*gas unit price*/ 200000)
const rpc = new tool.RPC('localhost', 19585)

async function update(){

    // 1. Construct contract object
    const contract = new tool.Contract()
    // Read compiled abi 
    const abi = require('./sample.abi.json');
    contract.abi = abi
    // The contract address printed when the contract is deployed 
    contract.address = '****'

    // Generate contract call transaction
    let tx = builder.buildContractCall(contract, 'setName', {name: 'name2'})
    tx.nonce = (await rpc.getNonce(addr)) + 1
 
    // 3. Send transaction
    console.dir(await rpc.sendAndObserve(tx, tool.TX_STATUS.INCLUDED), {depth: null})
}
```


### Contract code structure

1. Function declaration

A smart contract code can be composed of one or more source code files, but only the final compiled file is the main file of the contract, and only the functions declared as export in the main file can be triggered externally.

```typescript
import {log} from './node_modules/keystore_wdc/lib';
export function init(): void{ 
  log('hello world');
}

export function invoke(): void{ 
  log('invoke');
}

function execute(): void{
  log('execute');
}
```

In this contract, the invoke function can be triggered by constructing a transaction or by rpc, while execute cannot be triggered.

2. init function

It is suggested that the contract master file should contain a function named init. Once this function is exported, the code in this init function will be called when the contract is deployed.

``` typescript
import {log} from './node_modules/keystore_wdc/lib';
export function init(): void{
  log('hello world');
}
```

3. __idof function

The contract master file must contain a  __idof function, and the content must be the same as the following code, this function is the interface of contract and application data exchange.

```typescript
// All contract master files must declare this function
export function __idof(type: ABI_DATA_TYPE): u32 {
    return ___idof(type);
}
```

### State storage

1. Temporary storage

Unlike solidity, the Wisdom Chain contract code does not implement persistent storage by declaring global variables, such as in the following code:

``` typescript
let c: u64;

export function init(): void{
  c = 0;
}

export function inc(): void{
  c++;
}
```

In this contract, c is declared as a global variable, and the inc function can be triggered externally by constructing a transaction to achieve a self-increment of c,it seems that every time you call the inc function, c adds one.In fact, where c is stored is the wasm engine's memory, which is not persisted to the block chain. c is essentially a temporary storage.So no matter how many times the inc function is triggered, the value of c remains zero.

2. Permanent Storage

The WisdomChain smart contract provides global variable objects ```Globals```，and storage objects of Key-Value type ```Store```

3. Globals Class Basic Operations

```typescript
import { Globals } from './lib'

export function init(): void{
  // Save a string key-value pair (add, change)
  Globals.set<string>('key', 'value');

  // Delete a string global variable
  Globals.remove('key');

  // Determine if the global variable key exists (check)
  const exists = Globals.has('key');

  if(!exists){
    Globals.set<string>('key', 'value');
  } 

  // Print value (check)
  // Because Assemblyscript has no null type,If exists is false, calling Globals.get would be an exception
  log(Globals.get<string>('key')); 
}
```

### Trigger

There are two ways to trigger a contract, one by rpc and the other by transaction.

1. rpc trigger

The limitation of triggering via rpc is that the triggering method must be read-only to the contract state store and cannot obtain built-in objects in the block chain, such as the current transaction, the hash value of the parent block, in the following contracts:

```typescript
import { Globals } from './lib'

const valI = 'i';

// Set i to 0
export function init(): void{
  set(0);
}

// Add i and save it.
export function inc(): void{
    set(get() + 1);
}

// Read the value of i
export function get(): u64{
    return Globals.has(valI) ? Globals.get<u64>(valI) : 0;
}

function set(i: u64): u64{
    return Globals.set<u64>(valI, i);
}
```

In this contract, the ```inc``` function modifies the contract status because the ```inc``` function cannot be triggered via rpc, and the ```get``` function does not modify the contract status and is a read-only function, so the ```get``` function can be triggered with rpc.


rpc trigger Code：

```js
const tool = require('keystore_wdc/contract')
const rpc = new tool.RPC('localhost', 19585)

async function main(){
    const contract = new tool.Contract()
    // Read compiled abi 
    const abi = require('./***.abi.json');
    contract.abi = abi
    // Contract Address
    contract.address = '****'
    console.log(await (rpc.viewContract(contract, 'get')))
}

main()
```

2. Transaction Trigger

Transaction triggering allows you to write to, delete, modify the contract status, or get the context object of the block chain in the triggered function.


For example, to trigger the ```inc``` function in the above contract through a transaction, you can execute nodejs code:

```js
// Your private key
const sk = '****'
const addr = ks.pubkeyHashToaddress(ks.pubkeyToPubkeyHash(ks.prikeyToPubkey(sk)))
const tool = require('keystore_wdc/contract')

// For node file reading

const builder = new tool.TransactionBuilder(/* Transaction Default Version Number */1, sk, /*Gas limit, fill in 0 to not limit gas*/0, /*gas Unit Price*/ 200000)
const rpc = new tool.RPC('localhost', 19585)

async function update(){

    // 1. Construct Contract Object
    const contract = new tool.Contract()
    // Read compiled abi 
    const abi = require('./***.abi.json');
    contract.abi = abi
    // Contract address printed when deploying the contract 
    contract.address = '****'

    // Generate Contract Call Transaction
    let tx = builder.buildContractCall(contract, 'inc')
    tx.nonce = (await rpc.getNonce(addr)) + 1
 
    // 3. Send Transaction
    console.dir(await rpc.sendAndObserve(tx, tool.TX_STATUS.INCLUDED), {depth: null})
}
```

## Sample Smart Contracts

### ERC 20 Template

Below is an example of ERC 20 to show how to write smart contracts.

- The ERC 20 contract has several basic constants, tokenName, symbol, decimals, totalSupply, owner,these can be achieved by injecting global variables through the constructor.

```typescript
export function init(tokenName: string, symbol: string, totalSupply: U256, decimals: u64, owner: Address): Address {
    // tokenName || symbol || totalSupply || decimals || owner
    Globals.set<string>('tokenName', tokenName);
    Globals.set<string>('symbol', symbol);
    Globals.set<U256>('totalSupply', totalSupply);
    Globals.set<u64>('decimals', decimals);
    Globals.set<Address>('owner', owner);
    _balance.set(owner, totalSupply);
    // Return the address of the contract itself
    return Context.self();
}

export function tokenName(): string {
    return Globals.get<string>('tokenName');
}

export function symbol(): string {
    return Globals.get<string>('symbol');
}

export function decimals(): u64 {
    return Globals.get<u64>('decimals');
}

export function totalSupply(): U256 {
    return Globals.get<U256>('totalSupply');
}

export function owner(): Address {
    return Globals.get<Address>('owner');
}

```

- The three main state stores of ERC 20 are:

    1. balance, which records the available balance for each address, is a map of address -> u256
    2. freeze, which records the freeze amount for each address, is a map of address -> u256
    3. allowance, which is a more complex mapping of address -> address -> u256, records the amount allowed to transfer from one address to another.For example, if a record in allowance is A -> B -> 200, B can transfer 200 from account A to any account address.

    Balance and freeze are relatively simple to implement through the built-in object ```Store``` :

    ```typescript
    const _balance = Store.from<Address, U256>('balance'); // Create temporary variables _balance to manipulate persistent storage
    const _freeze = Store.from<Address, U256>('freeze'); // Create temporary variables _freeze to manipulate persistent storage
    ```

    The implementation of allowance is more complex, and we can implement it by prefixing it in front of Store.

    ```typescript
    // structure Store
    function getAllowanceDB(addr: Address): Store<Address, U256> {
        // Using'allowance'+ address as prefix to implement Store
        const prefix = Util.concatBytes(Util.str2bin('allowance'), addr.buf);
        return new Store<Address, U256>(prefix);
    }
    ```

- With state storage, there are corresponding state reading and changing functions:

    1. Read balance
    ```typescript
    // display balance
    export function balanceOf(addr: Address): U256 {
        return _balance.getOrDefault(addr, U256.ZERO);
    }    
    ```

    2. Transfer accounts
    ```typescript
    export function transfer(to: Address, amount: U256): void {
        // The msg object here is similar to solidity, which contains the current caller of the contract.
        const msg = Context.msg();
        assert(amount > U256.ZERO, 'amount is not positive');
        let b = balanceOf(msg.sender)
        // Assert that the balance is sufficient
        assert(b >= amount, 'balance is not enough');
        // + The operator is default safe for integer overflow
        _balance.set(to, balanceOf(to) + amount);
        _balance.set(msg.sender, balanceOf(msg.sender) - amount);
    }
    ```

    3. Frozen
    ```typescript
    export function freeze(amount: U256): void {
        const msg = Context.msg();
        assert(balanceOf(msg.sender) >= amount, 'balance is not enough');
        _balance.set(msg.sender, balanceOf(msg.sender) - amount);
        _freeze.set(msg.sender, freezeOf(msg.sender) + amount);
    }    
    ```


    4. Thaw
    ```typescript
    export function unfreeze(amount: U256): void {
        const msg = Context.msg();
        assert(freezeOf(msg.sender) >= amount, 'freeze is not enough');
        _freeze.set(msg.sender, freezeOf(msg.sender) - amount);
        _balance.set(msg.sender, balanceOf(msg.sender) + amount);
    }
    ```
    
    5. Allow others to use part of their own balance
    ```typescript
    export function approve(to: Address, amount: U256): void {
        const msg = Context.msg();
        assert(amount > U256.ZERO, 'amount is not positive');
        const db = getAllowanceDB(msg.sender);
        db.set(to, amount);
    }    
    ```
    
    6. See how much others agree us to use
    ```typescript
    export function allowanceOf(from: Address, sender: Address): U256 {
        const db = getAllowanceDB(from);
        return db.getOrDefault(sender, U256.ZERO);
    }
    ```
    
    7. Transfer from another person's account
    ```typescript
    export function transferFrom(from: Address, to: Address, amount: U256): void {
        const msg = Context.msg();
        assert(amount > U256.ZERO, 'amount is not positive');
        const allowance = allowanceOf(from, msg.sender);
        const balance = balanceOf(from);
        assert(balance >= amount, 'balance is not enough');
        assert(allowance >= amount, 'allowance is not enough');
    
        const db = getAllowanceDB(from);
        db.set(msg.sender, allowanceOf(from, msg.sender) - amount);
    
        _balance.set(from, balanceOf(from) - amount);
        _balance.set(to, balanceOf(to) + amount);
    }    
    ```


- Publishing events

    1. Event is a special ```class```，All fields of this class are read-only. There is only one constructor method. For example, the transfer event can be represented as follows:

    ```typescript
    @unmanaged class Transfer{
        constructor(readonly from: Address, readonly to: Address, readonly amount: U256)
    }
    ```

    Event class field type can only be Address, U256, String or ArrayBuffer, not bool, i64, u64 or f64

    2. Publish Events in Contracts

    Through ```Context.emit<T>``` function can post events outside of a contract, for example:

    ```typescript
    Context.emit<Transfer>(new Transfer(a, b, c));
    ```

    3. Out-of-contract Subscription Events

    sdk provides a way to subscribe to events out of contract

    ```js
    const c = new tool.Contract(addr, abi) // This assumes that both the contract address and abi are known
    const rpc = new tool.RPC('localhost', 19585)
    rpc.listen(c, 'Transfer', console.log) // Print events when a Transfer event is received
    ```


- Inter-contract call

    To invoke the function of Contract B in Contract A, first deploy Contract B between deployment contracts A and get the address of Contract B. Now assume the code of Contract B is as follows:

    ```typescript
    export function init(): void{

    }

    export function id(x: u64): u64{
        return x;
    }

    // The main file of all contracts must declare this function
    export function __idof(type: ABI_DATA_TYPE): u32 {
        return ___idof(type);
    }    
    ```

    Assuming contract B has been successfully deployed with  address B, now write the code for contract A

    ```typescript
    export function init(addr: Address): void{
        Globals.set<Address>('addr', addr);
    }

    export function getId(x: u64): u64{
        const addr = Globals.get<Address>('addr');
        // Construction parameters
        const bd = new ParametersBuilder();
        // The id function of contract B has only one parameter, the type is U64
        bd.push<u64>(26);
        return addr.call<u64>('id', bd.build(), 0);
    }
    ```

    Thus, when the contract A is deployed, the address of the contract B is set by the constructor. When the  ```getId``` method of the contract A is invoked later, the contract A will invoke the ```id``` method of the contract B.

- Contract Internal Deployment Contract

    To deploy Contract B in Contract A, first deploy a contract for B as a code template, assuming that Contract B has the following code:

    ```typescript
    export function init(name: string): void{
        Globals.set<string>('name', name);
    }
    ```

    Assuming contract B has been successfully deployed with address B, now write the code for contract A

    ```typescript
    export function init(template: Address): void{
        Globals.set<Address>('template', template);
    }    

    export function deploy(name: string): Address{
        const template = Globals.get<Address>('template');
        // Construction parameters
        const bd = new ParametersBuilder();
        // Contract B has only one constructor parameter of type string
        bd.push<string>(name);
        return Context.create(template.code(), template.abi(), bd.build(), 0);
    }
    ```

    This allows the address of contract B to be set through the constructor when contract A is deployed, and then when the ```delpoy``` method of contract A is later called, the constructor parameter of contract B is passed in, and the contract can be deployed by calling the contract.

































