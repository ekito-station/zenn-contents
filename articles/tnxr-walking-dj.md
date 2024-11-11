---
title: "Walking DJ 開発メモ"
emoji: "💿"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity"]
published: false
---
# はじめに
2024年10月に虎ノ門ヒルズで開催されたプログラム「TOKYO NODE OPEN LAB 2024 “XR PARADE” created with TNXR」にて、音楽レコメンドARアプリ「Walking DJ」を開発・展示しました。本記事では、このアプリの開発過程をまとめたいと思います。

## Walking DJ 概要
「Walking DJ」は、ユーザーが街を歩いているとその場所に合った楽曲をレコメンドしてくれるARアプリです。「“XR PARADE” created with TNXR」で展示したバージョンでは、虎ノ門ヒルズ・ステーションタワー・B2Fアトリウム内におけるユーザーの場所に基づいて楽曲をレコメンドします。使用したツールは以下のとおりです。

- Unity
- [Spotify API](https://developer.spotify.com/documentation/web-api)
- [Immersal](https://immersal.com/)
- [unity-webview](https://github.com/gree/unity-webview)


### Walking DJ デモ動画
https://www.youtube.com/watch?v=wLX4vHpueWQ

## プログラム概要
### TOKYO NODE OPEN LAB 2024
虎ノ門ヒルズに拠点を持つ都市体験の研究開発チーム「[TOKYO NODE LAB](https://www.tokyonode.jp/lab/)」の開設1周年を記念して開催されたプログラムです。虎ノ門ヒルズ・ステーションタワーの各所にて、展示やトークセッションなどのイベントが展開されました。
#### TOKYO NODE OPEN LAB 2024 ウェブサイト
https://tokyonode.jp/lab/events/openlab2024/index.html
### “XR PARADE” created with TNXR
TOKYO NODE OPEN LAB 2024の一環として、虎ノ門ヒルズ・ステーションタワーのB2Fアトリウムにて開催された展示会です。2024年2月に開催されたハッカソン「[TOKYO NODE “XR HACKATHON” powered by PLATEAU](https://tokyonode.jp/sp/xrhackathon2023)」の参加者を中心に結成された都市XR実装コミュニティ「TNXR」のメンバーが、虎ノ門の街を舞台としたXRコンテンツを展示しました。
#### “XR PARADE” created with TNXR ウェブサイト
https://www.tokyonode.jp/lab/events/openlab2024_xr_parade/index.html

# ユーザー体験
アプリのユーザー体験は以下のような流れになっています。

#### 0. スマホのカメラで周囲の景色を映して、空間をスキャンする
アプリ起動直後は、ユーザーの位置を特定するためにスマホのカメラで周囲の景色を見渡す必要があります（特定まで数秒から十秒ほどかかります）。
![](/images/tnxr-walking-dj/user_flow_0.jpg)
*デモ動画より抜粋（左側の画像は実際のアプリ画面）*
#### 1. スキャンが完了すると、アトリウム内におけるユーザーの座標が計測され始める
アトリウム内にXY平面を設定し、その上でユーザーの座標を計算します。X軸はEnergy（楽曲のエネルギッシュさ）、Y軸はDanceability（楽曲の踊りやすさ）に対応しています。X軸方向ではカフェからエスカレーターに近づくほどEnergyの値が高くなり、Y軸方向では駅からモールに近づくほどDanceabilityの値が高くなります。
![](/images/tnxr-walking-dj/map.png)
*アトリウムにおけるユーザーの座標と音楽特性の対応イメージ*
ユーザーの座標はアプリ画面の上端に表示されます。
![](/images/tnxr-walking-dj/user_flow_1.jpg)
*デモ動画より抜粋（左側の画像は実際のアプリ画面）*
#### 2. Select Trackボタンを押すと、その時のユーザーの座標に基づいて曲が選ばれる
ユーザーの座標に対応する音楽特性（Energy, Danceability）の値に最も近い楽曲がセレクトされます。セレクトされた楽曲のタイトルやアーティスト名、そしてジャケットは画面に表示されます。
![](/images/tnxr-walking-dj/user_flow_2.jpg)
*デモ動画より抜粋（左側の画像は実際のアプリ画面）*
#### 3. Play Trackボタンを押すと、選ばれた曲が再生される
![](/images/tnxr-walking-dj/user_flow_3.jpg)
*デモ動画より抜粋（左側の画像は実際のアプリ画面）*
Stop Trackボタンを押すと、楽曲の再生は止まります。
#### 4. 選ばれた曲は「ARレコード」としてその場に置かれる
Select Trackボタンを押した際に、選ばれた楽曲の情報が保存されたレコード型の3Dオブジェクト（以下、「ARレコード」）が目の前に配置されます。このARレコードをタップすると、保存されている楽曲の情報が表示され再生することも可能です。
![](/images/tnxr-walking-dj/user_flow_4.jpg)
*デモ動画より抜粋（左側の画像は実際のアプリ画面）*


# システム構成
以上のようなユーザー体験を実現するために、システムは以下の流れで動作しています。

A. Spotify APIのアクセストークンを取得
B. Immersalを使ってユーザーの座標を取得
C. ユーザーの座標に基づいてSpotify APIから楽曲情報を取得し表示
D. unity-webiviewを使って取得した楽曲を再生
E. 取得した楽曲の情報を紐づけた3Dオブジェクト（ARレコード）をAR空間に配置

システム構成図は以下のようになっています（こういう書き方で良いのかわかりませんが）。
![](/images/tnxr-walking-dj/system_config.png)
*Walking DJ システム構成図*

以下の章では、システムの各ステップの詳細を説明したいと思います。
説明のために一部実際のコードを変更・簡略化している箇所があります。

# A. Spotify APIのアクセストークンを取得
本アプリでは、楽曲情報の取得や再生にSpotify APIを使用しています。
そのため、まずSpotify APIのアクセストークンを取得します。アクセストークンの取得は以下の2つの方法で行います。

- [Authorization Code Flow](https://developer.spotify.com/documentation/web-api/tutorials/code-flow)（楽曲の再生向け、ログイン必要）
- [Client Credentials Flow](https://developer.spotify.com/documentation/web-api/tutorials/client-credentials-flow)（楽曲情報の取得向け、ログイン不要）

1つ目のAuthorization Code Flowは楽曲情報の取得と再生の両方に対応していますが、Spotifyアカウントへのログインが必要です。この方法のみを使用した場合、ログインができなくなるとすべての機能が使えなくなる恐れがあります。
2つ目のClient Credentials Flowは楽曲情報の取得のみ可能で楽曲の再生には対応していませんが、Spotifyアカウントにログインする必要がありません。
そのため、万が一に備えて2つの方法を併用しています。

![](/images/tnxr-walking-dj/access_token_flow.png)
*アクセストークン取得の流れ*

## Authorization Code Flow
Authorization Code Flowでは、以下の流れでアクセストークンを取得します。

1. アプリから外部ブラウザを起動しSpotifyアカウントのログインページを開く
2. ログインに成功したらSpotify Web APIに認証コードを要求する
3. 認証コードが返ってきたらアプリを再度開く
4. iOSプラグインを用いて認証コードを受け取る
5. 認証コードを用いてSpotify Web APIにアクセストークンを要求する
6. APIからアクセストークンを受け取る

（参考）
https://developer.spotify.com/documentation/web-api/tutorials/code-flow

#### 1. アプリから外部ブラウザを起動しSpotifyアカウントのログインページを開く

あらかじめ[Spotify for Developersサイト](https://developer.spotify.com/)のDashboardで、アプリのClient IDを取得しRedirect URI（例: `myapp://callback/`）を設定する必要があります。

（参考）
https://developer.spotify.com/documentation/web-api/tutorials/getting-started#create-an-app

Client IDやRedirect URIなどから認証用URLを作成し、そのURLを外部ブラウザで開きます。

```csharp:SpotifyManager
string clientId = ""; // Client IDをDashboardから取得
string redirectUri = ""; // Redirect URIをDashboardで設定
string authEndpoint = "https://accounts.spotify.com/authorize";
// 楽曲再生に必要なスコープを指定
string scopes = "user-modify-playback-state  user-read-playback-state user-read-currently-playing streaming";
// 認証用URLを作成
string authUrl = $"{authEndpoint}?response_type=code&client_id={_clientId}&redirect_uri={UnityWebRequest.EscapeURL(redirectUri)}&scope={UnityWebRequest.EscapeURL(scopes)}";
// 認証用URLを外部ブラウザで開く
Application.OpenURL(authUrl);
```

#### 2. ログインに成功したらSpotify Web APIに認証コードを要求する

外部ブラウザ（Safariなど）で開かれたSpotifyアカウントのログインページにおいてユーザがログインに成功したら、Spotify Web APIに認証コードを要求する処理が実行されます。

![](/images/tnxr-walking-dj/login_page.jpg =200x)
*Spotifyアカウントログインページ*

#### 3. 認証コードが返ってきたらアプリを再度開く

Spotify Web APIから認証コードが返ってきたら以前設定したRedirect URIにリダイレクトされます。リダイレクトされた際にアプリが再度開くように、URL Schemeを設定します。Unityの`Project Settings > Player > Other Settings > Supported URL schemes`に、Redirect URIのスキーム（例: `myapp`）を入力します。

![](/images/tnxr-walking-dj/custom_url_scheme.png)
*URL Schemeの設定*

この設定により、ユーザはログイン後アプリに戻ることができます。

![](/images/tnxr-walking-dj/return_to_app.jpg =200x)
*アプリに戻る*

#### 4. iOSプラグインを用いて認証コードを受け取る

また、アプリに戻った際にSpotify Web APIから返ってくるURLをアプリに受け渡すiOSプラグインを作成します。このURLに認証コードが含まれています。

:::details iOSプラグインの内容
```objc:UrlHandlerAppController
#import "UnityAppController.h"

// Unityにメッセージを送る関数を宣言
void UnitySendMessage(const char* obj, const char* method, const char* msg);

// UnityAppControllerを拡張するクラスの宣言
@interface UrlHandlerAppController : UnityAppController
@end

// UrlHandlerAppControllerクラスの実装
@implementation UrlHandlerAppController
// openURLメソッドのオーバーライド
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey, id> *)options {
    // URLをC文字列に変換
    const char *urlString = [[url absoluteString] cStringUsingEncoding:NSUTF8StringEncoding];
    // 変換されたURLをSpotifyManagerに送信
    UnitySendMessage("SpotifyManager", "HandleUrl", urlString);
    return YES;
}
@end

// UnityAppControllerのサブクラスとして登録
IMPL_APP_CONTROLLER_SUBCLASS(UrlHandlerAppController)
```
:::

#### 5. 認証コードを用いてSpotify Web APIにアクセストークンを要求する

アプリに受け渡されたURLから認証コードを抽出します。
```csharp:SpotifyManager
// iOSプラグインから呼び出される関数
public void HandleUrl(string url)
{
    // URLから認証コードを抽出
    string code = GetCodeFromUrl(url);
    // 認証コードを用いてアクセストークンを要求
    StartCoroutine(ExchangeCodeForToken(code));
}
private string GetCodeFromUrl(string url)
{
    var queryParams = System.Web.HttpUtility.ParseQueryString(new System.Uri(url).Query);
    return queryParams["code"];
}
```

抽出された認証コードを用いて、Spotify Web APIにアクセストークンを要求します。このとき[Unity Web Request](https://docs.unity3d.com/ScriptReference/Networking.UnityWebRequest.html)を使います。
:::details Unity Web Requestを使ってアクセストークンを要求する処理の内容
```csharp:SpotifyManager
private string clientId = ""; // Client IDをDashboardから取得
private string clientSecret = ""; // Client SecretをDashboardから取得
private string tokenEndpoint = "https://accounts.spotify.com/api/token";

private IEnumerator ExchangeCodeForToken(string code)
{
    // リクエストの準備
    var request = new UnityWebRequest(tokenEndpoint, "POST");
    var form = new WWWForm();
    form.AddField("grant_type", "authorization_code");
    form.AddField("code", code);
    form.AddField("redirect_uri", redirectUri);
    form.AddField("client_id", _clientId);
    form.AddField("client_secret", _clientSecret);
    request.uploadHandler = new UploadHandlerRaw(form.data);
    request.downloadHandler = new DownloadHandlerBuffer();
    request.SetRequestHeader("Content-Type", "application/x-www-form-urlencoded");

    // リクエストの送信
    yield return request.SendWebRequest();

    if (request.result == UnityWebRequest.Result.Success)
    {
        // アクセストークンの取得に成功した場合の処理
        // 後述
    }
    else
    {
        // アクセストークンの取得に失敗した場合の処理
        Debug.LogError("Error: " + request.error);
    }
}
```
:::

#### 6. APIからアクセストークンを受け取る

Spotify Web APIからJSON形式のレスポンスを受け取り、それを変換してアクセストークンを取り出します。

:::details アクセストークンを受け取る処理の内容
```csharp:SpotifyManager
private string accessTokenForPlay;
private DateTime tokenForPlayExpiryTime;
private string refreshTokenForPlay;

private IEnumerator ExchangeCodeForToken(string code)
{
    // 前略
    if (request.result == UnityWebRequest.Result.Success)
    {
        // アクセストークンの取得に成功した場合の処理
        string responseText = request.downloadHandler.text;
        // JSONをパースしてアクセストークンを取り出す
        var responseJson = JsonUtility.FromJson<TokenResponseForPlay>(responseText);
        accessTokenForPlay = responseJson.access_token; // アクセストークン
        tokenForPlayExpiryTime = DateTime.Now.AddSeconds(responseJson.expires_in); // アクセストークンの有効期限
        refreshTokenForPlay = responseJson.refresh_token; // リフレッシュトークン
    }
}

// APIから受け取ったJSON形式のレスポンスを変換するためのクラス
[System.Serializable]
public class TokenResponseForPlay
{
    public string access_token;
    public string token_type;
    public int expires_in;
    public string refresh_token;
}
```
:::

## Client Credentials Flow

Client Credentials Flowでは、Spotifyアカウントにログインして認証コードを受け取るプロセスがいらないので、Unity Web Requestを使ってアクセストークンを要求して受け取る処理で済みます。

（参考）
https://developer.spotify.com/documentation/web-api/tutorials/client-credentials-flow

:::details アクセストークンを要求して受け取る処理の内容
```csharp:SpotifyManager
private string accessToken;
private DateTime tokenExpiryTime;

private IEnumerator RequestClientCredentialsToken()
{
    // リクエストの準備
    string url = "https://accounts.spotify.com/api/token";
    WWWForm form = new WWWForm();
    form.AddField("grant_type", "client_credentials");
    UnityWebRequest www = UnityWebRequest.Post(url, form);
    string auth = System.Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(clientId + ":" + clientSecret));
    www.SetRequestHeader("Authorization", "Basic " + auth);

    // リクエストの送信
    yield return www.SendWebRequest();

    if (www.result == UnityWebRequest.Result.Success)
    {
        // アクセストークンの取得に成功した場合の処理
        var jsonResponse = www.downloadHandler.text;
        // JSONをパースしてアクセストークンを取り出す
        TokenResponse tokenResponse = JsonUtility.FromJson<TokenResponse>(jsonResponse);
        accessToken = tokenResponse.access_token; // アクセストークン
        tokenExpiryTime = DateTime.Now.AddSeconds(tokenResponse.expires_in); // アクセストークンの有効期限
    }
    else
    {
        // アクセストークンの取得に失敗した場合の処理
        Debug.LogError("Error: " + www.error);
    }
}

// APIから受け取ったJSON形式のレスポンスを変換するためのクラス
[System.Serializable]
public class TokenResponse
{
    public string access_token;
    public string token_type;
    public int expires_in;
}
```
:::

## アクセストークンの更新

アクセストークンには有効期限があります（1時間だったかな）。以前取得したアクセストークンの有効期限が切れた場合、新たなアクセストークンを取得する必要があります。
Client Credentials Flowで取得したアクセストークンの有効期限が切れた場合は、再度アクセストークンを要求する処理を実行すればいいだけです。
ただ、Authorization Code Flowで取得したアクセストークンの有効期限が切れるたびにユーザにログインしてもらうのは面倒です。そこで、最初にアクセストークンを取得した際に同時に取得したリフレッシュトークンを使うことで、ユーザにログインしてもらう必要なくアクセストークンを再取得することができます。

:::details リフレッシュトークンを使ってアクセストークンを再取得する処理の内容
```csharp:SpotifyManager
private IEnumerator RefreshAccessTokenForPlay()
{
    // リクエストの準備
    string url = "https://accounts.spotify.com/api/token";
    WWWForm form = new WWWForm();
    form.AddField("grant_type", "refresh_token");
    form.AddField("refresh_token", refreshTokenForPlay);
    form.AddField("client_id", clientId);
    form.AddField("client_secret", clientSecret);

    using (UnityWebRequest webRequest = UnityWebRequest.Post(url, form))
    {
        // リクエストの送信
        yield return webRequest.SendWebRequest();

        if (webRequest.result == UnityWebRequest.Result.Success)
        {
            // アクセストークンの再取得に失敗した場合の処理
            var jsonResponse = webRequest.downloadHandler.text;
            // JSONをパースしてアクセストークンを取り出す
            var responseData = JsonUtility.FromJson<RefreshTokenResponse>(jsonResponse);
            accessTokenForPlay = responseData.access_token; // アクセストークンの更新
            tokenForPlayExpiryTime = DateTime.Now.AddSeconds(responseData.expires_in); // アクセストークンの有効期限の更新
        }
        else
        {
            // アクセストークンの再取得に失敗した場合の処理
            Debug.LogError("Error refreshing token: " + webRequest.error);
        }
    }
}

// APIから受け取ったJSON形式のレスポンスを変換するためのクラス
[System.Serializable]
public class RefreshTokenResponse
{
    public string access_token;
    public int expires_in;
}
```
:::

# B. Immersalを使ってユーザーの座標を取得
// 精度が悪くなった時の処理

# C. ユーザーの座標に基づいてSpotify APIから曲の情報を取得し表示
// プレイリストの取得もここで説明

# D. unity-webiviewを使って取得した曲を再生

# E. 取得した曲の情報を紐づけた3Dオブジェクト（ARレコード）をAR空間に配置

# その他（UIなど）
// ヘルプやレコードぷかぷか、dotweenとか

# おわりに
// トラブルがなかったようで
// ryuさんなど謝辞
