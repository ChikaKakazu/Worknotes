# Lumenにlaravel-modulesの導入手順
- [laravel-modules](https://github.com/nWidart/laravel-modules)

## 導入
1. 基本的にはReadMeの手順を行う
	1. `composer require nwidart/laravel-modules`
	1. Lumenには`php artisan vendor:publish`コマンドがないので下記手順は飛ばす
		- `php artisan vendor:publish --provider="Nwidart\Modules\LaravelModulesServiceProvider"`
	1. composer.jsonに登録
		```
		"autoload": {
			"psr-4": {
				"App\\": "app/",
				"Modules\\": "Modules/", <- 追加
				"Database\\Factories\\": "database/factories/",
				"Database\\Seeders\\": "database/seeders/"
			}
		}
		```
	※上記内容がLaravelでの導入方法。Lumenだとさらに手順が増える
1. ドキュメントを見て追加の手順を行う
	- [ドキュメント](https://docs.laravelmodules.com/v10/lumen)
	- `$app->bind`はサービスプロバイダのすぐ上に記述すれば問題ないはず

## 発生したエラー
- モジュール作成コマンド`php artisan module:make <module-name>`を入力した際に下記エラーが発生
	- `Call to undefined method Laravel\Lumen\Application::booted() `
		- 原因
			- `LaravelModulesServiceProvider.php`の40行目で読んでいる`$this->app->booted()`メソッドが呼び出せないエラー
			- サービスプロバイダの`$this->app`は`vendor/illuminate/contracts/Foundation/Application.php`を持っている
			- `$app`で呼んでいるメソッドは`vendor/laravel/lumen-framework/src/Application.php`で実装されているものしか呼べ来っぽい
			- `booted()`メソッドが実装されていないため呼べない
		- 対応
			- サービスプロバイダが`booted()`メソッドを実装しているので、そちらを呼ぶようにする
			- `$this->app->booted` -> `$this->booted`
	- 1度目は成功したが、2度目を行うと、別のエラーが発生
		- `Class "Illuminate\Foundation\AliasLoader" not found `
		- 原因
			- `Module.php`で存在しないクラスを呼んでいるエラー
			- `AliasLoader::getInstance()->alias()`を呼びたいが、`AliasLoader`がない
		- 対応
			- `$this->app`の`vendor/illuminate/contracts/Foundation/Application.php`が同じ`alias()`メソッドを実装しているので、それを呼ぶことで対応

		- `Module.php`には他にも存在しないクラスを呼んでいるので、対応する
			- `registerProviders()`メソッドの中身をコメントアウトする
			- 上の対応はちょっと怪しい...けど動く！
