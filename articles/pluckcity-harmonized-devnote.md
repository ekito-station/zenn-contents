---
title: "Pluck City -harmonized- 開発メモ"
emoji: "🎶"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity", "csharp", "photon", "pun2", "個人開発"]
published: true
---
# はじめに
東京大学制作展2022で、サウンドARアプリ「Pluck City -harmonized-」を展示しました。本記事では、このアプリの開発過程をまとめたいと思います。
## Pluck City -harmonized- 概要
都市空間に張り巡らせたARの弦を弾いて，音を奏でるアプリ。まちの中を移動して通った場所にARのピンを置いていくと、ピンの間を繋ぐように弦が張られる。弦に近づくと音が鳴り、弦の長さに応じて音の高さは変化する。
### 体験の様子（動画）
https://youtu.be/xwWLqKrzqkg
### 作品詳細
https://ekito-station.com/pluck-city/
### GitHub
https://github.com/ekito-station/pluck-city-harmonized-iiiex
# 開発過程
## AR空間の原点をユーザー間で揃える
本アプリは複数ユーザーでの体験を想定しており、あるユーザーが置いたピンは他のユーザーに共有されます。そのためには、複数ユーザー間でAR空間の原点を揃える必要があります。

通常、AR空間の原点はアプリが起動された地点になるため、各ユーザーのAR空間の原点は揃っていません。たとえば、Aさんがアプリを起動し(x, y, z) = (1, 1, 1)の位置にピンを置いたとします。Bさんが別の場所でアプリを起動しAさんが置いたピンを見ようとしても、2人から見たピンの位置は一致しません。なぜなら、2人は別々の場所でアプリを起動したのでAR空間の原点がずれており、(x, y, z) = (1, 1, 1)の位置もずれているからです。

AR空間の原点について、こちらの記事が参考になります。
https://qiita.com/SAyanada9/items/806f202ccfc1ba55619a

そこで、特定の場所に設置された画像をカメラで読み取ったとき、その画像の位置がAR空間の原点になるように設定します。これによって、各ユーザーのAR空間の原点はその画像の位置に揃えられます。
![](/images/pluckcity-harmonized-devnote/image1.jpg =250x)
*展示会場に設置したディスプレイ*
展示会場に設置したディスプレイの下半分に表示されている画像が読み取られると、MakeContentAppearAtによってAR Session Origin（AR空間の原点）が画像の位置に移動します。

この処理を実装する際に、以下の記事を参考にしました。
https://qiita.com/OKsaiyowa/items/597c5db17be40ab3b265
## ユーザーの現在位置にピンを置く
続いて、ユーザーがボタンを押すとユーザーがいる場所にピンが置かれる処理を作ります。
置かれたピンをユーザー間で共有するために、PUN2（Photon Unity Networking 2）というマルチプレイヤーゲーム開発のためのネットワークサービスを導入します。

PUN2の初期設定については、以下の記事を参考にしました。
https://zenn.dev/o8que/books/bdcb9af27bdd7d/viewer/c04ad5

あらかじめピンのプレハブにPhoton ViewとPhoton Transform Viewをアタッチしておき、ボタンが押されるとカメラの目の前にピンがネットワークオブジェクトとしてインスタンス化されます。インスタンス化されたピンはネットワーク上で同期されるため、他のユーザーから見ても同じ位置にピンが現れます。
![](/images/pluckcity-harmonized-devnote/image2.jpg =250x)
*目の前でピンがインスタンス化されたときのアプリ画面*

## 2つ目以降のピンが置かれると弦を張る
ピンが置かれるとそのピンの座標は保存されます。2つ目以降のピンが置かれたときは、1つ前のピンの座標と置かれたばかりのピンの座標を結ぶように弦を張ります。
弦にはUnityのプリミティブオブジェクトのCapsuleを使用しています。弦を張るための処理は以下の通りです。
```csharp
// curPosは置いたばかりのピンの座標でprePosは1つ前のピンの座標
Vector3 strVec = curPos - prePos;           // 弦の方向を取得
float dist = strVec.magnitude;              // 弦の長さを取得
Vector3 strY = new Vector3(0f, dist, 0f);
Vector3 halfStrVec = strVec * 0.5f;
Vector3 centerCoord = prePos + halfStrVec;  // 弦の中点の座標        
// myStrPrefabは弦（Capsule）のプレハブ
GameObject str = Instantiate(myStrPrefab, centerCoord, Quaternion.identity);    // 弦をインスタンス化
// strRadiusは弦（Capsule）の太さ
str.transform.localScale = new Vector3(strRadius, dist / 2, strRadius);         // ひとまず弦をY軸方向に伸ばす

CapsuleCollider col = str.GetComponent<CapsuleCollider>();
col.isTrigger = true;   // 衝突判定を行わないように
str.transform.rotation = Quaternion.FromToRotation(strY, strVec);   // 弦を本来の方向に回転
```

## 他人のピンとの間にも弦を張る
自分がピンを置いたとき近くに他のユーザーの置いたピンがあれば、それとの間にも弦を張ります。ピンにはタグを付けており、自分がピンを置くたびに近くにあるそのタグが付いたゲームオブジェクト（つまりピン）を全取得します。

ピンはネットワークオブジェクトとしてインスタンス化されたので、各ピンには管理者が割り当てられています。ここでは、ピンをインスタンス化したユーザーがピンの管理者となっています。全取得したピンのうち管理者が自分以外のもの、すなわち他のユーザーがインスタンス化したピンを選び、前節と同じようにして自分が置いたピンとの間に弦を張ります。
```csharp
GameObject[] pins = GameObject.FindGameObjectsWithTag("Pin");   // ピンを全取得
foreach (GameObject pin in pins)
{
    PhotonView pinPhotonView = pin.GetComponent<PhotonView>();
    if (!pinPhotonView.IsMine)  // ピンの管理者が自分ではない場合true
    {
        // 弦を張る処理（省略）
    }
}
```
※説明のため実際のコードから一部変更しています。
![](/images/pluckcity-harmonized-devnote/image3.jpg =250x)
*自分が置いたピンとの間には金色の弦、
他のユーザーが置いたピンとの間には銀色の弦が張られる*

## 各弦に音を割り当てる
弦を張るときに、弦の長さに応じてその弦から鳴る音の高さを指定します。今回は最初に張られた弦から鳴る音をC3（ド）とし、2本目以降の弦については純正律を参考にして、最初の弦と長さを比較することで音の高さを指定します。たとえば、2本目の弦の長さが最初の弦の2/3倍ならば、2本目の弦から鳴る音はG3（ソ）になります。

実際のコードでは処理を軽くするために、長さの2乗を比較しています（Vector3.magnitudeよりVector3.sqrMagnitudeのほうがルートの計算をしていないぶん高速に処理できます）。比較した結果指定された音の高さについての値を、弦のインスタンスが持つフィールドに保存します。

ちなみに純正律については、以下の記事が参考になります。
https://inalesson.com/just-intonation/2443/
純正律とは、音の周波数が単純な整数比によって定められた音律です。純正律に基づくと、C3とG3の周波数の比は2:3になります。
弦楽器の音の高さ（周波数）は弦の長さに反比例するため、周波数を3/2倍したい場合は弦の長さを2/3倍する必要があります。

## 弦に近づいたら音が鳴るようにする
そして、張られた弦に近づいたら音が鳴るようにするために、AR CameraにSphere ColliderとRigidbodyをアタッチします。Capsule Colliderがもともとアタッチされている弦に近づいていくと、AR Cameraと弦が当たります（弦のCapsule ColliderのisTriggerをtrueにしたため、両者は当たってもすり抜けます）。

AR CameraにはさらにAudio SourceとAudio Listenerをアタッチし、弦と当たった時にその弦から鳴る音の高さを取得してその高さの音を鳴らす処理を作ります。
```csharp
public AudioSource audioSource;
public AudioClip b3;
public AudioClip a3;
// 略
private void OnTriggerEnter(Collider other)
{
    StringController stringController = other.gameObject.GetComponent<StringController>();  // 弦にアタッチされたスクリプトを取得
    float strPitch = stringController.pitch;    // 弦から鳴る音の高さについての値を取得

    switch (strPitch)
    {
        case 0.36f: // B3（シ）を表す値
            audioSource.PlayOneShot(b3);    // B3を鳴らす
            break;
        case 0.44f; // A3（ラ）を表す値
            audioSource.PlayOneShot(a3);    // A3を鳴らす
            break;
        // 略
    }
}
```

#  おわりに
以上が主な開発過程の振り返りになります。もともと夏に参加したハッカソンで思いついたアイデアでしたが想像以上に評判が良かったので、アイデアを発展させて大学の制作展でも展示することにしました。まだ発展可能性はありそうなので、引き続きアイデアを膨らませていきたいと思います。