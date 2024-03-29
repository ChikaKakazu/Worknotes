# AWS_RDS構築の作業ログ

## RDSの構築
1. AWSコンソール画面右上でリージョンを確認する
1. AWSコンソール画面でAmazon RDS->データベースに移動する
1. 画面右の「データベースの作成」をクリック
1. データベースの作成
    1. データベース作成方法を選択
        - ```標準作成```
    1. エンジンのオプション
        1. エンジンタイプ
            - ```Amazon Aurora```
        1. エディション
            - ```Amazon Aurora MySQL互換エディション```
        1. レプリケーション機能
            - ```シングルマスター```
        1. エンジンバージョン
            - 全てoff
        1. 利用可能なバージョン
            - ```Aurora MySQL3.02.0(compatible with MySQL8.0.23)```
            - -> 作り直し(サーバーレスv1が使えるMySqlバージョン)
    1. テンプレート
        - ```開発/テスト```
    1. 設定
        1. DBクラスター識別子
            - ```kakazu-study-sre```
        1. マスターユーザー名
            - ```admin```
        1. パスワード
            - ```パスワード自動生成```
            - 作成後に画面上に表示される作成完了表示から詳細画面を開き、パスワードを確認する。
            - それを逃すと確認できないので、その場合、変更からパスワードを変更する
    1. インスタンスの設定
        - ```サーバーレス```
        - 最小容量 ```2ACU(4GIB)```
        - 最大容量 ```4ACU(8GIB)```
    1. 可用性と耐久性
        - ```Auroraレプリカを作成しない```
    1. 接続
        1. Virtual Private Cloud(VPC)
            - ```chura-sre-vpc```
        1. サブネットグループ
            - ```kakazu_study_sre```
        1. パブリックアクセス
            - ```あり```
        1. vpcセキュリティグループ
            - ```既存の選択```
            - ```studying_sre_kakazu_sg_new```
        1. アベイラビリティーゾーン
            - ```ap-northeast-1a```
        1. データベースポート
            - ```3306```
    1. データベース認証
        - ```パスワード認証```
    1. 追加設定
        1. データベースの選択肢
            1. 最初のデータベース名
                - ```lumen_project```
            1. DB クラスターのパラメータグループ
            1. DB パラメータグループ
                - ```default-aurora-mysql8.0```
            1. フェイルオーバー優先順位
                - ```指定なし```
        1. バックアップ
            1. バックアップ保存期間
                - ```1日間```
            1. スナップショットにタグをコピー
                - ```on```
        1. 暗号化
            1. 暗号化を有効化
                - ```off```
            1. AWS KMSキー
                - ```(default)aws/rds```
        1. モニタリング
            1. 拡張モニタリングの有効化
                - ```on```
        1. ログのエクスポート
            - ```全てoff```
        1. メンテナンス
            1. マイナーバージョン自動アップグレードの有効化
                - ```on```
            1. メンテナンスウィンドウ
                - ```設定なし```
        1. 削除保護
            1. 削除保護の有効化
                - ```off```


## サブネットグループを作成する
1. RDSコンソール画面からサブネットグループを選択
1. 画面右上の「DBサブネットグループを作成」を選択
1. DBサブネットグループを作成
    1. サブネットグループの詳細
        - 名前
            - ```kakazu_study_sre```
        - 説明
            - ```study_sre```
        - VPC
            - ```chura-sre-vpc```
    1. サブネットを追加
        - アベイラビリティーゾーン
            - ```ap-northeast-1a```
            - ```ap-northeast-1c```
        - サブネット
            - サブネットID```subnet-0ed13de406c14abb7```
            - サブネットID```subnet-03a074f6358a07898```
1. 作成

## 疎通確認
ec2 -> RDSへの疎通確認
1. 同じvpc内であることを確認する
    - ```chura-sre-vpc```
1. 同じセキュリティグループが使われていることを確認する
    - ```studying_sre_kakazu_sg_new```
1. インバウンドルールの編集で、RDSに対してec2のPrivateIPアドレスを許可する
    - タイプ ```MYSQL/Aurora```
    - ポート範囲 ```3306```
    - ソース ```ec2のPrivateIPアドレス```
1. ec2にssh接続 -> 疎通コマンド
    ```
    nc -zv RDSエンドポイント 3306
    ```
    ```
    # 結果
    Ncat: Version 7.50 ( https://nmap.org/ncat )
    Ncat: Connected to xxx.xxx.xxx.
    Ncat: 0 bytes sent, 0 bytes received in 0.01 seconds.
    ```