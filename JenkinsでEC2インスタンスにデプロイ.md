# JenkinsでEC2インスタンスにデプロイ

## 始めに構築したec2インスタンスにDocker,docker-compose,gitを導入する

- ```sudo yum update```で各種アップデートしておく

### Dockerの導入
1. ```sudo yum install docker```
    - dockerのインストール
    - ```docker -v```でバージョン確認
1. ```sudo service docker start```
    - dockerの起動
1. ```sudo usermod -aG docker ec2-user```
    - sudoなしでdockerコマンドが動かせるように
1. 一度ログアウトし、再度sshログインする
1. ```docker info```
    - 起動確認

### docker-composeの導入
1. ```sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose```
    - 1.29.2の部分がバージョン
    - https://github.com/docker/compose/releases でバージョン確認
1. ```sudo chmod +x /usr/local/bin/docker-compose```
    - 権限の変更
1. ```docker-compose --version```
    - バージョン確認

### gitのインストール
1. ```sudo yum install git```
    - gitのインストール
1. ```git version```
    - バージョン確認

### 参考
- [AWS EC2 Amazon LinuxでDocker, Docker Composeをインストールする](https://qiita.com/shinespark/items/a8019b7ca99e4a30d286)
- [【AWS】EC2にDockerとDocker Composeをインストール](https://kacfg.com/aws-ec2-docker/)
- [AWS EC2 AmazonLinux2 Gitをインストールする](https://qiita.com/miriwo/items/8d5b35950232c1126d36)

## jenkinsサーバーにソースが反映されているか確認
1. 作成したジョブを実行する
    - 「ビルド実行」
1. jenkinsサーバーにsshログインする
1. jenkinsサーバー内の```workspace```ディレクトリに実行したジョブで連携したソースコードが反映される

## 自サーバーにjenkinsコンテナからssh接続できるようにする
jenkinsサーバーの公開鍵を自サーバーに登録し、jenkinsからssh接続の疎通確認を行う<br>
**前提：**
jenkins用のdockerコンテナがjenkinsサーバー内で立ち上がっている
1. コンテナを確認し、コンテナ内に入る
    - ```docker ps -a```でコンテナの確認
    - ```docker exec -ti コンテナID bash```でコンテナに入る
1. ```.ssh/id_rsa.pub```に記述されている公開鍵を自サーバーの```.ssh/authorized_keys```に登録する
    - **注意**
        - 文字数が多くて改行されているように見えるが1行に書かれている
        - oで「下の行にインサート」すると良き(vimの場合)
        - **元々記述されている内容はこのサーバーにssh接続する情報なので消したりすると終わるので注意**
        - **jenkinsの公開鍵情報を追記した後、サーバーをログアウトせずに別タブを開き、対象のサーバーにssh接続できるか確認する**
1. 対象サーバーのセキュリティグループにjenkinsのアクセスを許可する
1. jenkinsコンテナに戻り、対象のサーバーに対してssh接続を行う
    - 鍵はid_rsaを指定する

## rsyncコマンドでプロジェクトの同期を行う
#### 同期のテストを行う
1. jenkinsサーバーで同期したいjobのworkspaceに移動する
1. テスト用にテキストファイルを作成
1. jenkinsサーバーで動いているdockerコンテナに入る
1. コンテナ内で同期したいjobのworkspaceに移動する
1. 対象のサーバーに対して同期する
    ```
    # rsyncでテキストファイルを同期
    rsync --delete -avz -O --no-o --no-p --no-g \
        # --dry-runは同期は行わず、結果を返す。
        --dry-run \
        -e "ssh -i /var/jenkins_home/.ssh/id_rsa" ./test_rsync.txt \
        ec2-user@${TARGET_SERVER}:/home/ec2-user/
    ```
1. 結果が返ってきて問題なければ```--dry-run```オプションを外して実行する
1. 対象のサーバーにsshログインし、ファイルがあるか確認する

#### 実際にプロジェクトを同期する
1. jenkinsサーバーで動いているdockerコンテナに入る
1. コンテナ内で同期したいjobのworkspaceに移動する
1. 同期コマンドを実行する
    ※${TARGET_SERVER}には対象のサーバーのアドレスを入れる
    ```
    # プロジェクトを同期
    rsync --delete -avz -O --no-o --no-p --no-g \
        --dry-run \
        --exclude ".git*" --exclude "editorconfig" --exclude "vendor" \
        -e "ssh -i /var/jenkins_home/.ssh/id_rsa" ./ \
        ec2-user@${TARGET_SERVER}:/home/ec2-user/lumen-project/
    ```
1. 結果が返ってきて問題なければ```--dry-run```オプションを外して実行する
1. 対象のサーバーにsshログインし、プロジェクトがあるか確認する

## 対象のサーバー上でプロジェクトを動かす
1. 同期したプロジェクトディレクトリに移動し、dockerコンテナを立ち上げる
1. 立ち上がっているか確認する
1. vendorフォルダがプロジェクトには含まれていない(gitで除外)ので、プロジェクトに入れる
    - 対象のサーバーにはphpやcomposerがインストールされていないので、dockerコンテナ内で導入作業を行う
    - ```composer install```
1. 再度サーバーにログインし、dockerコンテナは立ち上がっていて、vendorは存在することを確認する
1. プロジェクトが動いているか確認する
    ※ dockerで8080を80番ポートに変換しているが、セキュリティグループで8080のアクセスを弾いている。
    1. セキュリティグループで8080を許可する
    1. apiのurlを叩く
        ※ https通信は証明書などを発行していないのでhttpで行う
1. Unityと通信できているか確認する
    - Unityの向き先をlocalhostからサーバーに変更する
1. lumen-projectを何かしら変更して、gitにpush。再度jenkinsでジョブを実行する
1. 変更が反映されていて、lumenが動き、unityと通信できていることを確認する

※ vendorに何かしら変更があった場合(バージョンアップなど)はgitでcomposer.jsonを管理しているので、都度サーバー内でcomposer installする

### rsync
ファイル同期のコマンド
```
# 使われているシェルスクリプト
rsync --delete -avz -O --no-o --no-p --no-g \
--exclude ".git*" --exclude "editorconfig" \
--exclude "storage" --exclude "bootstrap/cache" \
-e "ssh -i /var/jenkins_home/.ssh/id_rsa" ./ \
ec2-user@${TARGET_SERVER}:/home/ec2-user/${TARGET_DIR}/
```
| オプション           | 別名             | 説明                                                                                                                   |
| -------------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------- |
| -a                   | --archive        | 転送元のディレクトリを再帰的にオーナー・グループ・パーミッション・タイムスタンプをそのままコピーする(アーカイブモード) |
| -e                   | --rsh=COMMAND    | リモートシェルを指定する(ssh接続するときとかに使用)                                                                    |
| -v                   | --verbose        | コピーしたファイル名やバイト数などの転送情報を出力                                                                     |
| -z                   | --compress       | データ転送時に圧縮                                                                                                     |
| --no-o --no-p --no-g |                  | 所有者、パーミッション, グループの設定を同期から外す(別サーバーの場合)                                                 |
| -O                   | --omit-dir-times | タイムスタンプを保持しない                                                                                             |
| -n                   | --dry-run        | コピーや転送を実際には行わず転送内容のみ出力                                                                           |
| --delete             |                  | コピー元にない(削除された)ファイルをコピー先で削除する                                                                 |
| --exclude=PATTERN    |                  | 同期から除外(ファイルやフォルダ、ワイルドカードでの指定ができる)。複数ある場合は–exclude-fromが便利                    |
| –exclude-from        |                  | 除外リストファイルを用意し、それを指定する                                                                             |