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
「Walking DJ」は、ユーザーが街を歩いているとその場所に合った音楽をレコメンドしてくれるARアプリです。「“XR PARADE” created with TNXR」で展示したバージョンでは、虎ノ門ヒルズ・ステーションタワー・B2Fアトリウム内におけるユーザーの場所に基づいて音楽をレコメンドします。使用したツールは以下のとおりです。

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
アトリウム内にXY平面を設定し、その上でユーザーの座標を計算します。X軸はEnergy（曲のエネルギッシュさ）、Y軸はDanceability（曲の踊りやすさ）に対応しています。X軸方向ではカフェからエスカレーターに近づくほどEnergyの値が高くなり、Y軸方向では駅からモールに近づくほどDanceabilityの値が高くなります。
![](/images/tnxr-walking-dj/map.png)
*アトリウムにおけるユーザーの座標と音楽特性の対応イメージ*
ユーザーの座標はアプリ画面の上端に表示されます。
![](/images/tnxr-walking-dj/user_flow_1.jpg)
*デモ動画より抜粋（左側の画像は実際のアプリ画面）*
#### 2. Select Trackボタンを押すと、その時のユーザーの座標に基づいて曲が選ばれる
ユーザーの座標に対応する音楽特性（Energy, Danceability）の値に最も近い曲がセレクトされます。セレクトされた曲のタイトルやアーティスト名、そしてジャケットは画面に表示されます。
![](/images/tnxr-walking-dj/user_flow_2.jpg)
*デモ動画より抜粋（左側の画像は実際のアプリ画面）*
#### 3. Play Trackボタンを押すと、選ばれた曲が再生される
![](/images/tnxr-walking-dj/user_flow_3.jpg)
*デモ動画より抜粋（左側の画像は実際のアプリ画面）*
Stop Trackボタンを押すと、曲の再生は止まります。
#### 4. 選ばれた曲は「ARレコード」としてその場に置かれる
Select Trackボタンを押した際に、選ばれた曲の情報が保存されたレコード型の3Dオブジェクト（以下、「ARレコード」）が目の前に配置されます。このARレコードをタップすると、保存されている曲の情報が表示され再生することも可能です。
![](/images/tnxr-walking-dj/user_flow_4.jpg)
*デモ動画より抜粋（左側の画像は実際のアプリ画面）*


# システム構成
以上のようなユーザー体験を実現するために、システムは以下の流れで動作しています。

A. Spotify APIのアクセストークンを取得
B. Immersalを使ってユーザーの座標を取得
C. ユーザーの座標に基づいてSpotify APIから曲の情報を取得し表示
D. unity-webiviewを使って取得した曲を再生
E. 取得した曲の情報を紐づけた3Dオブジェクト（ARレコード）をAR空間に配置

システム構成図は以下のようになっています（こういう書き方で良いのかわかりませんが）。
![](/images/tnxr-walking-dj/system_config.png)
*Walking DJ システム構成図*

以下の章では、システムの各ステップの詳細を説明したいと思います。

# A. Spotify APIのアクセストークンを取得
// 二つのトークン

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
