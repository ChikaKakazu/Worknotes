# Unityでスレッドシートに書いてあるマスタをダウンロードする
- SQLiteのDBにマスタデータを入れ込む

## スプレッドシートのデータを取得する方法
- 対象のデータをもつスプレッドシートを作成する
  - Unitテーブル
  - Id, UnitId, Nameのカラムを持つ

### URL知っていれば取れるバージョン
- スプレッドシートを公開する
  - シートの`ファイル`->`共有`->`ウェブに公開`をクリック
  - 公開するシートを`Unit(シート名)`、ウェブページを`カンマ区切り形式(.csv)`に変更
  - URLをコピーする
- Unity側で`UnityWebRequest`を使ってコピーしたURLに対してリクエストを送る
  - `request.SendWebRequest()`を非同期(await)で呼んでいるが、C#のTaskではできない。UniTaskが必要。もしくはコルーチンでできるはず(多分)
    ```C#
    var url = "コピーしたURL";
    var request = UnityWebRequest.Get(url);
    await request.SendWebRequest();
    // デシリアライズは各々で実装
    var dataArray = CSVSerializer.Deserialize<T>(request.downloadHandler.text);
    ```
    ```C#
    // 取得したデータを入れる
    var connection = "SQLite-netのコネクションを取得"
    // 取得したテーブルのデータをまとめて追加
    await connection.InsertAllAsync(dataArray, false);
    ```

### Google Apiを利用して取得
- 参考
  - https://tanuhack.com/operate-spreadsheet/
  - https://techblog.forgevision.com/entry/2023/05/26/120009
#### Unity側
- UnityのPackage Managerを開き、左上にある+ボタンの`Add package from git URL...`をクリック
- このURLを入力
  - `https://github.com/GlitchEnzo/NuGetForUnity.git?path=/src/NuGetForUnity`
  - NugetがUnityで使えるようになる
- UnityのヘッダーにNugetの項目が増えているはずなので、そこから`Manage NuGet Packages`をクリック
- 検索窓に`Google.Apis.Sheets`と入力しSearch
- `Google.Apis.Sheets.v4`が表示されるのでインストールする
- インストールが完了すると`Assets/Packages`ディレクトリが作られ、その中に`Google.Apis.Sheets.v4`と依存関係のある他のライブラリがインストールされていることを確認できる

#### Google側
- GoogleApiを利用するためにはプロジェクトを作る必要がある
  - [Google Cloud Platform](https://console.cloud.google.com/apis/dashboard?project=ornate-compass-274403)
- プロジェクトの作成
  1. 画面上部の`My First Project`を選択
  ![picture 0](images/9611fd31fb980af2a6cbab0076ca56d7430c20f252046f47f82dafdc5675fc32.png)  
  1. 画面右上の`新しいプロジェクト`をクリック
  ![picture 1](images/534b6ea20c31daab9947ccaeddd38795eb93d30b4bfb27073178f10b69f75759.png)
  1. 適当なプロジェクト名と組織、場所を入力氏、作成
  ![picture 2](images/4c2f9ec22bd9477b488bfb0f7eae1b13b4af5de424ae889335ea3913b7aba59b.png)  
  1. ヘッダーからプロジェクト選択画面で作成したプロジェクトを選択できるようになるので、プロジェクトを切り替える
<!-- - OAuth同意画面でどちらに公開するか設定する。そのまま作成ボタンをクリック
  ![picture 3](images/46537c6a3f67a830055d13af484948f8daef75f9b5a19c0cba3948cf594ed949.png)
- そのまま、適当なアプリ名を入力し、画面下の`保存して次へ`をクリック
  - ※ 他必須項目も適宜入力
  ![picture 4](images/cf6d7bbc8bdb7d8e6284bc090813da96fb261254e76d90dec3bb643c08157bf3.png)   -->
- サービスアカウントを作成する
  - 画面上部の`認証情報を作成`から`サービスアカウント`をクリック
  - 適当なサービスアカウント名を入力し、作成する
  - 作成されたサービスアカウントをクリックし、編集画面に移動する
  - `キー`タブに移動し、鍵を作成する
    ![picture 6](images/22d3e1b22781402a3e376ae72ceb12f73b94a6f41ceaea9579f479055ae8ec31.png)
  - Jsonで作成しDLする
- 使用するAPIを有効にする
  - 画面左のサイドバーから`ライブラリ`をクリック
  - そこから使いたいApiを検索し、有効にする
  ![picture 5](images/8d3852756a7151e340bdb9b432f6207d21de1ce2d0456afeb32301e1461e6570.png)
- スプレッドシートの設定
  - アクセス制限をかける(デフォルトでなっているはず)
    - スプレッドシート右上の`共有`をクリックし、制限がかかっているか確認
  - `共有`モーダルの`ユーザーやグループを追加`の入力欄に、先ほど作成したサービスアカウントのメールアドレスを入力する

#### 呼び出し処理(Unity)
```C#
public class SpreedSheetManager
{
    private static SheetsService s_sheetService = default;
    private readonly static string s_applicationName = "適当な名前(多分)";
    // 対象スプレッドシートのURLの/d/の後ろ
    // https://docs.google.com/spreadsheets/d/この部分/edit?pli=1#gid=0
    private readonly static string s_spreadsheetId = "スプレッドシートID";

    /// <summary>
    /// シートのマスタを取得する
    /// </summary>
    /// <param name="sheetName">シート名</param>
    /// <returns></returns>
    public static List<string[]> GetSheetData(string sheetName)
    {
        if (s_sheetService == null)
        {
            s_sheetService = OpenSheet();
        }

        ValueRange rVR;
        string wRange;
        wRange = string.Format("{0}", sheetName);
        var getRequest = s_sheetService.Spreadsheets.Values.Get(s_spreadsheetId, wRange);
        rVR = getRequest.Execute();

        var result = new List<string[]>();
        foreach (var list in rVR.Values)
        {
            var stringList = new List<string>();
            foreach (var obj in list)
            {
                stringList.Add((string)obj);
                Debug.Log((string)obj);
            }
            result.Add(stringList.ToArray());
        }
        return result;
    }

    private static SheetsService OpenSheet()
    {
        GoogleCredential credential;
        // サービスアカウントで作成しDLしたjsonファイル
        var keyName = "jsonファイル";

        using (var stream = new FileStream(Application.dataPath + "/Editor/" + keyName, FileMode.Open, FileAccess.Read))
        {
            //CredentialファイルがcredPathに保存される
            credential = GoogleCredential.FromStream(stream).CreateScoped(SheetsService.Scope.Spreadsheets);
        }

        // serviceを作成、Requestパラメータを設定
        var service = new SheetsService(new BaseClientService.Initializer()
        {
            HttpClientInitializer = credential,
            ApplicationName = s_applicationName,
        });
        return service;
    }
}
```