# AWS_RDS構築
ここではAmazon Auroraを利用する

### 参考
- [DB クラスターを作成して Aurora MySQL DB クラスターのデータベースに接続する](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/AuroraUserGuide/CHAP_GettingStartedAurora.CreatingConnecting.Aurora.html)

## RDSの構築
1. AWSコンソール画面右上でリージョンを確認する
1. AWSコンソール画面でAmazon RDS->データベースに移動する
1. 画面右の「データベースの作成」をクリック
1. データベースの作成
    1. データベース作成方法を選択
        - 標準作成
        - 簡単作成
            - 作成後に細かく修正できる
    1. エンジンのオプション
        - エンジンタイプ
            - Amazon Auroraを選択
        - エディション
            - どちらの互換性を持つか
            - Amazon Aurora MySQL 互換エディションを選択
        - レプリケーション機能
            - 参考 : [【AWS】RDSのレプリケーションについて解説します。](https://www.acrovision.jp/service/aws/?p=2462)
            - シングルマスター
                - 一つの書き込みDBと最大15の読み込みDBでクラスターを作成する。
            - マルチマスター
                - 全てのDBに書き込みが可能になる。
                - 一つのリージョンないで複数のAZをまたがって構成できる。
        - エンジンバージョン
            - MySQLのバージョンを選択する
            - 使用できるインスタンスやサーバーレスが選択できるかが変わってくる
    1. テンプレート
        - 後々の設定のデフォルト値がそれぞれに適した値になっている
        - 本番稼働用
        - 開発/テスト
    1. 設定
        - DBクラスター識別子
            - DBクラスターの名前
        - 認証情報の設定
            1. マスターユーザー名
                - DBにアクセスするログインID
                - ※ デフォルトのadminでもいいかも
            1. パスワードの自動設定
                - マスターパスワードを自動生成できる
            1. マスターパスワード・パスワードを確認
                - パスワードを自分で設定する
    1. インスタンスの設定
        - DBインスタンスクラス
            - サーバーレス(v1, v2の2種類あるが、それぞれ特性が違う)
            - メモリ最適化クラス
            - バースト可能クラス
            - ミニマムなものや、開発環境などではバースト可能クラスの方が適している
            - 参考 : [ちょっと待ってください!あなたが使うべきは本当にT系インスタンスですか!?](https://dev.classmethod.jp/articles/ec2-t-or-m/)<br>
            [Amazon RDS インスタンスタイプ](https://aws.amazon.com/jp/rds/instance-types/)<br>
            [Aurora Serverlessの導入時に気をつけるべきこと](https://dev.classmethod.jp/articles/lessons-learned-from-up-and-running-aurora-serverless/)<br>
            [待たせたな！噂のAuroraサーバーレスv2がGA。初心者にも分かりやすくまとめてみた](https://qiita.com/minorun365/items/2a548f6138b6869de51a)<br>
            [Amazon Aurora Serverless](https://aws.amazon.com/jp/rds/aurora/serverless/)

    1. 可用性と耐久性
        - 同じリージョン内でもアベイラビリティゾーン(AZ)を分けることによって障害に強くなる。
        - 開発環境などではAZを分けなくてもいいかも
        - マルチAZ配置
            - 別のAZでAuroraレプリカ/リーダーノードを作成する(可用性のスケーリングに推奨)
            - Auroraレプリカを作成しない
    1. 接続
        - Virtual Private Cloud(VPC)
            - 予め用意したVPCを設定する
            - ※ 作成後にVPCの変更はできない
        - サブネットグループ
            - 選択したvpcで使用できるサブネットグループが選択できる
            - なければ新規で作成する必要がある
        - パブリックアクセス
        - vpcセキュリティグループ
            - 既存の選択
            - 新規作成
        - アベイラビリティーゾーン
        - データベースポート
            - デフォルトでいいかも
    1. データベース認証
        - パスワード認証
        - パスワードとIAMデータベース認証
    1. 追加設定
        1. データベースの選択肢
            1. 最初のデータベース名
                - RDS内で作成されるDBの名前
            1. フェイルオーバー優先順位
                - 障害発生時にレプリカへ切り替える際の優先順位
                - 参考 : [RDS for Auroraでフェイルオーバー先の優先順序を指定できます](https://dev.classmethod.jp/articles/amazon-aurora-failover-order/)
        1. Performance Insights
            - Performance Insights を有効にする
                - DBパフォーマンスモニタリング機能
            - 保持期間
                - Performance Insights が保存して使用可能にするデータ履歴
                - デフォルトの7日の場合は無料。それ以降は有料になる
                - 参考 : [Performance Insights の料金](https://aws.amazon.com/jp/rds/performance-insights/pricing/)
        1. モニタリング
            - 拡張モニタリングの有効化
                - さまざまなプロセスやスレッドで CPU がどのように使用されているのかを確認できる
                - 参考 : [拡張モニタリングを使用した OS メトリクスのモニタリング](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/USER_Monitoring.OS.html)
        1. ログのエクスポート
            - Amazon CloudWatch Logs に発行するログタイプを選択する
            - デフォルトで全てoff
            - Cloud Watch Logsについて別途調べる必要あり


## サブネットの作成
RDSを構築する際に使用するサブネットグループのために必要なサブネットを作成する
1. VPCコンソール画面左からサブネットを選択
1. 画面右上の「サブネットを作成」を選択
1. サブネットを作成
    1. VPC
        - VPC ID
            - サブネットを作成する場所(VPC)を選択する
            - その中でしか使用できない
    1. サブネットの設定
        - サブネット名
        - アベイラビリティーゾーン
            - 複数のサブネットを作成する場合は、AZは異なるものを選択する
        - IPv4 CIDR ブロック
            - IPアドレスを指定して、使用可能なIPアドレスを制限する
            - 複数のサブネットを作成する場合は、重複しないように注意する
            - 例
                ```
                // ap-northeast-1a
                > 10.0.0.0/24
                // ap-northeast-1c
                > 10.0.1.0/24
                ```

## サブネットグループの作成
RDSを構築する際に使用するサブネットグループを作成する
1. RDSのコンソール画面左にあるサブネットグループを選択
1. 画面右上の「DBサブネットグループを作成」を選択
1. DBサブネットグループを作成
    1. サブネットグループの詳細
        - 名前
        - 説明
        - VPC
            - 使用するサブネットに対応するVPCを選択する
    1. サブネットを追加
        - アベイラビリティーゾーン
            - 追加するサブネットを含むAZを選択する
        - サブネット
            - 選択したAZに含まれるサブネットを選択する
            - ***※ 最低でも異なるAZのサブネット2つ以上を選択する必要がある***

## 接続確認
ec2 -> RDSへの疎通確認
1. 同じvpc内であることを確認する
1. 同じセキュリティグループが使われていることを確認する
1. インバウンドルールの編集で、RDSに対してec2のPrivateIPアドレスを許可する
    - private Ipアドレスを設定する
    - public Ipアドレスだと疎通できない
1. ec2にssh接続 -> 疎通コマンド
    ```
    nc -zv RDSエンドポイント 3306
    ```
- ※ ncコマンドがAmazon Linux2に入っていないので導入する
    - ```sudo yum search netcat```で対応するncコマンドのプラグインを検索
    - ```sudo yum info nmap-ncat.x86_64```で検索に引っかかったやつの情報を見て、信用できるものか確認する
    - ```sudo yum install nmap-ncat.x86_64```でインストール
- GUIで確認するならMySQL Workbenchで接続して確認する

### サブネット
- ネットワーク内に設定する小さなネットワーク。
- awsだとVPC内に複数のサブネットを設定する
### サブネットマスク
- ネットワークの範囲を指定する
[サブネットとは？](https://www.cloudflare.com/ja-jp/learning/network-layer/what-is-a-subnet/)
[サブネットマスク計算（IPv4)](https://note.cman.jp/network/subnetmask.cgi)
[やってみようシリーズ：仮想ネットワークにインターネットゲートウェイを作ってみた](https://www.idaten.ne.jp/portal/page/out/secolumn/multicloud/column013.html)
[IPアドレスサブネットマスク早見表](https://www.ahref.org/doc/ipsubnet.html)
[VPC Subnet（サブネット）](https://dev.classmethod.jp/articles/re-introduction-2022-vpc/)

