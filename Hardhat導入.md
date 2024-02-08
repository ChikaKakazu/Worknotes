# Hardhat導入手順
- docker利用
- DockerfileのimageはNode. DockerfileにHardhatのinstallが記述されている

## プロジェクト作成
- `npx hardhat init`コマンドを、Hardhatコンテナ内で入力する
  - いくつか質問が投げられるが、基本Enterもしくはyで問題ない
    ```
    Need to install the following packages:
    hardhat@2.19.0
    Ok to proceed? (y) y
    888    888                      888 888               888
    888    888                      888 888               888
    888    888                      888 888               888
    8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
    888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
    888    888 .d888888 888    888  888 888  888 .d888888 888
    888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
    888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888

    Welcome to Hardhat v2.19.0

    ...
    ```
- 以下のようなディレクトリが作成される
    ```
    > tree -L 2
    .
    ├── contracts
    │   └── Lock.sol
    ├── hardhat.config.js
    ├── node_modules
    ├── package-lock.json
    ├── package.json
    ├── scripts
    │   └── deploy.js
    └── test
        └── Lock.js
    ```
- 確認
  - `npx hardhat help`コマンドで色々表示される

## コンパイル・テスト・デプロイ・確認まで
### コントラクト作成
- `contracts/`配下にコントラクトファイルを作る
  ```
  // Greeter.sol

  // SPDX-License-Identifier: UNLICENSED
  pragma solidity ^0.8.9;

  import "hardhat/console.sol";

  contract Greeter {
      string private greeting;

      constructor(string memory _greeting) {
          console.log("Deploying a Greeter with greeting:", _greeting);
          greeting = _greeting;
      }

      function greet() public view returns (string memory) {
          return greeting;
      }
  }
  ```
- コンパイルする
  - `npx hardhat compile`をターミナルで入力
    ```
    > npx hardhat compile
    Downloading compiler 0.8.19
    Compiled 1 Solidity file successfully (evm target: paris).
    ```

### テストする
- `test/`配下に、.jsのテストファイルを作成する
  ```js
  const { expect } = require("chai");

  describe("Greeter", function () {
    it("Should return the new greeting once it's changed", async function () {
      // コントラクトのデプロイ
      const greeter = await hre.ethers.deployContract("Greeter", ["Hello, world!"]);
      await greeter.waitForDeployment();

      // 初期設定が期待通りか検証
      expect(await greeter.greet()).to.equal("Hello, world!");

      // 値を変更
      const setGreetingTx = await greeter.setGreeting("Hello, hardhat!");

      // トランザクションが通るまで待機
      await setGreetingTx.wait();

      // 値が変わっているか検証
      expect(await greeter.greet()).to.equal("Hello, hardhat!");
    });
  });
  ```
- テストする
  - `npx hardhat test`をターミナルで入力
    ```
    > npx hardhat test
      Greeter
    Deploying a Greeter with greeting: Hello, world!
    Changing greeting from 'Hello, world!' to 'Hello, hardhat!'
        ✔ Should return the new greeting once it's changed (5738ms)
    ```

### デプロイする
- `scripts/`配下に、.jsのテストファイルを作成する
    ```js
    const hre = require("hardhat");

    async function main() {
        // Hardhatは実行前にコンパイルを行ってくれます。
        // 明示的に行いたい場合は以下の用にしてください。
        await hre.run('compile');

        const greeter = await hre.ethers.deployContract("Greeter", ["Hello, Hardhat!"]);
        // コントラクトのデプロイ
        await greeter.waitForDeployment();

        // コントラクトのアドレスを表示
        console.log("Greeter deployed to:", await greeter.getAddress());
    }

    main().catch((error) => {
        console.error(error);
        process.exitCode = 1;
    });
    ```
- ブロックチェーンノードを起動する
  - `npx hardhat node`をターミナルで入力
    ```
    > npx hardhat node
    Started HTTP and WebSocket JSON-RPC server at http://0.0.0.0:8545/

    Accounts
    ========

    WARNING: These accounts, and their private keys, are publicly known.
    Any funds sent to them on Mainnet or any other live network WILL BE LOST.

    Account #0: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 (10000 ETH)
    Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

    Account #1: 0x70997970C51812dc3A010C7d01b50e0d17dc79C8 (10000 ETH)
    Private Key: 0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d

    Account #2: 0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC (10000 ETH)
    Private Key: 0x5de4111afa1a4b94908f83103eb1f1706367c2e68ca870fc3fb9a804cdab365a

    ...
    ```
- デプロイする
  - `npx hardhat run scripts/GreeterDeploy.js --network localhost`をターミナルで入力
    ```
    > npx hardhat run scripts/GreeterDeploy.js --network localhost
    Nothing to compile
    Greeter deployed to: 0x959922bE3CAee4b8Cd9a407cc3ac1C251C2007B1
    ```
  - デプロイしたコントラクトアドレスを控えておく
    - 起動したnodeにログで表示される
    ```
    Contract address:    0x959922be3caee4b8cd9a407cc3ac1c251c2007b1
    ```
### 確認する
- consoleに入ってデプロイしたコントラクトを呼び出す
  - `npx hardhat console --network localhost`をターミナルで入力
    ```
    > npx hardhat console --network localhost
    Welcome to Node.js v20.7.0.
    Type ".help" for more information.
    >
    ```
- デプロイしたコントラクトの関数を呼ぶ
  ```
  > const Greeter = await ethers.getContractAt('Greeter', 'コントラクトアドレス');
  undefined
  > await Greeter.greet();
  'Hello, Hardhat!'
  ```