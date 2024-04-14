# LaravelからSlackにログを通知する
- Laravelには`config/logging.php`というファイルがあり、このファイルでログの設定を行なっている。
- `slack`というキーの設定とSlack側の設定を行うことで、slackにログを通知することができる
- Laravelの`config/logging.php`は基本触らない

## Slackの設定
- ログを通知する専用のチャンネルが存在する前提

### Webhook URLを取得する
1. 通知するチャンネルを右クリックし、**チャンネル詳細を表示する**をクリック
2. **インテグレーション**タグを選択し、**アプリを追加する**をクリックする
    ![alt text](<images/スクリーンショット 2024-04-10 14.23.36.png>)

3. **Incoming Webhook**と検索し追加する
   1. チャンネルを設定できるので、ログを通知するチャンネルを選択する
   ![alt text](<images/スクリーンショット 2024-04-10 14.27.09.png>)
4. 追加すると、「セットアップの手順」の項目に**Webhook URL**が表示されるので、これをコピーする
5. 上手く追加できると、追加したチャンネルに追加した旨のチャットが投稿される

## Laravelの設定
- .envファイルにある**LOG_SLACK_WEBHOOK_URL**にSlackでコピーしたWebhook URLを貼り付ける

## 設定の確認
1. .envの**LOG_CHANNEL**を`config/logging.php`の設定に合わせて変更する
2. `config/logging.php`の`slack`の設定に合わせてログを出す
   - `level`が**error**になっているのでエラーログを出す
    ```php
    // config/logging.php
    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'username' => env('LOG_SLACK_NAME'),
        'emoji' => env('LOG_SLACK_ICON'),
        'level' => 'error',
    ],
    ```
    ```php
    // routes/web.php
    // localhost:8080
    $router->get('/', function () use ($router) {
        \Log::error('slack通知のテスト');
        return 'api';
    });
    ```
3. slackにログが通知されていることを確認する