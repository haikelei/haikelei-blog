---
author: Gary Lu
pubDatetime: 2024-12-08T15:00:00+08:00
modDatetime: 2024-12-08T15:00:00+08:00
title: Solidity Smart Contract Development - Fundamentals
slug: solidity-smart-contract-development-fundamentals
featured: true
draft: false
tags:
  - blockchain
  - solidity
  - smart-contracts
  - ethereum
  - web3
  - programming
description: A comprehensive guide to Solidity smart contract development fundamentals, covering core syntax, data types, functions, and best practices for blockchain development.
---

# Solidity Smart Contract Development - Fundamentals

## Introduction

As a programmer with over 10 years of experience, I've witnessed the evolution of web development from traditional server-client architectures to the decentralized web3 ecosystem. Smart contract development represents one of the most exciting paradigms in modern programming, combining traditional software engineering principles with blockchain-specific considerations.

Recently, I've been working extensively with EVM-based blockchain projects involving data certification, verification, and migration contracts. This experience has reinforced my belief that Solidity is an essential skill for any developer looking to participate in the web3 revolution. This article serves as the first in a series covering Solidity fundamentals, designed to help experienced developers transition into blockchain development.

## Smart Contracts & Solidity Language

Smart contracts are programs that run on the blockchain, allowing contract developers to interact with on-chain assets and data. Users can call contracts through their blockchain accounts to access assets and data. Due to blockchain's characteristics - maintaining block history through chain structure, decentralization, and immutability - smart contracts offer greater fairness and transparency compared to traditional applications.

However, because smart contracts need to interact with the blockchain, deployment and data write operations consume fees, and data storage and modification costs are relatively high. Therefore, when designing contracts, careful consideration of resource consumption is essential. Additionally, conventional smart contracts cannot be modified once deployed, so contract design must carefully consider security, upgradability, and extensibility.

Solidity is a contract-oriented, high-level programming language created specifically for implementing smart contracts. It runs on the EVM (Ethereum Virtual Machine), with syntax similar to JavaScript. It's currently the most popular smart contract language and essential for entering blockchain and Web3 development. Solidity provides relatively comprehensive solutions for the contract development challenges mentioned above.

## Development/Debugging Tools

Unlike conventional programming languages, Solidity smart contract development often cannot be directly debugged through an IDE or local environment but requires interaction with a blockchain node. Development and debugging typically don't interact directly with the mainnet (where real assets, data, and business reside), as this would incur high transaction fees. Current development and debugging primarily use the following methods and frameworks:

1. **[Remix IDE](https://remix.ethereum.org)** - Ethereum's official browser-based development tool providing a complete IDE, compilation tools, deployment debugging test node environment, and accounts. Very convenient for testing and widely used in both learning and production environments.

2. **[Truffle](https://github.com/trufflesuite/truffle)** - A popular JavaScript-based Solidity contract development framework providing complete development, testing, and debugging toolchains for local or remote network interaction.

3. **[Brownie](https://github.com/eth-brownie/brownie)** - A Python-based Solidity contract development framework offering convenient debugging and testing toolchains with clean Python syntax.

4. **[Hardhat](https://github.com/NomicFoundation/hardhat)** - Another JavaScript-based development framework with a rich plugin system, suitable for developing complex contract projects.

Beyond development frameworks, effective Solidity development requires familiarity with additional tools:

1. **[Visual Studio Code](https://code.visualstudio.com)** with [Solidity extension](https://marketplace.visualstudio.com/items?itemName=juanblanco.solidity) for better syntax highlighting and IntelliSense.

2. **[MetaMask](https://metamask.io)** - A popular wallet application allowing browser plugin interaction with testnets and mainnet for convenient debugging.

3. **[Ganache](https://trufflesuite.com/ganache/)** - An open-source virtual local node providing a virtual blockchain network for interaction with Web3.js, Remix, or framework tools.

4. **[Infura](https://infura.io)** - An Infrastructure as a Service (IaaS) product for accessing Ethereum nodes through APIs, offering convenient debugging closer to production environments.

5. **[OpenZeppelin](https://www.openzeppelin.com)** - Provides extensive contract development libraries and applications, balancing security and stability while improving developer experience and reducing contract development costs.

## Contract Compilation/Deployment

Solidity contracts are files with `.sol` extensions that cannot execute directly but must be compiled into EVM (Ethereum Virtual Machine) recognizable bytecode to run on-chain.

![compile_solidity](https://image.pseudoyu.com/images/compile_solidity.png)

After compilation, contract accounts deploy to the blockchain, and other accounts can interact with contracts through wallets to implement on-chain business logic.

## Core Syntax

Having understood Solidity development, debugging, and deployment, let's explore Solidity's core syntax.

### Data Types

Similar to common programming languages, Solidity has built-in data types.

#### Basic Data Types

- **`boolean`** - Boolean type with `true` and `false` values, defined as `bool public boo = true;`, default value `false`
- **`int`** - Integer type, can specify `int8` to `int256`, defaults to `int256`, defined as `int public int = 0;`, default value `0`. Check min/max with `type(int).min` and `type(int).max`
- **`uint`** - Unsigned integer type, can specify `uint8`, `uint16`, `uint256`, defaults to `uint256`, defined as `uint8 public u8 = 1;`, default value `0`
- **`address`** - Address type, defined as `address public addr = 0xCA35b7d915458EF540aDe6068dFe2F44E8fa733c;`, default value `0x0000000000000000000000000000000000000000`
- **`bytes`** - Shorthand for `byte[]`, includes fixed-size and variable arrays, defined as `bytes1 a = 0xb5;`

#### Enum

`Enum` is an enumeration type defined with the following syntax:

```solidity
enum Status {
    Unknown,
    Start,
    End,
    Pause
}
```

Update and initialize with:

```solidity
// Instantiate enum type
Status public status;

// Update enum value
function pause() public {
    status = Status.Pause;
}

// Initialize enum value
function reset() public {
    delete status;
}
```

#### Arrays

Arrays store ordered collections of similar elements, defined as `uint[] public arr;`. Size can be pre-specified like `uint[10] public myFixedSizeArr;`.

Memory arrays must have fixed size: `uint[] memory a = new uint[](7);`

Basic array operations:

```solidity
// Define array type
uint[] public arr;

// Add data
arr.push(7);

// Remove last element
arr.pop();

// Delete element at index
delete arr[1];

// Get array length
uint len = arr.length;
```

#### Mapping

`mapping` is a mapping type defined as `mapping(keyType => valueType)`. Keys must be built-in types like `bytes`, `string`, or contract types, while values can be any type including nested `mapping`. Note that `mapping` types cannot be iterated - custom indexing implementation is required for traversal.

Operations:

```solidity
// Define nested mapping type
mapping(string => mapping(string => string)) nestedMap;

// Set value
nestedMap[id][key] = "0707";

// Read value
string value = nestedMap[id][key];

// Delete value
delete nestedMap[id][key];
```

#### Struct

`struct` is a structure type. For complex business logic, we often need custom structures combining related data:

```solidity
contract Struct {
    struct Data {
        string id;
        string hash;
    }

    Data public data;

    // Add data
    function create(string calldata _id) public {
        data = Data{id: _id, hash: "111222"};
    }

    // Update data
    function update(string memory _hash) public {
        // Query data
        string memory id = data.id;

        // Update
        data.hash = _hash;
    }
}
```

Structures can also be defined in separate files and imported as needed:

```solidity
// 'StructDeclaration.sol'
struct Data {
    string id;
    string hash;
}
```

```solidity
// 'Struct.sol'
import "./StructDeclaration.sol";

contract Struct {
    Data public data;
}
```

### Variables/Constants/Immutable

Variables are changeable data structures in Solidity, divided into three types:

- **`local`** variables - Defined in methods, not stored on-chain: `string var = "Hello";`
- **`state`** variables - Defined outside methods, stored on-chain: `string public var;`. Writing values sends transactions, reading doesn't
- **`global`** variables - Provide blockchain information: `uint timestamp = block.timestamp;`, `address sender = msg.sender;`

Variables can be declared with different keywords representing different storage locations:

- **`storage`** - Stored on-chain
- **`memory`** - In memory, exists only during method calls
- **`calldata`** - Exists when passed as method parameters

Constants are unchangeable variables that save gas fees: `string public constant MY_CONSTANT = "0707";`

`immutable` is a special type that can be initialized in the `constructor` but cannot be changed afterward. Using these types effectively saves gas and ensures data security.

### Functions

Functions define specific business logic in Solidity.

#### Visibility Declarations

Functions have different visibility levels:

- **`public`** - Any contract can call
- **`private`** - Only the defining contract can call internally
- **`internal`** - Only inheriting contracts can call
- **`external`** - Only other contracts and accounts can call

Query functions have different declarations:

- **`view`** - Can read variables but cannot modify
- **`pure`** - Cannot read or modify variables

#### Function Modifiers

`modifier` function modifiers can be called before/after function execution, mainly for access control, input parameter validation, and reentrancy attack prevention:

```solidity
modifier onlyOwner() {
    require(msg.sender == owner, "Not owner");
    _;
}

modifier validAddress(address _addr) {
    require(_addr != address(0), "Not valid address");
    _;
}

modifier noReentrancy() {
    require(!locked, "No reentrancy");
    locked = true;
    _;
    locked = false;
}
```

Using modifiers in function declarations:

```solidity
function changeOwner(address _newOwner) public onlyOwner validAddress(_newOwner) {
    owner = _newOwner;
}

function decrement(uint i) public noReentrancy {
    x -= i;

    if (i > 1) {
        decrement(i - 1);
    }
}
```

#### Function Selectors

When functions are called, the first four bytes of `calldata` specify which function to call, called the function selector:

```solidity
addr.call(abi.encodeWithSignature("transfer(address,uint256)", 0xSomeAddress, 123))
```

Pre-calculating function selectors can save gas fees:

```solidity
contract FunctionSelector {
    function getSelector(string calldata _func) external pure returns (bytes4) {
        return bytes4(keccak256(bytes(_func)));
    }
}
```

### Conditional/Loop Structures

#### Conditionals

Solidity uses `if`, `else if`, `else` keywords for conditional logic:

```solidity
if (x < 10) {
    return 0;
} else if (x < 20) {
    return 1;
} else {
    return 2;
}
```

Shorthand form:

```solidity
return x < 20 ? 1 : 2;
```

#### Loops

Solidity uses `for`, `while`, `do while` keywords for loops, but the latter two easily reach gas limits, so `for` is primarily used:

```solidity
for (uint i = 0; i < 10; i++) {
    // Business logic
}
```

### Contracts

#### Constructors

Solidity's `constructor` executes during contract creation, mainly for initialization:

```solidity
constructor(string memory _name) {
    name = _name;
}
```

With inheritance, constructors execute in inheritance order.

#### Interfaces

`Interface` declarations enable contract interaction with requirements:

- Cannot implement any methods
- Can inherit other interfaces
- All methods must be declared `external`
- Cannot declare constructors
- Cannot declare state variables

Interface definition:

```solidity
contract Counter {
    uint public count;

    function increment() external {
        count += 1;
    }
}

interface ICounter {
    function count() external view returns (uint);
    function increment() external;
}
```

Interface usage:

```solidity
contract MyContract {
    function incrementCounter(address _counter) external {
        ICounter(_counter).increment();
    }

    function getCount(address _counter) external view returns (uint) {
        return ICounter(_counter).count();
    }
}
```

#### Inheritance

Solidity contracts support multiple inheritance using the `is` keyword.

Functions can be overridden - parent contract methods need `virtual` declaration, overriding methods need `override`:

```solidity
// Define parent contract A
contract A {
    function foo() public pure virtual returns (string memory) {
        return "A";
    }
}

// B contract inherits A and overrides function
contract B is A {
    function foo() public pure virtual override returns (string memory) {
        return "B";
    }
}

// D contract inherits B, C and overrides function
contract D is B, C {
    function foo() public pure override(B, C) returns (string memory) {
        return super.foo();
    }
}
```

Note: inheritance order affects business logic, `state` variables cannot be overridden.

Child contracts can call parent contracts directly or through `super`:

```solidity
contract B is A {
    function foo() public virtual override {
        // Direct call
        A.foo();
    }

    function bar() public virtual override {
        // Call through super keyword
        super.bar();
    }
}
```

#### Contract Creation

Solidity allows creating contracts from other contracts using `new`:

```solidity
function create(address _owner, string memory _model) public {
    Car car = new Car(_owner, _model);
    cars.push(car);
}
```

Solidity 0.8.0+ supports `create2` for deterministic contract creation:

```solidity
function create2(address _owner, string memory _model, bytes32 _salt) public {
    Car car = (new Car){salt: _salt}(_owner, _model);
    cars.push(car);
}
```

#### Importing Contracts/External Libraries

Complex business often requires multiple contract cooperation. Use `import` for local imports `import "./Foo.sol";` and external imports `import "https://github.com/owner/repo/blob/branch/path/to/Contract.sol";`.

External libraries are similar to contracts but cannot declare state variables or send assets. If all library methods are `internal`, they're embedded in contracts; otherwise, libraries need pre-deployment and linking:

```solidity
library SafeMath {
    function add(uint x, uint y) internal pure returns (uint) {
        uint z = x + y;
        require(z >= x, "uint overflow");
        return z;
    }
}
```

```solidity
contract TestSafeMath {
    using SafeMath for uint;
}
```

#### Events

Events are crucial contract design elements allowing information recording on blockchain. DApps can monitor event data for business logic implementation with low storage costs:

```solidity
// Define event
event Log(address indexed sender, string message);
event AnotherLog();

// Emit event
emit Log(msg.sender, "Hello World!");
emit Log(msg.sender, "Hello EVM!");
emit AnotherLog();
```

Events can have `indexed` attributes (maximum three) for parameter filtering: `var event = myContract.transfer({value: ["99","100","101"]});`

### Error Handling

On-chain error handling is important for contract development. Solidity provides several error-throwing methods:

`require` validates conditions before execution, throwing exceptions if unsatisfied:

```solidity
function testRequire(uint _i) public pure {
    require(_i > 10, "Input must be greater than 10");
}
```

`revert` marks errors and performs rollback:

```solidity
function testRevert(uint _i) public pure {
    if (_i <= 10) {
        revert("Input must be greater than 10");
    }
}
```

`assert` requires condition satisfaction:

```solidity
function testAssert() public view {
    assert(num == 0);
}
```

Note: In Solidity, errors rollback all state changes in transactions, including assets, accounts, and contracts.

`try / catch` can catch errors but only from external function calls and contract creation:

```solidity
event Log(string message);
event LogBytes(bytes data);

function tryCatchNewContract(address _owner) public {
    try new Foo(_owner) returns (Foo foo) {
        emit Log("Foo created");
    } catch Error(string memory reason) {
        emit Log(reason);
    } catch (bytes memory reason) {
        emit LogBytes(reason);
    }
}
```

### Payable Keyword

We can declare `payable` to allow methods to receive `ether`:

```solidity
// Address type can be declared payable
address payable public owner;

constructor() payable {
    owner = payable(msg.sender);
}

// Method declared payable to receive Ether
function deposit() public payable {}
```

### Ether Interaction

Ether interaction is an important smart contract application scenario, divided into sending and receiving.

#### Sending

Mainly through `transfer`, `send`, and `call` methods. `call` is recommended for actual applications due to improved reentrancy attack prevention:

```solidity
contract SendEther {
    function sendViaCall(address payable _to) public payable {
        (bool sent, bytes memory data) = _to.call{value: msg.value}("");
        require(sent, "Failed to send Ether");
    }
}
```

For calling other functions, use `delegatecall`:

```solidity
contract B {
    uint public num;
    address public sender;
    uint public value;

    function setVars(uint _num) public payable {
        num = _num;
        sender = msg.sender;
        value = msg.value;
    }
}

contract A {
    uint public num;
    address public sender;
    uint public value;

    function setVars(address _contract, uint _num) public payable {
        (bool success, bytes memory data) = _contract.delegatecall(
            abi.encodeWithSignature("setVars(uint256)", _num)
        );
    }
}
```

#### Receiving

Receive `Ether` using `receive() external payable` and `fallback() external payable`.

`fallback()` is called when a function accepting no parameters returns no parameters, when `Ether` is sent to a contract but `receive()` is not implemented, or when `msg.data` is non-empty:

```solidity
contract ReceiveEther {
    // When msg.data is empty
    receive() external payable {}

    // When msg.data is non-empty
    fallback() external payable {}

    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
}
```

### Gas Fees

Executing transactions in EVM requires gas fees. `gas spent` indicates gas amount needed, `gas price` is the unit price. `Ether` and `Wei` are price units: 1 ether == 1e18 wei.

Contracts limit Gas: `gas limit` set by transaction initiators (maximum gas to spend), `block gas limit` set by blockchain networks (maximum gas allowed per block).

Contract development must consider gas optimization techniques:

1. Use `calldata` instead of `memory`
2. Load state variables into memory
3. Use `i++` instead of `++i`
4. Cache array elements

```solidity
function sumIfEvenAndLessThan99(uint[] calldata nums) external {
    uint _total = total;
    uint len = nums.length;

    for (uint i = 0; i < len; ++i) {
        uint num = nums[i];
        if (num % 2 == 0 && num < 99) {
            _total += num;
        }
    }

    total = _total;
}
```

## Summary

This covers Solidity fundamentals in our first article of the series. Future articles will explore common applications and practical coding techniques. As a developer with extensive experience in traditional web development, I find Solidity's approach to decentralized computing both challenging and rewarding. The immutable nature of smart contracts demands a higher standard of code quality and security awareness, making it an excellent skill for experienced developers looking to expand into the blockchain space.
