---
title: "Walking DJ 開発記録（TOKYO NODE OPEN LAB 2024 “XR PARADE”）"
emoji: "💿"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity", "csharp", "api", "ios", "javascript"]
published: true
---
# はじめに
2024年10月に虎ノ門ヒルズで開催されたプログラム「[TOKYO NODE OPEN LAB 2024 “XR PARADE” created with TNXR](https://www.tokyonode.jp/lab/events/openlab2024_xr_parade/index.html)」にて、音楽レコメンドARアプリ「Walking DJ」を開発・展示しました。本記事では、このアプリの開発過程をまとめたいと思います。特に、UnityとSpotify APIの連携について詳しめに記録するので、UnityでSpotify APIを活用したい人の参考になれば幸いです。

## Walking DJ 概要
「Walking DJ」は、ユーザーが街を歩いているとその場所に合った楽曲をレコメンドしてくれるARアプリです。「“XR PARADE” created with TNXR」で展示したバージョンでは、虎ノ門ヒルズ・ステーションタワー・B2Fアトリウム内におけるユーザーの場所に基づいて楽曲をレコメンドします。

https://x.com/tokyonodelab/status/1843899202935644619

使用したツールは主に以下のとおりです。

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
Walking DJのユーザー体験は以下のような流れになっています。

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
B. VPSを使ってユーザーの座標を取得
C. ユーザーの座標に基づいてSpotify APIから楽曲情報を取得し表示
D. unity-webiviewを使って楽曲を再生
E. 楽曲の情報を紐づけた3Dオブジェクト（ARレコード）をAR空間に配置

システム構成図は以下のようになっています（こういう書き方で良いのかわかりませんが）。
![](/images/tnxr-walking-dj/system_config.png)
*Walking DJ システム構成図*

以下の章では、システムの各ステップの詳細を説明したいと思います。
説明のために一部、実際のコードから変更・簡略化している箇所があります。

# A. Spotify APIのアクセストークンを取得
本アプリでは、楽曲情報の取得や再生にSpotify APIを使用しています。
そのため、まずSpotify APIのアクセストークンを取得します。アクセストークンの取得は以下の2つの方法で行います。

- [Authorization Code Flow](https://developer.spotify.com/documentation/web-api/tutorials/code-flow)（楽曲の再生向け、ログイン必要）
- [Client Credentials Flow](https://developer.spotify.com/documentation/web-api/tutorials/client-credentials-flow)（楽曲情報の取得向け、ログイン不要）

1つ目のAuthorization Code Flowは楽曲情報の取得と再生の両方に対応していますが、Spotifyアカウントへのログインが必要です。この方法のみを使用した場合、ログインできなくなるとすべての機能が使えなくなる恐れがあります。
2つ目のClient Credentials Flowは楽曲情報の取得のみ可能で楽曲の再生には対応していませんが、Spotifyアカウントにログインする必要がありません。
そのため、トラブルに備えて2つの方法を併用しています。

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

https://developer.spotify.com/documentation/web-api/tutorials/code-flow

#### 1. アプリから外部ブラウザを起動しSpotifyアカウントのログインページを開く

あらかじめ[Spotify for Developersサイト](https://developer.spotify.com/)のDashboardで、アプリのClient IDを取得しRedirect URI（例: `myapp://callback/`）を設定する必要があります。

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

外部ブラウザ（Safariなど）で開かれたSpotifyアカウントのログインページにおいてユーザーがログインに成功したら、Spotify Web APIに認証コードを要求する処理が実行されます。（ただし、楽曲を再生するためのアクセストークンを取得するには、Premiumアカウントにログインする必要があります。）

![](/images/tnxr-walking-dj/login_page.jpg =200x)
*Spotifyアカウントログインページ*

:::message
アプリがdevelopment modeだったため、あらかじめSpotify for DevelopersサイトのDashboardのUser Managementで、ログインするユーザーのEmailアドレスを登録する必要がありました。ここに登録していないと、ログインに成功してもアクセストークンが取得できません。
![](/images/tnxr-walking-dj/user_management.png)
*DashboardのUser Management*
:::

#### 3. 認証コードが返ってきたらアプリを再度開く

Spotify Web APIから認証コードが返ってきたら、以前設定したRedirect URIにリダイレクトされます。リダイレクトされた際にアプリが再度開くように、URL Schemeを設定します。Unityの`Project Settings > Player > Other Settings > Supported URL schemes`に、Redirect URIのスキーム（例: `myapp`）を入力します。

![](/images/tnxr-walking-dj/custom_url_scheme.png)
*URL Schemeの設定*

この設定により、ユーザーはログイン後アプリに戻ることができます。

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
    UnityWebRequest request = new UnityWebRequest(tokenEndpoint, "POST");
    var form = new WWWForm();
    form.AddField("grant_type", "authorization_code");
    form.AddField("code", code);
    form.AddField("redirect_uri", redirectUri);
    form.AddField("client_id", clientId);
    form.AddField("client_secret", clientSecret);
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
    UnityWebRequest request = UnityWebRequest.Post(url, form);
    string auth = System.Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(clientId + ":" + clientSecret));
    request.SetRequestHeader("Authorization", "Basic " + auth);

    // リクエストの送信
    yield return request.SendWebRequest();

    if (request.result == UnityWebRequest.Result.Success)
    {
        // アクセストークンの取得に成功した場合の処理
        var jsonResponse = request.downloadHandler.text;
        // JSONをパースしてアクセストークンを取り出す
        TokenResponse tokenResponse = JsonUtility.FromJson<TokenResponse>(jsonResponse);
        accessToken = tokenResponse.access_token; // アクセストークン
        tokenExpiryTime = DateTime.Now.AddSeconds(tokenResponse.expires_in); // アクセストークンの有効期限
    }
    else
    {
        // アクセストークンの取得に失敗した場合の処理
        Debug.LogError("Error: " + request.error);
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
ただ、Authorization Code Flowで取得したアクセストークンの有効期限が切れるたびにユーザーにログインしてもらうのは面倒です。そこで、最初にアクセストークンを取得した際に同時に取得したリフレッシュトークンを使うことで、ユーザーにログインしてもらう必要なくアクセストークンを再取得することができます。

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
    UnityWebRequest request = UnityWebRequest.Post(url, form);

    // リクエストの送信
    yield return request.SendWebRequest();

    if (request.result == UnityWebRequest.Result.Success)
    {
        // アクセストークンの再取得に成功した場合の処理
        var jsonResponse = request.downloadHandler.text;
        // JSONをパースしてアクセストークンを取り出す
        var responseData = JsonUtility.FromJson<RefreshTokenResponse>(jsonResponse);
        accessTokenForPlay = responseData.access_token; // アクセストークンの更新
        tokenForPlayExpiryTime = DateTime.Now.AddSeconds(responseData.expires_in); // アクセストークンの有効期限の更新
    }
    else
    {
        // アクセストークンの再取得に失敗した場合の処理
        Debug.LogError("Error refreshing token: " + request.error);
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

# B. VPSを使ってユーザーの座標を取得

次に、アトリウムにおけるユーザーの座標を取得します。

![](/images/tnxr-walking-dj/get_position_flow.png)
*ユーザーの座標を取得して表示する流れ*

ユーザーの座標取得には、都市XR実装コミュニティ「TNXR」内部で提供されるToranomon SDKのVPS機能を使用しています。このVPS機能には[Immersal](https://immersal.com/)が使われています。

ユーザーがデバイスのカメラで周囲の物理空間をスキャンし、一度ローカライズに成功するとAR空間にグリッドの描かれた平面が配置されるようにします。この平面上でユーザーの座標を計算します。
また、平面が配置されたタイミングでその平面を複製して、VPS機能からは独立して配置しておきます。もしVPSのローカライズが不安定になったら元の平面の位置がズレてユーザーの座標を正しく計算できなくなります。その際は、複製された平面を使ってユーザーの座標を計算するようにします。

![](/images/tnxr-walking-dj/map.png)
*アトリウムにおけるユーザーの座標と音楽特性の対応イメージ（再掲）*

ユーザーの位置に相当する`MainCamera`の座標を定期的に取得しそれを、平面を基準としたローカル座標に変換してから正規化しUIに表示させます。正規化することで、次のステップにおいてユーザーの座標と楽曲の音楽特性（Energy, Danceability）を対応させやすくします。

:::details ユーザーの座標を取得する処理の内容
```csharp:PlayerController
public Camera mainCamera; // MainCamera
public GameObject plane; // 平面
public float planeSizeX; // 平面の幅
public float planeSizeZ; // 平面の奥行き

private Vector2 UpdatePlayerPositionOnPlane()
{    
    // MainCameraのワールド座標を平面のローカル座標に変換
    Vector3 localPosition = plane.transform.InverseTransformPoint(mainCamera.transform.position);
    // 平面のサイズに基づいて座標を正規化
    Vector2 normalizedPosition = new Vector2(
        Mathf.InverseLerp(- planeSizeX / 2, planeSizeX / 2, localPosition.x),
        Mathf.InverseLerp(- planeSizeZ / 2, planeSizeZ / 2, localPosition.z)
    );

    return normalizedPosition;
}
```
:::

# C. ユーザーの座標に基づいてSpotify APIから曲の情報を取得し表示

Spotify APIのアクセストークンとユーザーの座標の両方を取得できたら、続いてユーザーの座標に基づいてSpotify APIから楽曲を取得します。

![](/images/tnxr-walking-dj/get_song_flow.png)
*ユーザーの座標に基づいて曲の情報を取得し表示する流れ*

ここで行う処理は、大きく分けて以下の2つです。

1. アクセストークン取得後にSpotify Web APIから楽曲のプレイリストと各楽曲の音楽特性を取得する
2. Select Trackボタンが押された時のユーザーの座標と各楽曲の音楽特性を比較し最も近い楽曲を選ぶ

#### 1. アクセストークン取得後にSpotify Web APIから楽曲のプレイリストと各楽曲の音楽特性を取得する

アクセストークンが取得できたタイミング（通常はアプリ起動直後）で、Spotify Web APIから楽曲の情報を集めます。
今回は、以下のSpotify公式のプレイリスト4つから楽曲を25曲ずつ、計100曲の情報を集めました（TOKYOという名のつくプレイリストをピックアップしました）。

- [Tokyo Super Hits!](https://open.spotify.com/playlist/37i9dQZF1DXafb0IuPwJyF)（日本の最新ヒットソング特集）
- [Tokyo Rising](https://open.spotify.com/playlist/37i9dQZF1DWX9u2doQ8Q2L)（日本の若手アーティスト特集）
- [Road Trip To Tokyo](https://open.spotify.com/playlist/37i9dQZF1DWV8IND7NkP2W)（インスト楽曲特集）
- [Tokyo City Pop 記憶の記録LIBRARY](https://open.spotify.com/playlist/37i9dQZF1DX4CovPIZya4z)（シティポップ特集）

各プレイリストにはIDが付与されていて、URLの末尾に記載されています（例: [Tokyo Super Hits!](https://open.spotify.com/playlist/37i9dQZF1DXafb0IuPwJyF)のURLは`https://open.spotify.com/playlist/37i9dQZF1DXafb0IuPwJyF`なのでIDは`37i9dQZF1DXafb0IuPwJyF`）。
また各楽曲にもIDが付与されていて、URLの末尾に記載されています（例: [東京砂漠](https://open.spotify.com/intl-ja/track/04WsSAv2YHpDA4ft52wcTm)のURLは`http://open.spotify.com/track/04WsSAv2YHpDA4ft52wcTm`なのでIDは`04WsSAv2YHpDA4ft52wcTm`）。
Unity Web Requestを使って、各プレイリストからそこに含まれる楽曲のIDを取得して、それらのIDをまとめたリストを作成します。

:::details Spotifyのプレイリストから楽曲のIDを集める処理の内容
`playlistId`はプレイリストのID、`limit`はプレイリストから取得したい楽曲の数（今回は25）、`trackIds`は取得した楽曲のIDを追加していくリストです。
```csharp:SpotifyManager
private IEnumerator GetTracksFromPlaylist(string playlistId, int limit, List<string> trackIds)
{
    // リクエストの準備
    string url = $"https://api.spotify.com/v1/playlists/{playlistId}/tracks?limit={limit}";
    UnityWebRequest request = UnityWebRequest.Get(url);
    request.SetRequestHeader("Authorization", "Bearer " + accessToken);

    // リクエストの送信
    yield return request.SendWebRequest();

    if (request.result == UnityWebRequest.Result.Success)
    {
        // プレイリストの取得に成功した場合の処理
        var jsonResponse = request.downloadHandler.text;
        // JSONをパースして楽曲のIDを取り出す
        PlaylistTracks playlistTracks = JsonUtility.FromJson<PlaylistTracks>(jsonResponse);
        foreach (var item in playlistTracks.items)
        {
            // 各楽曲のIDをリストに追加する
            trackIds.Add(item.track.id);
        }
    }
    else
    {
        // プレイリストの取得に失敗した場合の処理
        Debug.LogError("Error: " + request.error);
    }
}

// APIから受け取ったJSON形式のレスポンスを変換するためのクラス
[System.Serializable]
public class PlaylistTracks
{
    public PlaylistTrackItem[] items;
}
[System.Serializable]
public class PlaylistTrackItem
{
    public Track track;
}
[System.Serializable]
public class Track
{
    public string id;
    public string name;
    public Artist[] artists;
    public Album album;
}
```
:::

続いて、各楽曲の音楽特性を取得します。Spotifyは各楽曲の音楽特性（Audio Features）を数値化しています。音楽特性には、Walking DJで使用しているEnergyやDanceabilityのほかにもValence（ポジティブさ）などがあり、それぞれ0から1の間で数値化されています（例: [Bling-Bang-Bang-Born](https://open.spotify.com/intl-ja/track/0kdqcbwei4MDWFEX5f33yG)のEnergyは0.822、Danceabilityは0.853、Valenceは0.746）。

https://developer.spotify.com/documentation/web-api/reference/get-audio-features

Unity Web Requestを使って、先ほどIDを取得した楽曲の音楽特性を取得します。

:::details 各楽曲の音楽特性を取得する処理の内容
```csharp:SpotifyManager
public IEnumerator GetAudioFeatures(List<string> trackIds)
{    
    // リクエストの準備
    string ids = string.Join(",", trackIds);
    string url = $"https://api.spotify.com/v1/audio-features?ids={ids}";
    UnityWebRequest request = UnityWebRequest.Get(url);
    request.SetRequestHeader("Authorization", "Bearer " + accessToken);

    // リクエストの送信
    yield return request.SendWebRequest();

    if (request.result == UnityWebRequest.Result.Success)
    {
        // 音楽特性の取得に成功した場合の処理
        var jsonResponse = request.downloadHandler.text;
        // JSONをパースして音楽特性を取り出す
        AudioFeaturesResponse audioFeaturesResponse = JsonUtility.FromJson<AudioFeaturesResponse>(jsonResponse);
        List<AudioFeatures> audioFeaturesList = new List<AudioFeatures>(audioFeaturesResponse.audio_features);
    }
    else
    {
        // 音楽特性の取得に失敗した場合の処理
        Debug.LogError("Error: " + request.error);
    }
}

// APIから受け取ったJSON形式のレスポンスを変換するためのクラス
[System.Serializable]
public class AudioFeaturesResponse
{
    public AudioFeatures[] audio_features;
}
[System.Serializable]
public class AudioFeatures
{
    public float danceability;
    public float energy;
    public float loudness;
    public float speechiness;
    public float acousticness;
    public float instrumentalness;
    public float liveness;
    public float valence;
    public float tempo;
    public string id;
}
```
:::

#### 2. Select Trackボタンが押された時のユーザーの座標と各楽曲の音楽特性を比較し最も近い楽曲を選ぶ

ユーザーがアプリのSelect Trackボタンを押したら、その時のユーザーの座標と先述のステップで取得した各楽曲の音楽特性を比較します。ユーザーのX座標と各楽曲のEnergyの値の差、ユーザーのY座標と各楽曲のDanceabilityの値の差を計算し、それらが最も小さい楽曲、すなわちユーザーの座標と音楽特性（Energy, Danceability）が最も近い楽曲をピックアップします。

:::details 最も近い楽曲を選ぶ処理の内容
```csharp:SpotifyManager
private AudioFeatures selectedTrack;
public float distanceThreshold;

public void FindClosestTrack(float userPositionX, float userPositionY)
{
    AudioFeatures closestTrack = null;
    float closestDistance = float.MaxValue;

    foreach (var track in audioFeaturesList)
    {
        float energyDistance = Mathf.Abs(track.energy - userPositionX);
        float danceabilityDistance = Mathf.Abs(track.danceability - userPositionY);
        float distance = energyDistance + danceabilityDistance;
        if (distance < closestDistance)
        {
            closestDistance = distance;
            closestTrack = track;
            if (energyDistance < distanceThreshold && danceabilityDistance < distanceThreshold)
            {
                // ユーザーの座標と一定水準より近かったら、その後の楽曲との比較はスキップする
                break;
            }
        }
    }
    selectedTrack = closestTrack;
}
```
:::

選ばれた楽曲のIDから、Unity Web Requestを使ってその楽曲の情報（曲名、アーティスト名、ジャケット画像）を取得します。

:::details 選ばれた楽曲の情報を取得する処理の内容
```csharp:SpotifyManager
public IEnumerator GetTrackInfo(string trackId)
{   
    // リクエストの準備 
    string url = $"https://api.spotify.com/v1/tracks/{trackId}";
    UnityWebRequest request = UnityWebRequest.Get(url);
    request.SetRequestHeader("Authorization", "Bearer " + accessToken);

    // リクエストの送信
    yield return request.SendWebRequest();

    if (request.result == UnityWebRequest.Result.Success)
    {
        // 楽曲情報の取得に成功した場合の処理
        var jsonResponse = request.downloadHandler.text;
        // JSONをパースして楽曲情報を取り出す
        Track track = JsonUtility.FromJson<Track>(jsonResponse);
    }
    else
    {
        // 楽曲情報の取得に失敗した場合の処理
        Debug.LogError("Error: " + request.error);
    }
}

// APIから受け取ったJSON形式のレスポンスを変換するためのクラス
[System.Serializable]
public class Track
{
    public string id;
    public string name;
    public Artist[] artists;
    public Album album;
}
[System.Serializable]
public class Artist
{
    public string name;
}
[System.Serializable]
public class Album
{
    public Image[] images;
}
[System.Serializable]
public class Image
{
    public string url;
}
```
:::

そして、取得した楽曲の情報をアプリ画面に表示します。

![](/images/tnxr-walking-dj/display_track_info.jpg =200x)
*選ばれた楽曲の情報を表示するアプリ画面*

# D. unity-webiviewを使って楽曲を再生

楽曲が選ばれた後、ユーザーがPlay Trackボタンを押したらその楽曲が再生されるようにします。

![](/images/tnxr-walking-dj/play_track_flow.png)
*楽曲を再生する流れ*

具体的な仕組みとしては、[unity-webview](https://github.com/gree/unity-webview)を使ってアプリ内でhtmlファイルを開き、そのhtmlファイル内で[Spotify Web Playback SDK](https://developer.spotify.com/documentation/web-playback-sdk)というJavaScriptライブラリを使って楽曲を再生します。

:::message
Unityアプリ内でhtmlファイルを開けるアセットとして[UniWebView](https://docs.uniwebview.com/)もありますが、こちらを使ってhtmlファイルを開いたところ楽曲は再生されませんでした（原因未解明）。
:::

まず、unity-webviewを使ってWeb Viewを実装しアプリ内でhtmlファイルを開けるようにします。

:::details htmlファイルを開く処理の内容
```csharp:WebViewController
private WebViewObject webViewObject;

public void OpenWebView()
{
    webViewObject = (new GameObject("WebViewObject")).AddComponent<WebViewObject>();
    webViewObject.Init(
        // Webページが読み込まれた時の処理
        ld: (msg) =>
        {
            Debug.Log(string.Format("CallOnLoaded[{0}]", msg));
        },
        enableWKWebView: true);
#if UNITY_EDITOR_OSX || UNITY_STANDALONE_OSX
    webViewObject.bitmapRefreshCycle = 1;
#endif
    // webページ自体は非表示に
    webViewObject.SetVisibility(false);
    // ローカルのhtmlファイルをロード
    string filePath = System.IO.Path.Combine(Application.streamingAssetsPath, "spotify_player.html");
    webViewObject.LoadURL("file://" + filePath.Replace(" ", "%20"));
}
```
:::

以下のようなhtmlファイルをStreamingAssetsフォルダ内に配置し、このローカルhtmlファイルをWeb Viewで開きます。開くのに時間がかかるので、実際にはアプリの起動時に開くようにしています。

::: details 楽曲を再生するためのhtmlファイルの内容
```html:spotify_player.html
<!DOCTYPE html>
<html>
    <head>
        <title>Spotify Player</title>
    </head>
    <body>
        <script src="https://sdk.scdn.co/spotify-player.js"></script>
        <script>
            var token = '';
            var trackId = '';
            var player = null;
            
            // console.logをUnityに送信する
            window.console.log = function(...args) {
                const message = args.map(arg => {
                    if (typeof arg === 'object') {
                        try {
                            return JSON.stringify(arg);
                        } catch(e) {
                            return '[Object]';
                        }
                    }
                    return arg;
                }).join(' ');
                Unity.call(message);
            }

            // WebViewControllerからtokenとtrackUriを受け取る
            function receiveData(newToken, newTrackId) {
                token = newToken;
                trackId = newTrackId;
                console.log('Token: ', token, 'Track Id: ', trackId);
                
                // 既存のプレイヤーが存在する場合は、まず削除する
                if (player) {
                    console.log('Destroying existing player instance.');
                    player.disconnect();
                    player = null;
                }
                
                // 新しいプレイヤーの初期化
                player = new Spotify.Player({
                    name: 'Web Playback SDK Player',
                    getOAuthToken: cb => { cb(token); },
                    volume: 1
                });
                
                // プレイヤーのリスナーを追加してトラックを再生する
                player.addListener('ready', ({ device_id }) => {
                    console.log('Ready with Device ID', device_id);
                    var trackUri = 'spotify:track:' + trackId;
                    playTrack(device_id, trackUri);
                });

                // エラーハンドリング
                player.addListener('not_ready', ({ device_id }) => {
                    console.log('Device ID has gone offline', device_id);
                });

                player.addListener('initialization_error', ({ message }) => {
                    console.error(message);
                });

                player.addListener('authentication_error', ({ message }) => {
                    console.error(message);
                });

                player.addListener('account_error', ({ message }) => {
                    console.error(message);
                });

                player.connect();
            }
            
            window.onSpotifyWebPlaybackSDKReady = () => {
                console.log('Spotify Web Playback SDK is ready');
            };

            // 楽曲を再生する関数
            function playTrack(deviceId, trackUri) {
                fetch(`https://api.spotify.com/v1/me/player/play?device_id=${deviceId}`, {
                    method: 'PUT',
                    body: JSON.stringify({ uris: [trackUri] }),
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': `Bearer ${token}`
                    },
                }).then(response => {
                    if (response.ok) {
                        console.log('Track is playing!');
                    } else {
                        console.error('Error playing track', response.status, response.statusText);
                    }
                }).catch(error => {
                    console.error('Error: ', error);
                });
            }
            
            // 楽曲の再生を停止する関数
            function stopTrack() {                
                if (player) {
                    player.pause().then(() => {
                        console.log('Track has been paused.');
                        player.disconnect();
                        player = null;
                    }).catch(error => {
                        console.log('Error pausing track: ', error);
                    });
                } else {
                    console.error('No player instance found.');
                }
            }
        </script>
    </body>
</html>
```
:::

Play Trackボタンが押されたら、アクセストークン（Authorization Code Flowで取得されたほう）と楽曲IDを渡して、htmlファイル内の楽曲再生用の関数を実行します。同様に、Stop Trackボタンが押されたらhtmlファイル内の楽曲再生停止用の関数を実行します。

```csharp:WebViewController
// Web View上で楽曲を再生する
public void PlayTrackOnWebView(string token, string trackId)
{
    // htmlファイル内のJavaScript関数を呼び出す
    webViewObject.EvaluateJS($"receiveData('{token}', '{trackId}');");
}

// Web View上の楽曲の再生を止める
public void StopTrackOnWebView()
{
    // htmlファイル内のJavaScript関数を呼び出す
    webViewObject.EvaluateJS("stopTrack();");
}
```

# E. 楽曲の情報を紐づけた3Dオブジェクト（ARレコード）をAR空間に配置

Select Trackボタンが押されたとき、選ばれた楽曲の情報をアプリ画面に表示するとともに、その情報と紐づいたレコード型の3Dオブジェクト（ARレコード）をユーザーの目の前に配置します（ARレコードのモデルはBlenderで制作しました）。

![](/images/tnxr-walking-dj/display_track_info.jpg =200x)
*楽曲情報の表示とともに配置されるARレコード*

Select Trackボタンが押されるたびにその場にARレコードをInstantiateし、そのとき選ばれた楽曲をそのARレコードに"書き込み"ます（ARレコードにアタッチしたスクリプトに楽曲のIDを渡します）。
以前配置されたARレコードをタップすると、そこに書き込まれている楽曲の情報が表示される仕組みになっています（ARレコードにアタッチされたスクリプトから楽曲のIDを取り出し、先述の仕組みと同様にしてUnity Web Requestを使ってその楽曲の情報を取得します）。
Play Trackボタンを押せば先述の仕組みで、表示されている楽曲が再生されます。

# UIデザイン

最後にUIデザイン面で工夫した点をいくつか記録したいと思います。

### ヘルプ画面

アプリの右下に常時表示されている「？」ボタンを押すと、ヘルプ画面が表示されます。
❶におけるグリッド上の黒い点は、ユーザーがそのとき実際にいる座標を示しています。また、❷と❸のボタンや❹のARレコードはタップできます。
Canvas上でARレコードの3Dオブジェクトを表示させるために、[RenderTexture](https://docs.unity3d.com/ScriptReference/RenderTexture.html)を使用しています。

![](/images/tnxr-walking-dj/help.jpg =200x)
*ヘルプ画面*

https://youtu.be/xX6Log7jtRs

### UIアニメーション

以下のUIのアニメーションに[DOTween](https://assetstore.unity.com/packages/tools/animation/dotween-hotween-v2-27676)を使用しています。

#### 1. アプリ起動時にスマホを動かして空間をスキャンするよう指示するアニメーション

アプリ起動直後に画面右上に表示されるスマホアイコンには、`DOLocalPath`でパスを使った曲線アニメーションを設定しています。

https://youtu.be/nw43Pi4Upx8

#### 2. 楽曲再生時のアニメーション

楽曲再生時にジャケット画像の上に表示される水玉アイコンには、`DOScale`などで現れたり消えたりするアニメーションを設定しています。

https://youtu.be/qNxhklf2TS0

### ARレコード 

体験していただいた方から、「ARレコードが何を意味するのか初見ではわからなかった」という声をいただき、改善の余地があると感じました。
一方で、「ぷかぷか浮かんでいたから自然とタップしたくなった」という声もいただき、そのアニメーションをつけて良かったと思いました。

また、タップされたARレコードには緑色の輪をつけることで、現在表示されている楽曲がどのARレコードに紐づいているのか視覚的にわかりやすくなるよう工夫しました。さらに楽曲再生時には、Particle Systemを使ってARレコードから音符が湧き出すようにしました。

https://youtu.be/8ir5xJRdct4

# おわりに

以上、Walking DJの開発過程についてまとめました。
企画から開発まで一人で行ったりUnityとWeb APIの連携自体が初めてだったりかなりチャレンジングでしたが、展示期間中アプリは無事に動いていたようなのでホッとしています。

海外から企画書を作ったりアプリを開発したりしていたなか、TNXRのメンバーの方々や現地テストの手伝いをしていただいた[ryu](https://x.com/arukana0813)さんなど多くの人にサポートしていただきました。ありがとうございました！

https://x.com/tokyonodelab/status/1843821354438799413

https://x.com/tokyonodelab/status/1854513160520397255
