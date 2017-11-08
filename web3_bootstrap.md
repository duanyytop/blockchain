## 利用Web3完成智能合约发布测试全流程

`web3` 提供的 `web3.js` 库可以使开发以太坊智能合约变得更加方便快捷，`web3.js` 通过底层的 `RPC` 调用完成与以太坊网络的通信，利用 `web3` 提供的 `api`，我们可以很方便地完成合约的编译、部署、测试，`solidity` 本身并支持单元测试，但是通过 `web3` 可以弥补这一缺憾。

### 启动以太坊客户端

我们要想完成以太坊智能合约的部署和测试，首先需要先启动以太坊客户端，这里推荐[TestRPC](https://github.com/ethereumjs/testrpc)，如果你已经安装了 `TestRPC`，那么直接启动就可以了，启动命令如下：

```
$ testrpc
EthereumJS TestRPC v4.1.3 (ganache-core: 1.1.3)

Available Accounts
==================
(0) 0xad418f13ece0704260ce556a5884ced86d0cb381
(1) 0x069d85f282c904ed5b76f6eef15b022ee244c6de
(2) 0xbcee11894ed639d922b2340dc5cfa68c0b7ec947
(3) 0x0d81df16753f7068e450b2a06048e62c6efbf8d6
(4) 0xdf748b47aff450aa01ef8b07d5a112db16dad451
(5) 0xa88e69552773060eb5224087f6c2557a0d098104
(6) 0xd7749c51e2512983bcff5fdcf6dd8298757f1eee
(7) 0xd8fd234d11f71ce93918ce577ca2c858500c80a2
(8) 0xc2b80d9588c124a5edac5d6a5d241acea6a8df23
(9) 0x7a7348ab5c7e2bd0a16cc43c221e5aed4871199b

Private Keys
==================
(0) 908b601740dab78f983bfc79245cbc2a81cfaa53e11377a62b3ecfb015db554c
(1) edd6df731655df13743c4fb3a7900ae5ac3cbad4dd41a0abbfe8ec533aa0a164
(2) 4e82c7ca0678872a21b2aacf258eed470785e7f4539d2ba184a43c7d34f378fa
(3) 721ffc93c2a9e17da04d18f7dbc997f482bd14e5807f9b5414e34cf7a329d0ab
(4) 20226cddeb3e3c567924099fac28a9e39795de2ab9b9f8442f0ccdc74fe3f791
(5) 1b68649e575ed416e0a01dd1ddadd97108ca2a334aefc83ccbd3cef7f9942f1a
(6) ed54ab42c3ab27d322fe51a312ae0ee5317938bbf4926ed2b3938719480e4501
(7) 099867a47752f308a085626498b645c043539463cac446416befa0b8cd86cd05
(8) e4f3940dda7e236334548315a834f9b3002db21e96aa1d4760dfdd102e483e77
(9) 463c3e01803b191ebda1863a5d1b752f55a5c7e1ab31502a11f84719ceb32b5b

HD Wallet
==================
Mnemonic:      brave wood mimic club squeeze loop speak lamp alter fall cabbage happy
Base HD Path:  m/44'/60'/0'/0/{account_index}

Listening on localhost:8545

```

### 安装web3

要想在你的项目中使用 `web3`，首先你需要安装 `web3`，因为 `web3.js` 是基于 `JavaScript` 的库，所以直接借助 `npm` 即可完成安装，安转命令如下：

```
$ npm install web3
```
### 引入web3

安装完成后，你就可以开始在项目中使用 `web3` 了，首先你就需要构建一个 `web3` 实例，推荐如下方式引入 `web3` 实例：

```javascript
// To make sure you don't overwrite the already set provider when in mist
// check first if the web3 is available
if (typeof web3 !== 'undefined') {
  web3 = new Web3(web3.currentProvider);
} else {
  // set the provider you want from Web3.providers
  web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
}
```

拿到 `web3` 实例后，你就可以调用[web3](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethdefaultblock)的各种 `api` 接口了。

### 编译智能合约

`web3` 可以直接引入 `solc` 完成 `solidity` 合约文件的编译，`solc` 是专门用于编译 `solidity` 文件的工具，具体安装方法也很简单。

```
npm install -g solc
```
接下来就是具体的编译环节了，对于一项node工程，我们通常会将所有的依赖库通过 `package.json` 文件进行统一管理，你可以通过 `npm init` 命令来自动完成初始化，然后在 `package.json` 文件中手动添加所要依赖的库，如果你不清楚需要依赖库的版本号，可以通过[npmjs](https://www.npmjs.com/package/lib)网站查找。

```javascript
const fs = require('fs');
const solc = require('solc');
const Web3 = require('web3');

// Connect to local Ethereum node
if (typeof web3 !== 'undefined') {
  web3 = new Web3(web3.currentProvider);
} else {
  // set the provider you want from Web3.providers
  web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
}

// Compile the source code, SimpleStorage.sol is your solidity file
const input = fs.readFileSync('SimpleStorage.sol');
const output = solc.compile(input.toString(), 1);
// console.log("compile output: " + JSON.stringify(output));
```
如果你有兴趣可以打印一下编译后的输出值，里面包含了大量的智能合约信息，包括合约字节码、`abi` 文件、方法 `hash`值等，这些在部署合约的时候，都需要用到。

### 部署智能合约

编译完成后，我们就可以拿到合约文件的字节码、`abi` 文件以及方法 `hash` 值了，以太坊智能合约的部署本质上还是在发交易，只不过需要额外传入一些其他参数，如果我们借助 `web3.js` 库的话，我们就不需要关心参数的拼接和具体的请求调用，这就极大的方便了智能合约单元测试的编写和验证。

```javascript
// Note: ':SimpleStorag' is coming from your solidity file name, and ':' can not be omitted.
const bytecode = output.contracts[':SimpleStorage'].bytecode;
const abi = JSON.parse(output.contracts[':SimpleStorage'].interface);

// Get contract object through abi parameter
const contract = web3.eth.contract(abi);

// Deploy contract instance
const contractInstance = contract.new({
    data: '0x' + bytecode,    // contract byte code
    from: web3.eth.coinbase,  // the address of sender
    gas: 900000*2             // the gas amount of contract
}, (err, res) => {
    if (err) {
        console.log("Deploy smart contract error: " + err);
        return;
    }
    // Log the tx, you can explore status with eth.getTransaction()
    console.log(res.transactionHash);
    // If we have an address property, the contract will be deployed successfully
    if (res.address) {
        console.log("Deploy smart contract successfully and Contract address: " + res.address);
        // Let's test the deployed contract
        testContract(res.address);
    }
});
```
代码的注释应该已经很清楚了，可以看到 `web.js` 帮我们封装了合约实例、交易发起方法等，如果部署没有问题，那么接下来我们就可以测试合约了。

这里的 `SimpleStorage.sol` 文件是一个非常简单的合约，只包含了两个方法，代码如下：

```solidity
pragma solidity ^0.4.18;

contract SimpleStorage {
    string storedData;

    function set(string x) {
        storedData = x;
    }

    function get() constant returns (string) {
        return storedData;
    }
}

```
如果我们要验证合约是否生效，只需要先调用 `set` 方法存储一串任意的字符串，然后再调用 `get` 方法验证存储是否生效即可，所以我们的测试代码就很简单了。

### 测试智能合约

测试代码如下：

```javascript
function testContract(address) {
    // Reference to the deployed contract
    const storage = contract.at(address);

    // We storage a string of “Hello Solidity”, sender andress is coinbase
    storage.set("Hello Solidity", {from: web3.eth.coinbase}, (err, res) => {
        if(!err) {
        	// Calling set function is successful and you can get the receipte hash
            console.log("set data successfully, and response: " + res);
            const result = storage.get.call({from: web3.eth.coinbase});
            // Verify smart contract 
            console.log("set and get function test result: " + (result == "Hello Solidity"));
        } else {
            console.log("set data error, and error is " + err);
        }
    })
}
```

如果你看到测试日志返回 `true`，那就说明合约是正常的，当然我们还可以测试很多边界问题，这些就留待你自己定制吧。

以上就是利用 `web.js` 库完成整个合约的编译、部署和测试，这要比手动编译，并且拼接参数调用RPC方法简单很多，当然也更有利于编写单元测试。