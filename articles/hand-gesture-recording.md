---
title: "Meta XR SDKを使って、ハンドジェスチャをレコーディングする"
emoji: "🖐️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity", "vr"]
published: true
---
# はじめに
Meta Quest向けアプリケーションの開発において下図のように、ハンドジェスチャの見本を3Dアニメーションで表現したい場合があるかと思います。そのために、ハンドジェスチャのアニメーションをレコーディングする方法をメモします。
（あくまで自分が取った方法なので、他にもやり方はあると思います。）
![](/images/hand-gesture-recording/image1.gif)

# 1. レコーディング用の手を用意する
まず、Unityプロジェクトに`Meta XR Interaction SDK OVR Samples`をインストールします。
https://assetstore.unity.com/packages/tools/integration/meta-xr-interaction-sdk-ovr-samples-268521
このとき、`Samples > Example Scenes`もインポートします。
![](/images/hand-gesture-recording/image2.png =400x)
次に、シーンを新規作成します。ここで、`GestureExamples`というサンプルシーン（`Assets/Samples/Meta XR Interaction SDK OVR Samples/59.0.0/Example Scenes/GestureExamples.unity`）から`OVRCameraRig`をコピーし、作成したシーンにペーストします。

この`OVRCameraRig`の配下に、手のオブジェクトが含まれています。

# 2. レコーディングする
レコーディングにはUnity Recorderを使用します。
https://soredemo-apparel.net/vfx-and-3dcg/howtouse_unity_recorder
Unity RecorderをインストールしRecorder Windowを表示させたら、レコーディングの設定をします。
![](/images/hand-gesture-recording/image3.png =400x)
*Recorder Window*
`+ Add Recorder > Animation Clip`をクリックしてから、右下のエリアの`Input > GameObject`に手のオブジェクト（左手の場合`OVRCameraRig > OVRInteraction > OVRHands > LeftHand > HandVisualsLeft > OVRLeftHandVisual > OculusHand_L`）をドラッグ＆ドロップします。
![](/images/hand-gesture-recording/image4.png =400x)
*OculusHand_L / OculusHand_Rの場所*
`Recorded Components`は`Transform`を選択し、`Record Hierarchy`にはチェックを入れます。これによって手のオブジェクトの`Transform`と、手のオブジェクトの子オブジェクト（すなわち指のオブジェクト）の`Transform`をレコーディングできるようになります。

両手を同時にレコーディングしたい場合は再度`+ Add Recorder > Animation Clip`をクリックしてから、`Input > GameObject`にもう片方の手のオブジェクトをドラッグ＆ドロップします。

設定が完了したら`START RECORDING`を押して、レコーディングを開始します。`STOP RECORDING`を押すと、レコーディングを終了できます。これによって、指定したフォルダ内にAnimation Clipが作成されます。

# 3. レコーディングしたAnimation Clipを再生する
Animation Clipを再生するための手のオブジェクトを用意します。
`OVRCameraRig`配下の手のオブジェクト（左手の場合`OVRCameraRig > OVRInteraction > OVRHands > LeftHand > HandVisualsLeft > OVRLeftHandVisual > OculusHand_L`）を複製して、`OVRCameraRig`の外側に出します。
![](/images/hand-gesture-recording/image5.png =400x)
この手のオブジェクトに、レコーディングで作成したAnimation Clipをドラッグ＆ドロップします。これによって手のオブジェクトが、レコーディングされたアニメーションを再生できるようになります。

# おわりに
以上、ハンドジェスチャのアニメーションをレコーディングして再生するまでの方法をまとめました。
詳しい手順の記述を省いた箇所も多々あるので、分かりにくい箇所はコメントで指摘していただけると幸いです。