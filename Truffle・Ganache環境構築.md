# TruffleとGanacheでローカルにスマートコントラクト開発環境の構築
- [Truffle](https://trufflesuite.com/docs/truffle/)
- [Ganache](https://trufflesuite.com/docs/ganache/)

## 前提
- Truffleを導入するDocker環境が用意されている

## Ganache
1. [Ganacheダウンロード](https://trufflesuite.com/ganache/)からダウンロードする
2. `NEW WORKSPACE`をクリックして起動する
3. 設定画面でワークスペース名を入力
4. `TRUFFLE PROJECT`にはTruffleプロジェクトの`truffle-config.js`ファイルを後ほど設定する
5. SERVERタブに移動し、`PORT NUMBER`を`7545`から`8545`に変更する
   1. 変更しなくてもいいが、その場合、Truffle側のポートをこちらに合わせる必要がある
6. 設定を保存する
7. 10個の100ETHを持つアカウントが作られる
- Truffle側でスマートコントラクトの開発を行う際は、Truffleと接続したGanacheを起動しておく必要がある

## Truffle
- Truffleの導入は、Dockerfileに記述されているものとする。
1. Truffleプロジェクトの作成
   1. Truffleがインストールされているか確認する
      1. ```
         > truffle version
         Truffle v5.11.5 (core: 5.11.5)
         Ganache v7.9.1
         Solidity v0.5.16 (solc-js)
         Node v18.18.0
         Web3.js v1.10.0
         ```
   2. 作業ディレクトリで```truffle init```コマンドを入力
      1.    ```
            > truffle init
            Starting init...
            ================

            > Copying project files to /usr/src/app/truffle

            Init successful, sweet!

            Try our scaffold commands to get started:
            $ truffle create contract YourContractName # scaffold a contract
            $ truffle create test YourTestName         # scaffold a test

            http://trufflesuite.com/docs
            ```
      2. contract, migrations, testの3つのディレクトリが作成される
      3. ```truffle-config.js```ファイルが作成される
2. ```truffle-config.js```をGanacheと接続できるように変更する
   1. コメントアウトされている以下のコメントアウトを解除する
      1.    ```
            // development: {
            //  host: "127.0.0.1",     // Localhost (default: none)
            //  port: 8545,            // Standard Ethereum port (default: none)
            //  network_id: "*",       // Any network (default: none)
            // },
            ↓↓
            development: {
                host: "127.0.0.1",     // Localhost (default: none)
                port: 8545,            // Standard Ethereum port (default: none)
                network_id: "*",       // Any network (default: none)
            },
            ```
   2. hostを`host.docker.internal`に変更する。変更することで、Ganacheでたてたプライベートネットワークに接続できる
3. Ganacheとの接続確認
   1. `truffle console`コマンドでGanacheと接続。エラーが出なければ接続完了。
      1.    ```
            # 対話モードになる
            > truffle console
            truffle(development)> 
            ```
   2. Ganache側で作られたアカウントを表示する
      1.    ```
            # Ganacheのアカウントと目視で比較。同じになっているはず
            truffle(development)> web3.eth.getAccounts()
            [
                '0x472fa6f6F25Ac28DcAe0b4E075342d3148e2101D',
                '0x0D670062De92a1dCa69292DF3FEB42c2309e9028',
                '0xf11FcB3e5eb085662bbc9D29c6E7323343A74763',
                '0x969760A4bE82367fb3b0b992ffF40DfE168cC7c6',
                '0x92b6c59d33d1B97b6A4009C611Ab86D3Cc416B2B',
                '0x2450d8cACEfA2819210EF4981D846A92b81BBc19',
                '0x28390535B8687F0189B2bC3472718B6e912c7850',
                '0x2EC2f34895aFBc1FB9ecf29355d9C59b125Ba6c6',
                '0x1cdFC00A0eA648Cca5543c3c5B7519470Cb1Ba1e',
                '0xaE06F2e77b1CB5cD79feDE72734C5f5d16836043'
            ]
            ```
4. ```truffle-config.js```でsolidityのコンパイラバージョンを変更する(2023年10月10日時点)
   1. `version: "0.8.21"` => `version: "0.8.19"`
   2. このバージョンでないとコントラクトをコンパイルはできるが、デプロイできない。理由は不明。
   3. コントラクトを記述する際のsolidityバージョンも以下にする必要がある
      1. `pragma solidity ^0.8.19;`

## スマートコントラクト開発 (Hello World)
1. solidityファイルとそれをデプロイするmigrationファイル、テストファイルをコマンドで作成する
   1.   ```
        # それぞれ一つずつ作成できる
        > truffle create contract ファイル名
        > truffle create migrate ファイル名
        > truffle create test ファイル名
        # まとめて作成する
        > truffle create all ファイル名
        ```
   2.   ```
        > truffle create all HelloWorld
        > tree .
        ├── contracts
        │   └── HelloWorld.sol
        ├── migrations
        │   └── 1696930341_hello_world.js
        ├── test
        │   └── hello_world.js
        ```
2. solidityファイルを修正する
   1. solidityバージョンを変更する
      1. `pragma solidity >=0.4.22 <0.9.0;` => `pragma solidity ^0.8.19;`
   2. HelloWorld記述
      1.    ```
            function helloworld() public pure returns (string memory) {
                return "Hello World!";
            }
            ```
3. コンパイルする
   1.   ```
        > truffle compile
        Compiling your contracts...
        ===========================
        ✓ Fetching solc version list from solc-bin. Attempt #1
        ✓ Downloading compiler. Attempt #1.
        > Compiling ./contracts/HelloWorld.sol
        > Compilation warnings encountered:

        > Artifacts written to /usr/src/app/truffle/build/contracts
        > Compiled successfully using:
        - solc: 0.8.19+commit.7dd6d404.Emscripten.clang
        ```
   2. コンパイルすると新しく`build/contracts`ディレクトリが作成される
      1.    ```
            > tree .
            ├── build
            │   └── contracts
            │       └── HelloWorld.json
            ```
4. migrationファイルを修正してデプロイする
   1.   ```
        const HelloWorld = artifacts.require("HelloWorld");
        module.exports = function (_deployer) {
            _deployer.deploy(HelloWorld);
        };
        ```
5. デプロイする
   1.   ```
        > truffle deploy
        Compiling your contracts...
        ===========================
        > Everything is up to date, there is nothing to compile.


        Starting migrations...
        ======================
        > Network name:    'development'
        > Network id:      5777
        > Block gas limit: 6721975 (0x6691b7)


        1696930341_hello_world.js
        =========================

        Deploying 'HelloWorld'
        ----------------------
        > transaction hash:    0x80769512116d10af7306259da79627b078e4f073ac3c14355929b358dbd8402b
        > Blocks: 0            Seconds: 0
        > contract address:    0xC787807cb4DA11c2D541525d5Ce121Dc4FF4CAFa
        > block number:        1
        > block timestamp:     1696930882
        > account:             0x472fa6f6F25Ac28DcAe0b4E075342d3148e2101D
        > balance:             99.999550669375
        > gas used:            133135 (0x2080f)
        > gas price:           3.375 gwei
        > value sent:          0 ETH
        > total cost:          0.000449330625 ETH

        > Saving artifacts
        -------------------------------------
        > Total cost:      0.000449330625 ETH

        Summary
        =======
        > Total deployments:   1
        > Final cost:          0.000449330625 ETH
        ```
6. Ganacheでブロックが積み上がっていることを確認する
7. テストする
   1.   ```
        > truffle test
        Using network 'development'.
        Compiling your contracts...
        ===========================
        > Everything is up to date, there is nothing to compile.

        Contract: HelloWorld
            ✔ should assert true

        1 passing (51ms)
        ```
   2. テストでもブロックは積み上がるみたい
- コンパイルとデプロイをまとめて行うこともできる
  - `truffle migration`