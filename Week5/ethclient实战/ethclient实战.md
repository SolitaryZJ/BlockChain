# 以太坊 ethclient 包使用引导

# 环境准备

## Go 环境准备

Golang 官网下载页面：https://go.dev/dl/

下载好 golang 的压缩包之后解压:

```shell
mkdir $HOME/go1.22.6 && tar -xf go1.22.6.darwin-arm64.tar.gz --strip 1 -C $HOME/go1.22.6
```

设置 Golang 所需的环境变量:

```shell
export GOROOT=$HOME/go1.22.6
export GOPATH=$HOME/.go_path
export PATH=$GOROOT/bin:$GOPATH/bin:$PATH
```

检查 go 语言环境:

```shell
go version
```

设置 GOPROXY:

```shell
go env -w GOPROXY="https://goproxy.cn,direct"
```

## 初始化 Go 项目

```shell
mkdir go-eth-demo && cd go-eth-demo && go mod init github.com/local/go-eth-demo
```

## 添加 ethclient 包依赖

```shell
go get github.com/ethereum/go-ethereum
go get github.com/ethereum/go-ethereum/rpc
```

## 使用 nodejs，安装 solc 工具

```shell
npm install -g solc
```

## 安装 abigen 工具

```shell
go install github.com/ethereum/go-ethereum/cmd/abigen@latest
```

# KeyStore 文件转私钥

以太坊提供了 ethkey 工具把 keystore 文件中包含的私钥信息转换成可以导入到钱包中的私钥。

从源码编译 ethkey 工具:

```shell
go run build/ci.go install ./cmd/ethkey
```

使用 ethkey 把 keystore 文件转换成私钥并打印出来，需要把生成 keystore 文件时输入的密码写入到 pw.txt 中，并通过 passwordfile 参数传给 etheky，：

```shell
./ethkey inspect --private --passwordfile pw.txt --json keyfile.json
```

具体实现代码：

```shell
// Read key from file.
keyjson, err := os.ReadFile("<keystore文件路径>")
if err != nil {
    utils.Fatalf("Failed to read the keyfile at '%s': %v", keyfilepath, err)
}

// Decrypt key with passphrase.
passphrase := "<keyStore密码>"
key, err := keystore.DecryptKey(keyjson, passphrase)
if err != nil {
    utils.Fatalf("Error decrypting key: %v", err)
}

address := key.Address.Hex()
privateKey := hex.EncodeToString(crypto.FromECDSA(key.PrivateKey))

fmt.Println(fmt.Sprintf("address: %s,\nprivateKye: %s", address, privateKey))
```

# 实操前准备

## 测试网环境获取

测试网环境服务平台
[https://www.alchemy.com](https://www.alchemy.com)

[https://access.rockx.com/](https://access.rockx.com/)

[https://www.quicknode.com/](https://www.quicknode.com/) (信用卡)

[https://www.infura.io/](https://www.infura.io/)

[https://ethereum.publicnode.com/?sepolia](https://ethereum.publicnode.com/?sepolia) (无需注册，可以直接使用，但偶尔会抽风)

## 测试网代币获取

需要关联 Alchemy 账号

[https://www.alchemy.com/faucets/ethereum-sepolia](https://www.alchemy.com/faucets/ethereum-sepolia)

需要关联 RockX 账号

[https://access.rockx.com/faucet-sepolia](https://access.rockx.com/faucet-sepolia)

需要关联 Quicknode 账号

[https://faucet.quicknode.com/ethereum/sepolia](https://faucet.quicknode.com/ethereum/sepolia)

需要关联 Infura 账号

[https://www.infura.io/faucet/sepolia](https://www.infura.io/faucet/sepolia)

不需要注册账号

[https://console.optimism.io/faucet](https://console.optimism.io/faucet)

这个不需要关联账号 但是需要自己挖

[https://www.ethereum-ecosystem.com/faucets/ethereum-sepolia](https://www.ethereum-ecosystem.com/faucets/ethereum-sepolia)

## devnet 环境

### 使用 hardhat 启动 devnet

安装 node 版本管理器，从 n 和 nvm 选择一个安装即可

- n

```bash
curl -fsSL https://raw.githubusercontent.com/tj/n/master/bin/n | bash -s lts
```

- nvm

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
```

在 node 项目下，安装 hardhat

```bash
npm install --save-dev hardhat
```

使用 hardhat 初始化项目

```bash
npx hardhat init
```

进入 hardhat 项目目录后，执行命令

```bash
npx hardhat node
```

节点启动后，将会在控制台打印测试预分配了测试 ETH 的地址以及私钥。

这些私钥是固定的，并且是公开的，所以不要使用这些私钥在主网上做任何转账操作。

### 使用 ethpandaops 启动 devnet

仓库地址
[https://github.com/ethpandaops/ethereum-package](https://github.com/ethpandaops/ethereum-package)

需要先安装 kurtosis 工具([https://docs.kurtosis.com/install/](https://docs.kurtosis.com/install/))，然后在控制台执行命令，将会自动下载镜像，以及初始化一个以太坊本地的 devnet 环境：

```bash
kurtosis run --enclave my-testnet github.com/ethpandaops/ethereum-package
```

测试使用的密钥的助记词:

```bash
giant issue aisle success illegal bike spike question tent bar rely arctic volcano long crawl hungry vocal artwork sniff fantasy very lucky have athlete
```

# 查询操作演示

## 查询区块

[查询区块](https://github.com/Base1-Go/Golang/blob/main/06.ethclient%E5%AE%9E%E6%88%98/2.01%20%E6%9F%A5%E8%AF%A2%E5%8C%BA%E5%9D%97/%E6%9F%A5%E8%AF%A2%E5%8C%BA%E5%9D%97.md)

## 查询交易与 Receipt

[查询交易](https://github.com/Base1-Go/Golang/blob/main/06.ethclient%E5%AE%9E%E6%88%98/2.02%20%E6%9F%A5%E8%AF%A2%E4%BA%A4%E6%98%93/%E6%9F%A5%E8%AF%A2%E4%BA%A4%E6%98%93.md)



# 发起交易演示

## 转账交易

[转账以太币 ETH](https://github.com/Base1-Go/Golang/blob/main/06.ethclient%E5%AE%9E%E6%88%98/2.05%20ETH%E8%BD%AC%E8%B4%A6/ETH%E8%BD%AC%E8%B4%A6.md)

## 部署合约交易

[部署智能合约](https://github.com/Base1-Go/Golang/blob/main/06.ethclient%E5%AE%9E%E6%88%98/2.10%20%E9%83%A8%E7%BD%B2%E5%90%88%E7%BA%A6/%E9%83%A8%E7%BD%B2%E5%90%88%E7%BA%A6.md)

## 调用合约

[写入智能合约](https://github.com/Base1-Go/Golang/blob/main/06.ethclient%E5%AE%9E%E6%88%98/2.11%20%E5%8A%A0%E8%BD%BD%E5%90%88%E7%BA%A6/%E5%8A%A0%E8%BD%BD%E5%90%88%E7%BA%A6.md)

# 订阅演示

## 订阅最新区块

[订阅新区块](https://github.com/Base1-Go/Golang/blob/main/06.ethclient%E5%AE%9E%E6%88%98/2.09%20%E8%AE%A2%E9%98%85%E5%8C%BA%E5%9D%97/%E8%AE%A2%E9%98%85%E5%8C%BA%E5%9D%97.md)

## 订阅事件

[订阅事件日志](https://github.com/Base1-Go/Golang/blob/main/06.ethclient%E5%AE%9E%E6%88%98/2.13%20%E5%90%88%E7%BA%A6%E4%BA%8B%E4%BB%B6/%E5%90%88%E7%BA%A6%E4%BA%8B%E4%BB%B6.md)

## 源码
