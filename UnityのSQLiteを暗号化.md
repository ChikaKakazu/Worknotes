# UnityのSQLiteを暗号化をかける
- [SQLite-net](https://github.com/praeclarum/sqlite-net#using-sqlcipher)
- 参考：[UnityにSQLCipherを導入して暗号化DBを使う](https://zenn.dev/shiena/articles/unity-sqlcipher#%E8%87%AA%E5%89%8D%E3%83%93%E3%83%AB%E3%83%89-1)

## sqlcipherのインストール
以前の方法では、**apple silicon(M1, M2)** のcpuだと動作しなかったので、こちらで対応
- [netpyoung/prebuilt-libsqlcipher](netpyoung/prebuilt-libsqlcipher)リポジトリをDLする
- `build-on-〇〇.sh`のシェルスクリプトを利用して、各OS毎にビルドする
  - `sh build-on-〇〇.sh`
  - 対象バージョンはシェルスクリプト内に埋め込まれているので、要確認
- ビルドが完了するとoutputフォルダが作られ、その中にそれぞれのOSのsqlcipherが作られる
  ```
  .
  ├── iOS
  │   ├── arm64
  │   │   └── libsqlcipher.a
  │   ├── armv7
  │   │   └── libsqlcipher.a
  │   ├── armv7s
  │   │   └── libsqlcipher.a
  │   ├── lipo
  │   │   └── libsqlcipher.a
  │   └── x86_64
  │       └── libsqlcipher.a
  ├── macOS
     ├── arm64
     │   └── sqlcipher.bundle
     ├── lipo
     │   └── sqlcipher.bundle
     └── x86_64
         └── sqlcipher.bundle
  ```
- できたファイルをそれぞれUnityの`Assets/Plugins`配下にコピーする
  - macOSだと以下のようになる
    - `Intel` => x86_64
    - `apple silicon` => arm64
  - iosだとおそらく`x86_64`で問題ない
- Unityに持ってきたファイルを選択し、インスペクターでosやcpuの設定が正しいか確認する
- Unityは同じファイルを置くことができないので、macOSのファイルはgitignoreで除外し、それぞれが管理する

## ~~(旧) sqlite-net-sqlcipherのインストール~~
<details>

- [jfcontart/SqlCipher4Unity3D_Apple](https://github.com/jfcontart/SqlCipher4Unity3D_Apple)リポジトリをDLする
- このリポジトリのREADMEを参考にビルドを行う
  - DLしたリポジトリに移動
  - `sh SQLCipherBuilt_Apple.sh (4.5.1)バージョン指定`
  - バージョンはリポジトリにある数字のフォルダを入れる。とりあえず一番数字の大きいものを入れておけばいいはず
- ビルドが完了すると、指定したバージョンのフォルダを開きそれをれをUnityの`Assets/Plugins`配下に**バージョン配下のフォルダ毎**コピーする
    ```sh
    # ビルド後のフォルダ
    .
    ├── iOS
    │   └── libsqlcipher.a
    ├── macOS
    │   └── sqlcipher.bundle
    └── tvOS
        └── libsqlcipher.a

    # Unityにコピー
    ├── iOS
    │   ├── libsqlcipher.a
    │   └── libsqlcipher.a.meta
    ├── iOS.meta
    ├── macOS
    │   ├── sqlcipher.bundle
    │   └── sqlcipher.bundle.meta
    └── macOS.meta
    ```
- 参考にしたサイトもしくは`jfcontart/SqlCipher4Unity3D_Apple`のREADMEを参考に、Unityにコピーしたファイルの設定を行う
- 同じく参考にしたサイトに従い、SQLiteからコピーした`SQLite.cs`に処理を追記する
- パスワードを使用したDBの作成処理に変更する
  
</details>

## 備考
- 下記エラーがでた場合、`Assets/StreamingAssets`配下にDBや、それ以外の場所・処理で暗号化を行う前のDBを参照している場合があるので、確認する
```
SQLiteException: file is not a database
SQLite.SQLite3.Prepare2 (System.IntPtr db, System.String query) (at Assets/Scripts/SQLiteNet/SQLite.cs:5022)
...
```