# Foundry 备忘录


对[foundry-cheatsheet](https://github.com/dabit3/foundry-cheatsheet)的翻译


![Foundry 备忘录](header.jpg)

Foundry GitHub - [https://github.com/foundry-rs](https://github.com/foundry-rs)   
Foundry Book - [https://book.getfoundry.sh/](https://book.getfoundry.sh/)   
Installing Foundry - [https://github.com/foundry-rs/foundry#installation](https://github.com/foundry-rs/foundry#installation)

Foundry 由三个部分组成:

[Forge](https://github.com/foundry-rs/foundry/tree/master/forge): 以太坊测试框架（如Truffle，Hardhat和DappTools）.   
[Cast](https://github.com/foundry-rs/foundry/tree/master/cast): 用于与EVM智能合约交互，发送交易和获取链数据.   
[Anvil](https://github.com/foundry-rs/foundry/tree/master/anvil): 本地以太坊节点，类似于Ganache或Hardhat网络.

Forge 既包括一个CLI，也包括一个[标准库](https://github.com/foundry-rs/forge-std).

有关Foundry的介绍，请查看以下资源:

- [Foundry vs Hardhat: Differences in performance and developer experience](https://chainstack.com/foundry-hardhat-differences-performance/)
- [Smart Contract Development with Foundry (Video)](https://www.youtube.com/watch?v=uelA2U9TbgM)
- [Building and testing smart contracts with Foundry (tutorial)](https://nader.mirror.xyz/6Mn3HjrqKLhHzu2balLPv4SqE5a-oEESl4ycpRkWFsc)

电报频道[here](https://t.me/foundry_support)

## 初始化新项目

从一个空文件夹初始化新项目:

```sh
forge init
```

或者定义一个新文件夹，在其中创建项目:

```sh
forge init my-app
```

会创建4个文件夹:

__src__ - 用于存放智能合约

__script__ - 部署脚本

__test__ - 测试用例

__lib__ - 这类似于node_modules

### Remappings

可以重新映射依赖项，使其更易于导入。Forge会自动尝试为您推断一些重新映射:

```sh
$ forge remappings

ds-test/=lib/solmate/lib/ds-test/src/
forge-std/=lib/forge-std/src/
```

如果使用的是 VS Code，则可以通过安装 [vscode-solidity](https://github.com/juanfranblanco/vscode-solidity) 来配置重新映射。

在此处了解有关重新映射工作原理的[详细信息](https://book.getfoundry.sh/projects/dependencies.html?highlight=remappings#remapping-dependencies)

## 测试

可以运行 `forge test` 命令来进行测试.

```sh
forge test
```

任何用`test` 关键字开头的函数都是测试函数.

```solidity
function testAssertEquality() public {
  int some_int = 1;
  assertEq(1, some_int);
}
```

通常, 测试脚本放置在 `src/test`文件夹中,并且文件名以 `.t.sol`结尾。

Foundry使用 `Dappsys Test` (DSTest) 来提供基础的日志和断言功能.

断言的一些例子:

```solidity
assertTrue
assertEq
assertEqDecimal
assertEq32
assertEq0
assertGt
assertGtDecimal
assertGe
assertGeDecimal
assertLt
assertLtDecimal
assertLe
assertLeDecimal
```

详细列表这里[查看](https://book.getfoundry.sh/reference/ds-test.html#asserting).

从一个基本的计数器合约开始,如下所示 (__src/Counter.sol__):

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

contract Counter {
    int private count;
    
    constructor(int _count) {
        count = _count;
    }

    function incrementCounter() public {
        count += 1;
    }
    function decrementCounter() public {
        count -= 1;
    }

    function getCount() public view returns (int) {
        return count;
    }
}
```

我们的测试这样写:(__test/Counter.t.sol__):

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import 'src/Counter.sol';

contract ContractTest is Test {
    Counter counter;
    function setUp() public {
        counter = new Counter(10);
    }

    function testGetCount() public {
        int value = counter.getCount();
        assertEq(value, 10);
        emit log_int(value);
    }

    function testIncrement() public {
        counter.incrementCounter();
        counter.incrementCounter();
        int value = counter.getCount();
        assertEq(value, 12);
        emit log_int(value);
    }

    function testDecrement() public {
        counter.decrementCounter();
        int value = counter.getCount();
        assertEq(value, 9);
        emit log_int(value);
    }
}
```



### 日志和跟踪

你可以注意到了，我们使用`log_int`来打印日志。你也可以使用以下的emit:

```solidity
emit log(string);

emit log_address(address);
emit log_bytes32(bytes32);
emit log_int(int);
emit log_uint(uint);
emit log_bytes(bytes);
emit log_string(string);

emit log_named_address(string key, address val);
emit log_named_bytes32(string key, bytes32 val);
emit log_named_decimal_int(string key, int val, uint decimals);
emit log_named_decimal_uint(string key, uint val, uint decimals);
emit log_named_int(string key, int val);
emit log_named_uint(string key, uint val);
emit log_named_bytes(string key, bytes val);
emit log_named_string(string key, string val);
```

默认 `forge test` 仅显示通过和失败测试的日志,你可以使用`-v`来展示更多的日志:

__Level 2 (`-vv`)__: 还会显示测试期间发出的日志。这包括来自测试的断言错误，显示诸如预期与实际等信息。
__Level 3 (`-vvv`)__: 还会显示失败测试的堆栈跟踪。
__Level 4 (`-vvvv`)__: 显示所有测试的堆栈跟踪，并显示失败测试的设置跟踪。 
__Level 5 (`-vvvvv`)__: 始终显示堆栈跟踪和设置跟踪。

为了查看上面我们写的日志, 我们最少要输入`-vv`:

```sh
forge test -vv
```

### Cheatcodes

Cheatcodes 为您提供了额外的断言、更改 EVM 状态、模拟数据等的功能。

这个例子, 你可以使用 [prank](https://book.getfoundry.sh/cheatcodes/prank.html) 和 [startPrank](https://book.getfoundry.sh/cheatcodes/start-prank.html)来模拟用户:

```solidity
address bob = address(0x1);
vm.startPrank(bob);
```

使用[setNonce](https://book.getfoundry.sh/cheatcodes/set-nonce.html)给账户设置nonce。

```solidity
vm.setNonce(address(100), 1234);
```

使用[roll](https://book.getfoundry.sh/cheatcodes/roll.html)设置`block.number`。

```solidity
vm.roll(100);
emit log_uint(block.number); // 100
```

使用[warp](https://book.getfoundry.sh/cheatcodes/warp.html)设置`block.timestamp`。

```solidity
vm.warp(1641070800);
emit log_uint(block.timestamp); // 1641070800
```

这里[查看更多](https://github.com/foundry-rs/foundry/tree/master/forge#cheat-codes)

### 模拟用户

在上面，我们使用`.prank` 或 `.startPrank`来模拟用户,让我们看看是怎么做到的吧!

假设我们有一个ERC721合同，我们希望确保只有代币的所有者才能转移或销毁该代币。我们的测试可能如下:

```solidity
// only the owner can transfer
function testTransferToken() public {
    // mint the token to bob's address
    erc721 = new ERC721();
    erc721.mint(bob, 0);

    // emulate bob
    vm.startPrank(bob);

    // transfer to mary
    erc721.safeTransferFrom(bob, mary, 0);

    // check to make sure mary is the new owner
    address owner_of = erc721.ownerOf(0);
    assertEq(mary, owner_of);
}

// only the owner can burn
function testBurn() public {
    erc721 = new ERC721();
    erc721.mint(bob, 0);
    vm.startPrank(bob);
    erc721.burn(0);
}
```

### Fuzzing

模糊测试允许我们定义函数参数类型，框架会在运行时自动测试这些值。

例如，我们可以创建一个测试函数来接收函数参数，并在测试中使用该值，而无需定义它是什么:

合约如下:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.13;

contract HelloWorld {
  string private greeting;
  uint public version = 0;
  
  constructor (string memory _greeting) {
    greeting = _greeting;
  }

  function greet() public view returns(string memory) {
    return greeting;
  }
}
```

测试合约如下:

```solidity
contract ContractTest is Test {
  function testFuzzing(string memory _greeting) public {
      HelloWorld hello = new HelloWorld(_greeting);
      assertEq(
          hello.greet(),
          _greeting
      );
  }
}
```

### Gas

您可以轻松打印gas测试报告:

```sh
forge test --gas-report
```

### ABIs

在运行`forge build`脚本进行构建或脚本部署之后，ABI 将位于`out`目录中。

### Test Options



```sh
forge test --help
```

## 脚本和部署

Foundry 最近发布了[Solidity Scripting](https://twitter.com/gakonst/status/1531056739470016512) . 

[Scripting](https://book.getfoundry.sh/tutorials/solidity-scripting.html) 让你对如何使用Solidity脚本部署合约进行大量控制, 我相信它旨在取代以前部署合约的[`forge create`](https://book.getfoundry.sh/reference/forge/forge-create.html) 。

From [Foundry Book](https://book.getfoundry.sh/tutorials/solidity-scripting.html):



Scripts are executed by calling the function named `run`, our entrypoint:

```sh
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

import "forge-std/Script.sol";

import {Counter} from "src/Counter.sol";

contract ContractScript is Script {
    function setUp() public {}

    function run() public {
        vm.startBroadcast();
        new Counter(10);
        vm.stopBroadcast();
    }
}
```

现在，我们可以使用此脚本将我们的智能合约部署到实时或测试网络. 🚀

`startBroadcast` 和 `startBroadcast(address)` 将让所有后续调用（仅在此调用深度）创建可以稍后在链上签名和发送交易。

`stopBroadcast` 停止收集交易以供以后的链上广播。



## 本地部署

启动Anvil运行本地测试网:

```sh
anvil
```

启动后，Anvil 将为您提供一个本地 RPC 和一些您可以使用的私钥和帐户。

现在，我们可以使用本地 RPC 在本地部署:

```sh
forge script script/Contract.s.sol:ContractScript --fork-url http://localhost:8545 \
--private-key $PRIVATE_KEY --broadcast
```

### 使用 cast 执行以太坊 RPC 调用

在本地部署合约后，Anvil 将注销合约地址。

接下来，将合约地址设置为环境变量

```sh
export CONTRACT_ADDRESS=<contract-address>
```

然后，我们可以使用`cast send` 发送事务。

```sh
cast send $CONTRACT_ADDRESS "incrementCounter()" \
--private-key $PRIVATE_KEY
```

我们可以使用`cast call`执行读取操作:

```sh
cast call $CONTRACT_ADDRESS "getCount()(int)"
```

## 部署到网络

现在我们已经在本地部署和测试，我们可以部署到网络

运行以下脚本:

```sh
forge script script/Contract.s.sol:ContractScript --rpc-url $RPC_URL \
 --private-key $PRIVATE_KEY --broadcast
```

一旦合约部署到网络，我们可以使用`cast send`来发送事务:

```sh
cast send $CONTRACT_ADDRESS "incrementCounter()" --rpc-url $RPC_URL \
--private-key $PRIVATE_KEY 
```

我们可以使用`cast call`执行读取操作:

```sh
cast call $CONTRACT_ADDRESS "getCount()(int)" --rpc-url $RPC_URL
```

### Cast 帮助

可以通过运行``-- help`来获取命令的完整列表

```sh
cast --help
```