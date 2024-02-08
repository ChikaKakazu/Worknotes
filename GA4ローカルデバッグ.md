# GA4をローカルでデバッグする
- 埋め込みタグを利用する
- GA4 DebugViewを利用してデバッグする

## Google Analytics Debugger
- Google Chromeにデバッグ用の拡張機能をインストールする
- https://chrome.google.com/webstore/detail/google-analytics-debugger/jnkmfdileelhofjcijamephohjechhna

## GA4画面での準備
1. HTMLに埋め込んだタグに`{‘debug_mode’:true}`を追加する
	- 以下のように追加する
		```
		gtag(‘config’, ‘G-(ID)’,{‘debug_mode’:true});
		```

## DebugViewの動作確認
1. 設定の真ん中あたりに「DebugView」があるのでクリック
1. 設定が正しくされていれば、ウェブ上でのアクションに合わせて動作する
