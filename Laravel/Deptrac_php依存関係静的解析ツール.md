# Deptracの導入
- https://github.com/qossmic/deptrac
- [ドキュメント](https://qossmic.github.io/deptrac/)

## インストール
- `composer require --dev qossmic/deptrac-shim`
- `vendor/bin/deptrac init`コマンドで設定ファイルのテンプレートを作成する
	- 作成された`deptrac.yaml`に依存関係の設定を記述する

## 参考
- [レイヤー間の依存関係の静的解析 - PHP deptrac ~ 導入編](https://engineering.otobank.co.jp/entry/2021/01/25/185242)
- [モジュールのドメイン境界を強制](https://youtu.be/qsDKaO-lLdw?t=2895)
	- [GitHub](https://github.com/avosalmon/modular-monolith-laravel/tree/main)
- [アーキテクチャのコード品質を守りたい！](https://zenn.dev/ysit/articles/layered-architecture-lint-check#deptrac%E3%81%AE%E8%A8%98%E6%B3%95%E3%83%AB%E3%83%BC%E3%83%AB)
- [Laravelをモジュラモノリスで開発する](https://zenn.dev/ukeloop/articles/99337c50da2086)

