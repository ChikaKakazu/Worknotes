# Amazon ECSについて

ECSについて調べる
[Amazon Elastic Container Service とは](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/Welcome.html)

## ECSとは

コンテナ群をまとめて管理するフルマネージドなコンテナオーケストレータで、下記のようのな特徴がある
参考：[Amazon ECSとは？](https://dev.classmethod.jp/articles/re-introduction-2022-ecs/)

- コンテナの配置管理
- Load Balancer統合による負荷分散
- コンテナの状態監視と自動復旧
- コンテナのスケーリング
- コンテナのIAM権限管理
- CloudWatch Metrics・Logs統合

## 起動タイプ

参考：[Amazon ECS 起動タイプ](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/launch_types.html)

- Fargate
  - サーバーレスの従量制料金オプション。インフラストラクチャを管理することなく、コンテナを実行できる
    ※ サーバーレス：保守管理が必要ない(aws側がやってくれる)
  - [AWS Fargate での Amazon ECS](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/AWS_Fargate.html)
- EC2
  - クラスターにEC2インスタンスを設定してデプロイし、コンテナを実行する

## コンポーネント

参考：[Amazon ECS コンポーネント](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/welcome-features.html)

### クラスター

参考：[Amazon ECS クラスター](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/clusters.html)

- タスク、サービスの論理グループ

### タスク定義・タスク

参考：[Amazon ECSの タスク定義](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/task_definitions.html)
**タスク定義**

- アプリケーションを形成する1つ以上のコンテナを記述するテキストファイルでJSON形式になっている。これをもとに

**タスク**

- タスク定義をもとにコンテナを起動する役割を担う

### サービス
参考：[Amazon ECS サービス](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/ecs_services.html)
- クラスター内で必要な数のタスクを同時に実行し維持する。
- タスクが何らかの理由で停止、失敗した場合でも、別のタスク定義から別のタスクを起動し、設定された必要数を維持する
- オートスケーリングもできる
  
