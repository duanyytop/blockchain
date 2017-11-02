# 利用Truffle完成智能合约发布全流程

Truffle是一款以太坊智能合约非常流行的集开发、测试、发布的框架，它让开发者可以更专注于编码。

## 安装Truffle

Truffle的安装需要借助node的npm工具，所以需要提前安装node，node版本必须是5.0以上，Windows, Linux 和 Mac OS X都支持，安装命令如下：

```
npm install -g truffle
```

此外，truffle运行时还需要用到web3.js，所以你还需要安装web3.js，安装命令如下：

```
npm install web3
```

## 选择以太坊客户端

既然是以太坊开发工具，那么就需要提前安装好以太坊客户端，如果只是用于开发智能合约，那么安装[TestRPC](https://github.com/ethereumjs/testrpc)即可，如果需要发布到真实的以太坊环境，则需要安转以下任一客户端即可。

- [Geth (go-ethereum)](https://github.com/ethereum/go-ethereum) 基于go语言开发的以太坊客户端
- [WebThree (cpp-ethereum)](https://github.com/ethereum/cpp-ethereum) 基于c++开发的以太坊客户端
- [Parity](https://github.com/paritytech/parity) 基于Rust开发的以太坊客户端
- [More](https://www.ethereum.org/cli) 更多其他客户端


## 创建项目

Truffle为开发者提供了标准工程模板，只需要一条命令即可完成所有的初始化工作，命令如下：

```
$ mkdir myproject
$ cd myproject
$ truffle init
```
执行完成后，myproject目录下会出现以下文件：

- contracts/ 存放智能合约文件
- migrations/ 存放发布合约的脚本文件
- test/ 存放应用和合约的测试文件
- truffle.js 主要的truffle配置文件

contracts目录下已经包含了用于发布合约的Migrations.sol，migrations目录下包含了用于初始化的js脚本文件，其他文件都需要开发者自行补充。

## 配置网络信息

在正式开始之前，你还需要先配置一下网络信息，v3.0版相比v2.0版有一些变化，本文以最新的3.0版为准。具体配置方法为在truffle.js文件中输入以下信息：

```javascript
module.exports = {
  networks: {
    development: {
      host: "localhost",
      port: 8545,
      network_id: "*"
    },
    staging: {
      host: "localhost",
      port: 8546,
      network_id: 1337
    },
    ropsten: {
      host: "158.253.8.12",
      port: 8545,
      network_id: 3
    }
  }
};
```
网络信息就算配置好了，下文会有具体的用法介绍。

## 新建智能合约文件

接下来我们就以一个简单的智能合约文件为例子，详细介绍一下整个开发、测试流程。首先在contracts目录下新建一个智能合约solidity文件，文件名为Greeter.sol，文件内容如下：

```solidity
pragma solidity ^0.4.17;

contract Greeter         
{
    address creator;     
    string greeting;     

    function Greeter(string _greeting) public   
    {
        creator = msg.sender;
        greeting = _greeting;
    }

    function greet() constant returns (string)          
    {
        return greeting;
    }
    
    function setGreeting(string _newgreeting) 
    {
        greeting = _newgreeting;
    }
    
     /**********
     Standard kill() function to recover funds 
     **********/
    
    function kill()
    { 
        if (msg.sender == creator)
            suicide(creator);  // kills this contract and sends remaining funds back to creator
    }

}
```

这个合约逻辑很简单，setGreeting()可以向区块链中存入字符串信息，greet()方法用于获取之前存入的字符串信息，所以我们下文会通过先设置信息，然后再读取，通过比较信息是否相同来验证合约是否生效。

## 完善发布脚本

上文提到过migrations目录下已经包含了初始化的脚本文件，我们还需要一个发布上线的脚本文件，这个文件也很简单，文件名为2_deploy_contracts.js，代码如下：

```javascript
var Greeter = artifacts.require("./Greeter.sol");
module.exports = function(deployer) {
    deployer.deploy(Greeter);
};
```

## 编译智能合约

准备工作都已经完成了，此时我们就可以编译整个智能合约工程了，回到工程的跟目录，编译命令如下：

```
$ truffle compile
Compiling ./contracts/Announcement.sol...
Compiling ./contracts/Migrations.sol...
Writing artifacts to ./build/contracts
```

编译过程中可能会出现warning，可以先行忽略，完成后会在根目录下多出一个build文件夹，存放了solidity文件编译后的[abi](https://github.com/ethereum/wiki/wiki/Solidity,-Docs-and-ABI)文件。

## 启动以太坊客户端

编译完成之后就可以测试了，不过在测试之前还需要启动以太坊客户端，毕竟智能合约需要运行在以太坊上，如果你安装了TestRPC，那么直接启动就可以了，启动命令如下：

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
以太坊的测试服务就算是起来了，等一下我们发起交易时，就可以在这里看到日志。

## 部署合约(migrate)

合约已经编译完成，以太坊测试服务也已经正常起来，那么接下来就可以部署合约了，部署的命令如下：

```
truffle migrate
Using network 'development'.

Running migration: 1_initial_migration.js
  Deploying Migrations...
  ... 0x45c219d19d14da2d412bbff5087c89783fdba09ba2c81c35bd666ef1e7f6130d
  Migrations: 0xca8b6137af3fad038448855f34a473fd5ced0d96
Saving successful migration to network...
  ... 0x56dd511df86afb74e0c10f7efa924e93d3f852cf0b2d9565c7fab2a7b9b16e9c
Saving artifacts...
Running migration: 2_deploy_contracts.js
  Deploying Announcement...
  ... 0x3b49d899673a5f01f334bf43aa072aeb2fd67f87df775a38b555842661154c48
  Announcement: 0xcf52ecd3fa1ec5fa436e22fe606e725ce5b4736d
Saving successful migration to network...
  ... 0x6546ae69d0abd322b826d852801387fe028dcec9812ffdae114cfaed34cee719
Saving artifacts...

```
运行这条命令过程中可能会遇到以下两种错误信息：

```
No network specified. Cannot determine current network
```
这条错误信息是之前没有在truffle.js文件中配置网络信息，如果配置了，就不会出现这条错误。

```
Exceeds block gas limit
或者
Intrinsic gas too low
或者
tx has a higher gas limit than the block
```
这些错误信息都是因为gas信息没有配置或者配置有问题，解决方式是在truffle.js中添加gas信息，代码如下：

```
development: {
  host: "localhost",
  port: 8545,
  network_id: "*",
  gas:4712388
},
```
这个gas值就是你需要支付的费用，单位是wei，关于这个值是如何来的，其实是可以查询到的，具体查询方式如下：

```
// 先进入以太坊控制台

$ truffle consloe
truffle(development)> web3.eth.getBlock("pending").gasLimit
4712388

```
这里的getBlock方法是web3中的一个api接口，参数一般是具体的块高度值，或者是一些特定的字符串，定义如下：

- Number - a block number
- String - "earliest", the genisis block
- String - "latest", the latest block (current head of the blockchain)
- String - "pending", the currently mined block (including pending transactions)

更多信息可以参考web3官方api文档。

## 测试智能合约

合约部署成功后，我们就可以测试合约的执行效果了。还记得根目录下的test文件夹吧，这里放置的就是测试js文件，之所以是js文件，是因为我们利用了web3.js，可以更加方便做各种复杂的测试。

在test目录下新建greeter.js文件，代码如下：

```javascript

const Greeter = artifacts.require("./Greeter.sol");

const Web3 = require('web3');		// 引入web3
// create an instance of web3 using the HTTP provider.
// NOTE in mist web3 is already available, so check first if it's available before instantiating
const web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));

contract('Greeter', function(accounts) {

    describe("set/get test", function () {
        it("set something to ethereum by setGreeting method", async function() {
            // Get a reference to the deployed Greeter contract, as a JS object.
            const instance = await Greeter.deployed();       
        
            // 先通过set方法存储一个Hello字符串，然后再通过get方法获取，并验证是否是该字符串
            await instance.setGreeting("Hello");
            const content = await instance.greet();
            console.log("greet(): " + content);
            assert.equal(content, "Hello");

            // 以下是利用web3可以获取的以太坊信息，更多接口调用可以参考web3官方api文档
            var version = web3.version.api;
            console.log(version); // "0.20.0"
            
            var version = web3.version.node;
            console.log(version); // "EthereumJS TestRPC/v1.1.3/ethereum-js"
      
          });
    });
    
  });
```
然后执行命令： `truffle test`，如果一切正常的话，就可以看到如下日志信息：

```
Using network 'development'.

  Contract: Greeter
    set/get test
greet(): Hello
      ✓ set something to ethereum by setGreeting method (52ms)

  1 passing (66ms)

```
可以看到测试成功，也就是说我们的合约文件是正常可用的。

以上就是利用truffle完成整个合约从开发到部署再到测试验证的整个过程，希望对刚入门的你有帮助。








