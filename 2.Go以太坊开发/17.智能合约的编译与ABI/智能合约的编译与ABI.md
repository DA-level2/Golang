# 转换文档

与智能合约交互，我们要先生成相应智能合约的应用二进制接口 ABI(application binary interface)，并把 ABI 编译成我们可以在 Go 应用中调用的格式。

第一步是安装 <u>Solidity 编译器</u> (`solc`).

```
npm install -g solc
```

我们还得安装一个叫 `abigen` 的工具，来从 solidity 智能合约生成 ABI。

假设您已经在计算机上设置了 Go，只需运行以下命令即可安装 `abigen` 工具。

```
go get -u github.com/ethereum/go-ethereum
cd $GOPATH/src/github.com/ethereum/go-ethereum/
make
make devtools
```

这里只是一个简单的合约，就是一个键/值存储，只有一个外部方法来设置任何人的键/值对。 我们还在设置值后添加了要发出的事件。

```
pragma solidity ^0.8.26;

contract Store {
  event ItemSet(bytes32 key, bytes32 value);

  string public version;
  mapping (bytes32 => bytes32) public items;

  constructor(string memory _version) {
    version = _version;
  }

  function setItem(bytes32 key, bytes32 value) external {
    items[key] = value;
    emit ItemSet(key, value);
  }
}
```

虽然这个智能合约很简单，但它将适用于这个例子。

现在我们可以从一个 solidity 文件生成 ABI。

```
solcjs --abi Store.sol
```

它会将其写入名为“Store_sol_Store.abi”的文件中

现在让我们用 `abigen` 将 ABI 转换为我们可以导入的 Go 文件。 这个新文件将包含我们可以用来与 Go 应用程序中的智能合约进行交互的所有可用方法。

```
abigen --abi=Store_sol_Store.abi --pkg=store --out=Store.go
```

为了从 Go 部署智能合约，我们还需要将 solidity 智能合约编译为 EVM 字节码。 EVM 字节码将在事务的数据字段中发送。 在 Go 文件上生成部署方法需要 bin 文件。

```
solcjs --bin Store.sol
```

现在我们编译 Go 合约文件，其中包括 deploy 方法，因为我们包含了 bin 文件。

```
abigen --bin=Store_sol_Store.bin --abi=Store_sol_Store.abi --pkg=store --out=Store.go
```

在接下来的课程中，我们将学习如何部署智能合约，然后与之交互。

### **完整代码**

Commands

```
go get -u github.com/ethereum/go-ethereum
cd $GOPATH/src/github.com/ethereum/go-ethereum/
make
make devtools

solc --abi Store.sol
solc --bin Store.sol
abigen --bin=Store_sol_Store.bin --abi=Store_sol_Store.abi --pkg=store --out=Store.go
```

Store.sol

```
pragma solidity ^0.8.26;

contract Store {
  event ItemSet(bytes32 key, bytes32 value);

  string public version;
  mapping (bytes32 => bytes32) public items;

  constructor(string memory _version) {
    version = _version;
  }

  function setItem(bytes32 key, bytes32 value) external {
    items[key] = value;
    emit ItemSet(key, value);
  }
}
```

solc version used for these examples

```
$ solcjs --version
0.8.26+commit.8a97fa7a.Emscripten.clang
```
