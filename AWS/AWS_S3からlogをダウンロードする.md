# s3からlogをダウンロードする
- 参考  
  - [AWS CLI での高レベル (S3) コマンドの使用](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-services-s3-commands.html)
  - [AWS CLIでS3を操作するコマンド一覧](https://qiita.com/uhooi/items/48ef6ef2b34162988295)
  - [【AWS CLI】実際に業務で使ったAWSのコマンドたち](https://qiita.com/Ryoga_aoym/items/686c03a9ac09ead3c4af)

## ディレクトリを指定してダウンロード

```
# aws s3 cp s3://対象ディレクトリ(DL元) DL先 --recursive
> aws s3 cp s3://〇〇-laravel-logs/development/ . --recursive
download: s3://〇〇-laravel-logs/development/2021-04-21/23db5787-2889-4601-873a-99e4fa09456d/i-0377be99224f/000000.gz to 2021-04-21/23db5787-2889-4601-873a-99e4fa09456d/i-0377be99224f/000000.gz
download: s3://〇〇-laravel-logs/development/2021-04-21/aws-logs-write-test to 2021-04-21/aws-logs-write-test
download: s3://〇〇-laravel-logs/development/2021-04-21/23db5787-2889-4601-873a-99e4fa09456d/i-e3ca2cacd5d1/000000.gz to 2021-04-21/23db5787-2889-4601-873a-99e4fa09456d/i-e3ca2cacd5d1/000000.gz
download: s3://〇〇-laravel-logs/development/2021-04-21/23db5787-2889-4601-873a-99e4fa09456d/i-ebbf3ca2a515/000000.gz to 2021-04-21/23db5787-2889-4601-873a-99e4fa09456d/i-ebbf3ca2a515/000000.gz
download: s3://〇〇-laravel-logs/development/2021-04-21/23db5787-2889-4601-873a-99e4fa09456d/i-5aaeb4bc60bd/000000.gz to 2021-04-21/23db5787-2889-4601-873a-99e4fa09456d/i-5aaeb4bc60bd/000000.gz
```
```
> tree
.
└── 2021-04-21
    ├── 23db5787-2889-4601-873a-99e4fa09456d
    │   ├── i-0377be99224f
    │   │   └── 000000.gz
    │   ├── i-5aaeb4bc60bd
    │   │   └── 000000.gz
    │   ├── i-e3ca2cacd5d1
    │   │   └── 000000.gz
    │   └── i-ebbf3ca2a515
    │       └── 000000.gz
    └── aws-logs-write-test
```

## 特定の月のログファイルを表示する

```
> aws s3 ls s3://〇〇-laravel-logs/production/2023-01
PRE 2023-01-01/
PRE 2023-01-02/
....
PRE 2023-01-30/
PRE 2023-01-31/
```

## 指定した月のログファイルをダウンロード
```
# --dryrunでテスト
aws s3 cp s3://〇〇-laravel-logs/production/ . --recursive --dryrun --exclude "*" --include "2023-01*"

# 実行
aws s3 cp s3://〇〇-laravel-logs/production/ . --recursive --exclude "*" --include "2023-01*"
```

## 一括解凍

```
> find . -name '000000.gz' | xargs -n1 gunzip
```
```
> tree
.
└── 2021-04-21
    ├── 23db5787-2889-4601-873a-99e4fa09456d
    │   ├── i-0377be99224f
    │   │   └── 000000
    │   ├── i-5aaeb4bc60bd
    │   │   └── 000000
    │   ├── i-e3ca2cacd5d1
    │   │   └── 000000
    │   └── i-ebbf3ca2a515
    │       └── 000000
    └── aws-logs-write-test
```