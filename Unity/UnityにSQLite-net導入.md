# UnityにSQLite-netを導入する
UnityでSQLiteを使用するための導入まとめ。
UnityプロジェクトとSQLite-netをダウンロードしているものとする。
MacOS, Ios, Androidで動作することを確認する。
非同期でのDB作成やデータ流し込みなども可能だが、一旦は行わない。
- 以下テスト環境
  - Unity : 2022.3.11f1
  - SQLite-net : https://github.com/praeclarum/sqlite-net

## 全プラットフォーム共通(前提作業)
- ダウンロードしたSQLite-netから`SQLite.cs`, `SQLiteAsync.cs`をプロジェクトに追加する
  - 例：`Assets/SQLiteNet/SQLite.cs, SQLiteAsync.cs`
- MacOSでSQLite-netを扱う場合は使用するファイルで`using SQLite`を記述する
- SQLiteのDBを作成する場所に気をつける
  - それぞれのプラットフォームごとに参照するDBパスが違う
  - Unity側が用意している`Application.persistentDataPath`の場所に動的に更新可能なDBが保存される
    - 基本的にDB作成する時のパスは`Application.persistentDataPath`を指定すれば問題ない

## MacOS
- BuildSettings->TargetPlatformからOSが`macOS`になっていることを確認する
- ビルドし、確認

## Android
- ビルドのプラットフォームが`Android`になっていることを確認する
- AndroidでSQLiteを動作させるにはnative pluginが必要
  1. [SQLite](https://www.sqlite.org/download.html)
  2. `Precompiled Binaries for Android`の項目にある`sqlite-android-〇〇.aar`というファイルをDLする
  3. DLしたファイルをUnityプロジェクトの`Assets/Plugins`配下に置く
     1. 例：`Assets/Plugins/Android/sqlite-android-3430200.aar`
     2. おそらく`Plugins`配下であれば前後にディレクトリが存在しても問題なく動く
  4. `sqlite-android-〇〇.aar`を選択し、Inspectorを見る
  5. Inspectorの`Select platform for plugin`にAndroidのみチェックを入れる。その後Applyをクリックし更新。元々入っていれば何もしない。
- `SQLite.cs`に処理を追加する
    ```C#
    // SQLite.cs

    // before
    const string LibraryPath = "sqlite3";

    // after
    #if UNITY_ANDROID && !UNITY_EDITOR
            const string LibraryPath = "sqliteX";
    #else
            const string LibraryPath = "sqlite3";
    #endif
    ```
- ビルドを行い、作成された`.apk`ファイルをAndroid端末でDLして解凍
- アプリが作られるので実行し確認

## Ios
- ビルドのプラットフォームが`iOS`になっていることを確認する
- iOSもMacOSと同様に、何もせずビルドする
- iOSビルドには`Xcode`を利用する必要がある
  - [参考](https://rabbitsui.com/unity-ios-machinetest/)
- iOS16から`Developer Mode(開発者モード)`が導入され、それを有効にする必要がある
  - [参考](https://softmoco.com/devenv/how-to-enable-developer-mode-iphone.php)



## サンプル
- データベース作成
    ```C#
    // パスワード・保存先パス
    var pass = "password";
    var databasePath = string.Format ("{0}/{1}", Application.persistentDataPath, ConnectionManager.DATABASE_NAME);
    // データベースの設定
    var option = new SQLiteConnectionString(databasePath, SQLiteOpenFlags.Create | SQLiteOpenFlags.ReadWrite | SQLiteOpenFlags.FullMutex, true, pass);
    // データベースの作成
    var db = new SQLiteAsyncConnection(option);
    ```
- テーブル作成
    ```C#
    // テーブルの元となるクラス
    [Table("Units"), System.Serializable]
    public class Unit
    {
        [PrimaryKey, AutoIncrement]
        [Column("id")]
        public int Id { get; set; }
        [Column("unit_id")]
        public int UnitId { get; set; }
        [Column("name")]
        public string Name { get; set; }

        public Unit(int unitId, string name)
        {
            UnitId = unitId;
            Name = name;
        }
    }
    ```
    ```C#
    // データベース作成処理の返り値
    SQLiteAsyncConnection connection = new SQLiteAsyncConnection(option);
    // テーブルの作成
    connection.CreateTable<Unit>();
    ```
- データの流し込み
    ```C#
    SQLiteAsyncConnection connection = new SQLiteAsyncConnection(option);
    // データの流し込み
    connection.InsertAsync(new Unit(1, "ユニット1"));
    ```