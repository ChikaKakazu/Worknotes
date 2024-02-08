## Larastan導入・利用

### 導入
1. composerを使ってインストール
	```
	$ composer require larastan/larastan:^2.0 --dev
	```
1. 確認
	```
	$ ./vendor/bin/phpstan analyse -V
	PHPStan - PHP Static Analysis Tool 1.8.11
	```
	```
	$ composer show | grep larastan
	nunomaduro/larastan 1.0.4 Larastan - Discover bugs in your code without running it. A phpstan/phpstan wrapper for...
	```
1. 設定ファイルの作成
	- ファイル名は`phpstan.neon`もしくは`phpstan.neon.dist`にする
	```
	includes:
		- vendor/larastan/larastan/extension.neon

	parameters:

		paths:
			- app/

		# Level 9 is the highest level
		level: 5

	#    ignoreErrors:
	#        - '#PHPDoc tag @var#'
	#
	#    excludePaths:
	#        - ./*/*/FileToBeExcluded.php
	#
	#    checkMissingIterableValueType: false
	```
1. `phpstan.neon`の解説
	- 元々は`PHPstan`の設定ファイルでもある
	1. Larastanの設定を読み込むために必要
		```
		includes:
			- ./vendor/nunomaduro/larastan/extension.neon
		```
	1. 解析するディレクトリの指定
		```
		paths:
			- app
		```
	1. 解析レベル
		```
		level: 5
		```
	1. 正規表現でエラーを無視する
		```
		ignoreErrors:
			- '#Unsafe usage of new static#'
		```
	1. 解析したくないディレクトリ・ファイルの指定
		```
		excludePaths:
			- ./*/*/FileToBeExcluded.php
		```
	1. level6のタイプヒントのチェックで型がどちらでもいい場合にfalseを設定することで無効にできる
		```
		checkMissingIterableValueType: false
		```
1. 実行
	- 以下コマンドで実行
		```
		# メモリ足りないエラーが出たら--memory-limit=2Gを追加する 
		./vendor/bin/phpstan analyse --memory-limit=2G -c phpstan.neon
		```
