# LarastanのLevelについて調査する

## Level
- [こちら](https://qiita.com/shimabox/items/df03dde8bd6db4733231)を参考に各Levelを判断する
- [公式](https://phpstan.org/user-guide/rule-levels)

## Level 0
- 基本的なチェック。未知のクラス、未知の関数、$this上で呼び出された未知のメソッド、それらのメソッドや関数に渡された引数の数が間違っているか、常に未定義の変数をチェックする
- 主なエラー
	- `Access to an undefined property`
- エラーの修正
	- 動的プロパティを使用していたので、プロパティを宣言した

## Level 1
- 未定義の変数、__call と __get を持つクラスの未知のマジックメソッドとプロパティがある可能性がある 
- 主なエラー
	- `Access to an undefined property`
		- `Eloquent\Model`でエラーが出る。DBにアクセスして存在するカラムの中身を取得するが、ソースコードには定義されていないので出るっぽい
- エラーの修正
    - 案1: DocBlockにコメントを追加する
        ```php
        /**
        * @property int $user_id ユーザーID
        */
        class User extends Model {}
        ```
	- 案2: `phpstan.neon`に無視するエラーとして以下追加
		```
		ignoreErrors:
			- '#Access to an undefined property App\\Domain(.*)#'
		```
- 問題
	- 案2は通常の`Access to an undefined property`も無視されるので、非推奨。

## Level 2
- ($this だけでなく)すべての式で未知のメソッドをチェックし、PHPDocs を検証する
- 主なエラー
	- `Unsafe call to private method`
		- メソッドの呼び出し方が違うエラー。
- エラーの修正
	- `private static`メソッドを`static::function`で呼んでいたので`self::function`に変更

## Level 3
- 戻り値の型、プロパティに割り当てられた型の確認
- 主なエラー
	- `should return int but returns `
		- 返り値が違う
- エラーの修正
	- 返り値の修正

## Level 4
- 基本的なデッドコードチェック - instanceofやその他の型チェックが常にfalse、到達しないelse文、return後の到達不能コードなど
- 主なエラー
	- `Unreachable statement - code above always terminates.` 
		- 早期終了し、処理が最後まで通らない
- エラーの修正
	- 以前の不具合の調査用処理だったので早期リターンした。なので、エラーの出たファイルを無視するように設定

## Level 5
- メソッドや関数に渡される引数の型チェック
- エラー無し

## Level 6
- タイプヒントの欠落を報告する
- エラー内容が多いので修正対応せず
- 主なエラー
	- `has no return type specified.`
		- 返り値のないメソッド
	- `has parameter property with no type specified.`
		- 型のない引数