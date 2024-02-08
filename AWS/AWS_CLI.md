# AWS CLI
AWS CLIについて調べる。
導入からS3の操作についてまとめたい

## インストール
- 参考
  - [AWS CLI のインストールと更新の手順](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html)
  - 確認
    ```
    > which aws
    /usr/local/bin/aws
    > aws --version
    aws-cli/2.9.21 Python/3.9.11 Darwin/21.6.0 exe/x86_64 prompt/off
    ```


## AWS CLIを使用する

### 1. IAMユーザーを作成する
- 参考
  - [AWS CLI バージョン 2 を使用するための前提条件](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-prereqs.html)
  - [普段使いのIAMユーザーを作る - AWSをはじめからていねいに](https://zenn.dev/sway/articles/aws_biginner_create_iam_user)

1. AWSコンソール画面からIAMユーザーを作成する
   1. IAMサービスの画面から、ユーザー画面に移動する
   2. 画面右上の「ユーザーを追加」を選択
   3. ユーザーの作成
      1. ユーザーの詳細を指定
         1. ユーザー名：kakazu_S3
         2. コンソールアクセスを有効化 - オプション：チェック入れる
            1. コンソールパスワード
               1. 自動生成されたパスワード：チェック入れる
               2. ユーザーは次回のサインイン時に新しいパスワードを作成する必要があります (推奨)：チェック外す　(自動生成したパスワードをそのまま使いたいため)
         ![picture 1](images/823cba7b84f20d45f550f64b85819376784c33ac80b844c3fe7d71eceeb5810f.png)
      2. 許可を設定
         1. 許可のオプション：ポリシーを直接アタッチする (S3にしかアクセスしないため。他で使い回す予定もないため)
         2. 許可ポリシー：AmazonS3ReadOnlyAccess (S3の読み取りを許可する)
         ![picture 2](images/3d51d82abe5fd939896b91665725173bf48eec1707f8fccdf8237bc25ea71872.png)

      3. 確認して作成
         1. 最終的な設定の確認
         ![picture 3](images/e0ac40f6bcec709345deb03c9abed1e12aac7762458872bb8d560d1860d92a38.png)

      4. パスワードを取得
         1. コンソールにログインするためのパスワードを取得する
         ![picture 4](images/5bdd09081f66fa8891732b21c59f9d44e13997df0088fefeb3c3927a3f298d39.png)


2. アクセスキーを作成する
   1. 作成したIAMユーザーから「セキュリティ認証情報」-> 「アクセスキー」-> 「アクセスキーを作成」を選択
   2. アクセスキーを作成
      1. 主要なベストプラクティスと代替案にアクセスする：コマンドラインインターフェース(CLI)
      2. 説明タグを設定 - オプション
         1. 特に何も行わず「アクセスキーを作成」に進む
         ![picture 5](images/1f7e9f99603b973ab1ef81e40e1fbae88a73ae2b373e8641820489292aee5338.png)

      3. アクセスキーとシークレットアクセスキーを取得する
         ![picture 6](images/5a4cf0540673847fb6196538e4a9c0ced8e77e3c15ca4f39efbce934caad730a.png)

### 2. AWS CLIの設定をする
- 参考
  - [設定の基本](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-configure-quickstart.html)
  - [AWS CLIでS3を操作する - AWSをはじめからていねいに](https://zenn.dev/sway/articles/aws_biginner_use_cli)

1. aws configureを使用して設定する
   1. `aws configure`をターミナルで打ち、必要な情報を入力する
        ```
        > aws configure
        AWS Access Key ID [None]: アクセスキー作った時のもの
        AWS Secret Access Key [None]: アクセスキー作った時のもの
        Default region name [None]: ap-northeast-1
        Default output format [None]: json
        ```
   2. AWS S3に接続できるかの確認
        ```
        # バケットの一覧
        > aws s3 ls
        2021-04-19 16:21:24 invoice-report
        2021-04-22 16:24:16 laravel-logs
        ...
        ```
