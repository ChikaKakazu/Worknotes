# Lumenで複数のAuthミドルウェアを追加する方法
- Lumen10, php8.2で実装

## MiddlewareとProviderの作成
- Middlewareの作成
  - デフォルトで用意されている**Authenticate.php**を複製してもよい
- Providerの作成
  - デフォルトで用意されている**AuthServiceProvider.php**を複製してもよい
- Providerのbootメソッド内の処理を修正する
    ```php
    // 修正前
    $this->app['auth']->viaRequest('api', function ($request) {
    });

    // 修正後
    // viaRequest('api' -> viaRequest('sample'
    $this->app['auth']->viaRequest('sample', function ($request) {
    });
    ```
- **bootstrap/app.php**を編集する
```php
// ミドルウェアの追加
$app->routeMiddleware([
    'auth' => App\Http\Middleware\Authenticate::class,
    // 追加
    'loginAuth' => \App\Http\Middleware\LoginAuthenticate::class,
]);
...
// プロバイダの追加
$app->register(App\Providers\AppServiceProvider::class);
$app->register(App\Providers\AuthServiceProvider::class);
// 追加
$app->register(App\Providers\AuthLoginServiceProvider::class);
```

## auth.phpの追加
- Laravelではデフォルトで**config/auth.php**がある(らしい)
- **vendor/laravel/lumen-framework/config/auth.php**を**config/auth.php**に複製する
- 以下を追加
    ```php
    'guards' => [
            'api' => ['driver' => 'api'],
            // 追加
            'loginAuth' => [
                'driver' => 'sample',// ProviderのviaRequestと同じにする
                'provider' => 'loginAuth',
            ],
        ],

    'providers' => [
            // 追加
            'loginAuth' => [
                'driver' => 'eloquent',
                'model' => App\Models\User\User::class,
            ],
        ],
    ```
    - [[図解] Laravel の認証周りのややこしいあれこれ。](https://zenn.dev/ad5/articles/48671b32c89897)

## ルーティングを設定する
- 作成したAuthミドルウェアを**web.php**に設定する
- 設定するミドルウェアは**bootstrap/app.php**に記述したミドルウェア設定のkeyを記述する
- authのみ**auth:〇〇**の形で記述する
    ```php
    $router->group(['prefix' => 'v1'], function () use ($router) {
        // デフォルトのAuthミドルウェア
        // auth:api と記述する
        $router->group(['middleware' => ['auth:api']], function () use ($router) {
            $router->group(['prefix' => 'User'], function () use ($router) {
                $router->get('', 'UserApi@index');
            });
        });
    });

    $router->group(['prefix' => 'v2'], function () use ($router) {
        // 新規追加したAuthミドルウェア
        // auth:loginAuth と記述する
        $router->group(['middleware' => ['auth:loginAuth']], function () use ($router) {
            $router->group(['prefix' => 'User'], function () use ($router) {
                $router->get('', 'UserApi@index2');
            });
        });
    });
    ```

## 注意
- `$request->user()`はデフォルト設定で**auth.php**のdefaultsに設定してあるものが呼ばれる(基本は`api`)
  - web.phpでデフォルトのAuthミドルウェアを設定していなくても、`$request->user()`を呼ぶと、なぜかデフォルトのAuthミドルウェアが呼ばれる。
  - 以下対応方法
    ```php
    $user = Auth::guard('loginAuth')->user();
    if (!$user instanceof \App\Models\User\User) {
        return response('Unauthorized.', 401);
    }
    ```