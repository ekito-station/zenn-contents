---
title: "Meta XR SDKを使って、動きを伴うハンドジェスチャを認識する"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity", "vr"]
published: true
---
# はじめに
Meta XR SDKを使って、動きを伴うハンドジェスチャを認識する（つまり、手の向きや形に加えて動きを同時に認識する）方法をメモします。

この記事では具体例として、赤の手話を認識する実装を作りたいと思います。

https://hs84.blog.jp/archives/383318.html

赤の手話は以下の要素に分解できるので、それぞれの要素を認識する実装を作っていきたいと思います。
- （手の形）右手の人差し指だけ伸ばしてて、残りの指は閉じている
- （手の向き）右手の手のひらを自分の顔に向ける
- （手の動き）右手を左から右に移動させる

![](/images/hand-gesture-with-movement/movie1.gif)

# 1. 手を用意する
まず、Unityプロジェクトに`Meta XR Interaction SDK OVR Samples`をインストールします。

https://assetstore.unity.com/packages/tools/integration/meta-xr-interaction-sdk-ovr-samples-268521

このとき、`Samples > Example Scenes`もインポートします。

次に、シーンを新規作成します。ここで、GestureExamplesというサンプルシーン（`Assets/Samples/Meta XR Interaction SDK OVR Samples/59.0.0/Example Scenes/GestureExamples.unity`）から`OVRCameraRig`をコピーし、作成したシーンにペーストします。

この`OVRCameraRig`の配下に、手のオブジェクトが含まれています。

# 2. ハンドジェスチャを認識するためのオブジェクトを作成する
空のゲームオブジェクトを作成し、名前をRedGestureとします。
RedGestureの下に、新たに空のゲームオブジェクトを3つ作成し、名前をそれぞれ以下の通りにします。

- RedShape（手の形の認識用）
- RedDirection（手の向きの認識用）
- RedMovement（手の動きの認識用）

![](/images/hand-gesture-with-movement/image1.png)

# 3. 手の形を認識する（RedShape）
まず、手の形を設定するためのファイルを作成します。

Projectウィンドウで右クリックし、`Create > Oculus > Interaction > SDK > Pose Detection > Shape`をクリックして、`Shape Recognizer`を新規作成します。名前をRedShapeRecognizerとします。

ここで赤の手話をするときの手の形（人差し指だけ伸ばしてて、残りの指は閉じている）を設定します。具体的には、人差し指（Index Feature Configs）は

- Curl: `Is` `Open`
- Abduction: `Is` `Open`

と設定します。
Curlは指先の2つの関節がどの程度曲がっているかを表し、CurlがIs Openなら指が完全に伸びた状態を表しています。
Abductionは隣の指から離れているかを表し、AbductionがIs Openなら隣の指から離れている状態（人差し指の場合中指と離れている状態）を表します。

残りの指（Thumb Feature Configsなど）は

- Curl: `Is Not` `Open`
- Flexion: `Is Not` `Open`

と設定します。
CurlがIs Not Openなら指が完全に伸びてない状態（少し曲がっているかしっかり曲がっている状態）を表します。
Flexionは指の付け根がどの程度曲がっているかを表し、FlexionがIs Not Openなら付け根が完全に伸びていない状態（少し曲がっているかしっかり曲がっている状態）を表します。

![](/images/hand-gesture-with-movement/image2.png)

`Shape Recognizer`の設定については、以下のwebページの指機能の節に詳しく記載されています。

https://developer.oculus.com/documentation/unity/unity-isdk-hand-pose-detection/

次に、RedShapeに`Shape Recognizer Active State`をアタッチし、

- `Hand`: RightHand（`OVRCameraRig/OVRInteraction/OVRHands/RightHand`）
- `Finger Feature State Provider`: HandFeaturesRight（`OVRCameraRig/OVRInteraction/OVRHands/RightHand/HandFeaturesRight`）
- `Shapes > Element 0`: （今作成した）RedShapeRecognizer

と設定します。これで、赤の手話の手の形を認識できるようになりました。

![](/images/hand-gesture-with-movement/image3.png)

# 4. 手の向きを認識する（RedDirection）
RedDirectionに`Transform Recognizer Active State`をアタッチし、

- `Hand`: RightHand
- `Transform Feature State Provider`: HandFeaturesRight
- `Transform Feature Configs`
    - Wrist Up: Is `True`
    - Palm Towards Face: Is `True`
- `Transform config`
    - `Up Vector Type`: `World`
    - `Feature Thresholds`: DefaultTransformFeatureStateThresholds（`Packages/Meta XR Interaction SDK/Runtime/DefaultSettings/PoseDetection/DefaultTransformFeatureStateThresholds.asset`）

と設定します。

![](/images/hand-gesture-with-movement/image4.png)

`Transform Feature Configs`のWrist Upは手首の親指側が上を向いているか（`True`なら向いている）、Palm Towards Faceは手のひらが顔のほうを向いているか（`True`なら向いている）を表します。

これで、赤の手話の手の向きを認識できるようになりました。

# 5. 手の動きを認識する（RedMovement）
RedMovementに`Joint Velocity Active State`をアタッチし、

- `Hand`: RightHand
- `Joint Delta Provider`: HandFeaturesRight
- `Joints`
    - `Joint`: `Hand Start`
    - `Relative To`: `Hand`
    - `Hand Axis`: `Wrist Backward`

と設定します。これで、右手が手首側に動くのを認識できるようになりました。

![](/images/hand-gesture-with-movement/image5.png)

# 6. 手の形・向き・動きを同時に認識する
RedGestureに`Active State Group`をアタッチし、

- `Active States`
    - `Element 0`: RedShape
    - `Element 1`: RedDirection
    - `Element 2`: RedMovement
- `Logic Operator`: `AND`

と設定します。これにより手の形・向き・動きのすべてが同時に認識された時に、はじめてこの`Active State Group`がアクティブになります。

![](/images/hand-gesture-with-movement/image6.png)

ここで、赤の手話が認識されたかどうかを確認するためのテキストを用意します。
シーンにTextMeshPro（DebugTextと名付けます）を配置し、DebugTextに`Active State Unity Event Wrapper`をアタッチします。

- Active State: RedGesture

と設定し、以下の図のようにイベントを登録します。これによって、

- 手話が認識されたらRedと表示されます。
- 手話が認識されなくなったら何も表示されなくなります。

![](/images/hand-gesture-with-movement/image7.png)

# 7. おわりに
以上の手順で、動きのあるハンドジェスチャ（今回は赤の手話）を認識できるようになりました！

![](/images/hand-gesture-with-movement/movie1.gif)