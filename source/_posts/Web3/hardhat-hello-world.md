---
title: 使用 Hardhat3 创建简单合约并部署到测试网
description: 使用 Hardhat3 创建简单合约并部署到测试网, 并与合约进行交互
date: 2024-07-11 14:11:43
categories:
  - Web3
tags:
  - Web3
  - Contract
  - Hardhat
  - Viem
  - Sepolia
---

# 使用 Hardhat3 创建简单合约并部署到测试网

使用 Hardhat3 创建简单合约并部署到测试网

## 创建 Hardhat3 项目

选择文件目录输入如下命令:

```sh
mkdir HelloWorld
cd HelloWorld
pnpm init
pnpm dlx hardhat --init
```

选择项均使用默认选项

## 编写 HelloWorld 合约

创建一个基本的 HelloWorld 合约

```solidity
pragma solidity ^0.8.28;

contract HelloWorld {
  event messageChanged(string oldEvent, string newEvent);

  string public message;

  constructor(string memory firstMessage) {
    message = firstMessage;
  }

  function update(string memory newMessage) public {
    string memory oldMessage = message;
    message = newMessage;

    emit messageChanged(oldMessage, newMessage);
  }
}
```

## 编译 solidity 合约代码

编写 hardhat 配置文件 ./hardhat.config.ts:

```ts
import type { HardhatUserConfig } from "hardhat/config";

import hardhatToolboxViemPlugin from "@nomicfoundation/hardhat-toolbox-viem";
import { configVariable } from "hardhat/config";

const config: HardhatUserConfig = {
  plugins: [hardhatToolboxViemPlugin],
  solidity: {
    profiles: {
      default: {
        version: "0.8.28",
      },
      production: {
        version: "0.8.28",
        settings: {
          optimizer: {
            enabled: true,
            runs: 200,
          },
        },
      },
    },
  },
  networks: {
    sepolia: {
      type: "http",
      chainType: "l1",
      url: configVariable("SEPOLIA_RPC_URL"),
      accounts: [configVariable("SEPOLIA_PRIVATE_KEY")],
    },
  },
};

export default config;
```

hardhat 会在部署时根据配置文件对合约代码进行编译

## 部署到 Sepolia

在 [Alchemy](https://dashboard.alchemy.com/) 上申请 app 并选择 sepolia 网络, 获取 dapp 的 rpcurl

编写部署点火器文件 ./ignition/modules/HelloWorld.ts

```ts
import { buildModule } from "@nomicfoundation/hardhat-ignition/modules";

export default buildModule("HelloWorldModule", (m) => {
  // 第一个参数初始化合约, 第二个参数向构造函数传递参数
  const helloWorld = m.contract("HelloWorld", ["Hello, world!"]);

  return { helloWorld };
});
```

使用 hardhat 提供的 keystore 为部署工具提供 sepolia rpc url 和你的账户私钥的环境变量

```sh
pnpm hardhat keystore set SEPOLIA_RPC_URL
pnpm hardhat keystore set SEPOLIA_PRIVATE_KEY
```

执行部署命令

```sh
pnpm hardhat ignition deploy ignition/modules/HelloWorld.ts
```

## 调用智能合约

在 ./scripts/helloWorld.ts 中编写调用合约的逻辑

```ts
import { network } from "hardhat";

// 连接到Sepolia
const { viem } = await network.connect({
  network: "sepolia",
  chainType: "l1",
});

// 获取公共客户端
const publicClient = await viem.getPublicClient();

// 根据地址获取合约
const contract = await viem.getContractAt(
  "HelloWorld",
  "YOUR_CONTRACT_ADDRESS",
  {
    client: {
      public: publicClient,
    },
  }
);

console.log(contract);

const message1 = await contract.read.message();

console.log(message1);

await contract.read.update(["Hello, sepolia!"]);

const message2 = await contract.read.message();

console.log(message2);
```

执行命令调用合约

```sh
pnpm hardhat run ./scripts/helloWorld.ts
```
