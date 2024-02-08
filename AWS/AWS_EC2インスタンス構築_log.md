# AWS_EC2インスタンス構築の作業ログ

作業内容
- [x] EC2インスタンスを起動する
- [x] セキュリティグループの設定
- [x] sshログイン/configの設定
- [x] ec2インスタンスの時間を日本時間に変更
- [x] セキュリティアップデート

# サーバー構築時の作業ログ

1. キーペアの作成
    1. 名前を```studying_sre_kakazu.pem```とし、キーペアのタイプを```RSA```、拡張子を```.pem```で作成
    1. 作成したら自動でDLされたので、作成したキーペアを自分の.sshに移動
        - ```cp Downloads/studying_sre_kakazu.pem .ssh/```
    1. そのままだとsshアクセスしようとする際に権限ないよエラーでるので、パーミッションの変更
        - ```chmod 600 studying_sre_kakazu.pem```

    - 参考：https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair

1. セキュリティグループの作成
    1. セキュリティグループ名、説明、VPCの設定
        - ```studying_sre_kakazu_sg```で作成
        - VPCは```vpc-07ab899cf13572def(ChurappsVPN-VPC)```
    1. インバウンドルールの設定
        - SSH接続のためにタイプをSSHで設定
        - ブラウザから確認もしたいのでHTTP通信も許可
        - ソースの設定をマイIPにすることで、自分のIPアドレスを調べずに済む。他の人を許可するならカスタムにする。全て許可するならカスタムにして```0.0.0.0/0```を設定する(ガバガバだから基本しなそう。HTTP/Sならやるかも)
    1. アウトバウンドルールの設定
        - 基本的には設定しなくてもいい
        - インバウンドルールで許可した通信をみて自動的に割り振ってくれる。(**ステートフル**)
    1. タグの設定
        - キーをNameにして、値を適当な名前
        - 今回は```studying_sre_kakazu_sg```
        - 参考：https://dev.classmethod.jp/articles/aws-tagging-basic/
    - 参考：https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/working-with-security-groups.html

1. インスタンスの作成
    1. AMIの選択
        - Linux2のAMIを選択
        - カーネルの新しいものを選択
        - ```Amazon Linux 2 AMI (HVM) - Kernel 5.10, SSD Volume Type ```
        ※選択項目に64ビット(x86)と(Arm)があるが、cpuのアーキテクチャの違いらしい。x86がintel, AMDらしいので、基本それでいいはず。
        参考：https://urashita.com/archives/12325
    1. インスタンスタイプの選択
        - 今回は指定のあった```t3.micro```を選択
        - インスタンスの詳細の設定を選択
    1. インスタンスの詳細の設定
        - インスタンス数
            - 今回は1台
        - 購入のオプション
            - チェックは外す(デフォルトで外れていた)
        - ネットワーク
            - セキュリティグループと同様に```ChurappsVPN-VPC```
        - サブネット
            - VPCを選択したら自動で選択された
        - 自動割り当てパブリック IP
            - 有効にする
        - ホスト名のタイプ
            - サブネット設定を使用
        - DNS Hostname
            - リソースベースの IPv4 (A レコード) DNS リクエストを有効化
        - 配置グループ
            - チェックは外す(デフォルトで外れていた)
        - キャパシティーの予約
            - なし
        - ドメイン結合ディレクトリ
            - ディレクトリなし
        - IAMロール
            - なし
        - CPUオプション
            - なし
        - シャットダウン動作
            - 停止
        - 停止 - 休止動作
            - チェック外す
        - 終了保護の有効化
            - チェック外す
        - モニタリング
            - チェック外す
        - テナンシー
            - 共有 - 共有ハードウェアインスタンスの実行
        - Elastic Inference
            - チェック外す
        - クレジット仕様
            - チェック入れる
        - ネットワークインターフェース
            - 触らない
        - Enclave
            - チェック外す
        - アクセス可能なメタデータ
            - 有効
        - メタデータのバージョン
            - v1およびv2
        - メタデータトークンレスポンスのホップ制限
            - 1
        - Allow tags in metadata
            - 無効
        - ユーザーデータ
            - 触らない
    1. ストレージの追加
        - デフォルトの設定のまま
    1. タグの追加
        - 名前をつけるため、キーを```Name```にして値を```studying_sre_kakazu_ec2```で追加
    1. セキュリティグループの設定
        - はじめに作成した```studying_sre_kakazu_sg```を選択
    1. 確認
        - しっかり確認！
    1. 起動
        - キーペアを選択する
        - はじめに作成した```studying_sre_kakazu```を選択

1. インスタンスに接続
    - コマンドの確認
        ```
        ssh -i キーペア.pem ユーザー名@インスタンスのパプリック名
        ```
    - キーペアの確認
        - 作成時に```.ssh```に移動したので、確認
        - 読み取り権限があるかの確認
        ```
        # 移動
        cd .ssh/

        # ファイルがあるか・読み取り権限があるか
        ls -l studying_sre_kakazu.pem
        -rw-------@ 1 kakazu  staff  1678  1 28 11:51 studying_sre_kakazu.pem
        ```
    - ユーザー名
        - ```ec2-user```
    - インスタンスのパプリック名
        - コンソール画面で```パブリック IPv4 DNS```を確認
            ない...
            - 原因
                - vpcの設定```「DNSホスト名/DNS解決」```が足りていないとだめらしい
                - Elastic IPで接続する
                ### Elastic IPの作成
                1. ネットワークボーダーグループが東京リージョンの```ap-northeast-1```になっている
                1. パブリック IPv4 アドレスプールがAmazon IPv4になっている
                1. タグに名前をつけるため、キーを```Name```にして値を```studying_sre_kakazu```で追加
                1. 作成したElastic IPをec2インスタンスに割り当てる
                ### Elastic IPアドレスの関連付け
                1. 画面右上のアクションを選択
                1. Elastic IPアドレスの関連付けを選択
                1. リソースタイプをインスタンスに、インスタンスを作成した```studying_sre_kakazu_ec2```に設定。プライベートIPアドレスもインスタンスに設定されているものに。
                1. 関連付けたあと、インスタンスの一覧に戻り、作成したインスタンスにElastic IPアドレスが関連付けられているか確認
    - 接続
        - ```ssh -i .ssh/studying_sre_kakazu.pem ec2-user@3.114.132.247```

## .ssh/configの作成

1. configファイルがあるか確認
    ```
    cd .ssh
    ls -la ls -la config
    -rw-r--r--  1 kakazu  staff  1162  2  8 11:30 config
    ```
    すでにあったので新たに作成はしない。
1. 作成したインスタンスにssh接続できるように記述
    ```
    # ssh接続時のコマンド
    ssh -i .ssh/studying_sre_kakazu.pem ec2-user@3.114.132.247
    ```
    ```
    # 設定
    Host study_sre
	User ec2-user
	HostName 3.114.132.247
	IdentityFile ~/.ssh/studying_sre_kakazu.pem
	port 22
    ```
1. 実際にssh接続できるか
    ```
    ssh study_sre

    Last login: Tue Feb  8 02:30:17 2022 from 072213014222.ppp-oct.au-hikari.ne.jp

        __|  __|_  )
        _|  (     /   Amazon Linux 2 AMI
        ___|\___|___|

    https://aws.amazon.com/amazon-linux-2/
    ```

## インスタンスのタイムゾーンを日本時間にする

1. 確認
    1. ssh接続
    ```
    ssh study_sre
    ```
    1. 現在のタイムゾーンの確認
    utcになっている
    ```
    [ec2-user@ip-10-0-0-124 ~]$ date
    2022年  2月  8日 火曜日 03:07:22 UTC

    [ec2-user@ip-10-0-0-124 ~]$ timedatectl
          Local time: 火 2022-02-08 03:15:15 UTC
      Universal time: 火 2022-02-08 03:15:15 UTC
            RTC time: 火 2022-02-08 03:15:16
           Time zone: n/a (UTC, +0000)
         NTP enabled: yes
    NTP synchronized: yes
     RTC in local TZ: no
          DST active: n/a
    ```
1. 日本時間のタイムゾーンを探す
    ```
    [ec2-user@ip-10-0-0-124 ~]$ timedatectl list-timezones | grep Tokyo
    Asia/Tokyo
    ```
1. タイムゾーンを設定する
    ```
    sudo timedatectl set-timezone Asia/Tokyo
    # 変更されているか確認
    date
    2022年  2月  8日 火曜日 12:30:18 JST
    ```

## セキュリティアップデートを行う
1. screenコマンドでsshのセッションが切れても大丈夫な状態を作る
1. updateの内容の確認
    ```
    yum check-update
    読み込んだプラグイン:extras_suggestions, langpacks, priorities, update-motd
    amzn2-core

    ca-certificates.noarch                                                               2021.2.50-72.amzn2.0.3                                                                amzn2-core
    ec2-net-utils.noarch                                                                 1.6-1.amzn2                                                                           amzn2-core
    ec2-utils.noarch                                                                     1.2-46.amzn2                                                                          amzn2-core
    kernel.x86_64                                                                        5.10.96-90.460.amzn2                                                                  amzn2extra-kernel-5.10
    kernel-tools.x86_64                                                                  5.10.96-90.460.amzn2                                                                  amzn2extra-kernel-5.10
    openssh.x86_64                                                                       7.4p1-22.amzn2.0.1                                                                    amzn2-core
    openssh-clients.x86_64                                                               7.4p1-22.amzn2.0.1                                                                    amzn2-core
    openssh-server.x86_64                                                                7.4p1-22.amzn2.0.1                                                                    amzn2-core
    ```
1. updateを行う
    ```
    sudo yum update
    ...
    ...

    更新:
    ca-certificates.noarch 0:2021.2.50-72.amzn2.0.3    ec2-net-utils.noarch 0:1.6-1.amzn2             ec2-utils.noarch 0:1.2-46.amzn2               kernel-tools.x86_64 0:5.10.96-90.460.amzn2
    openssh.x86_64 0:7.4p1-22.amzn2.0.1                openssh-clients.x86_64 0:7.4p1-22.amzn2.0.1    openssh-server.x86_64 0:7.4p1-22.amzn2.0.1

    完了しました!
    ```
1. updateがもう無いかを再度確認
    ```
    yum check-update
    読み込んだプラグイン:extras_suggestions, langpacks, priorities, update-motd
    Security: kernel-5.10.96-90.460.amzn2.x86_64 is an installed security update
    Security: kernel-5.10.93-87.444.amzn2.x86_64 is the currently running version
    ```
1. 再起動が必要かどうかの確認
    ```
    needs-restarting -r
    Core libraries or services have been updated:
    kernel -> 5.10.96-90.460.amzn2

    Reboot is required to ensure that your system benefits from these updates.

    More information:
    https://access.redhat.com/solutions/27943
    ```
1. 再起動が必要そうなので、再起動
    ```
    sudo reboot
    Connection to 3.114.132.247 closed by remote host.
    Connection to 3.114.132.247 closed.
    ```
1. もう一度ssh接続
    ```
    ssh study_sre
    ```