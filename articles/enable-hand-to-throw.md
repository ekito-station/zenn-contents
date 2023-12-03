---
title: "Meta XR SDKを使って、手でオブジェクトを投げられるようにする"
emoji: "⚾️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity", "vr"]
published: true
---
# はじめに
Meta XR SDKを使って、手でオブジェクトを投げられるようにする方法をメモします。Building BlocksのThrowable Itemを使っても簡単に実装できますが、ここではプレファブを使いながら自分で手を組み立てていく方法をまとめます。

![](/images/enable-hand-to-throw/image1.png =400x)
*Building BlocksのThrowable Item*

# 1. 手を用意する
まず、Unityプロジェクトに`Meta XR Core SDK`と`Meta XR Interaction SDK OVR Integration`をインストールします。

https://assetstore.unity.com/packages/tools/integration/meta-xr-core-sdk-269169

https://assetstore.unity.com/packages/tools/integration/meta-xr-interaction-sdk-ovr-integration-265014

新規作成したシーンに、OVRCameraRig（`Packages/Meta XR Core SDK/Prefabs/OVRCameraRig.prefab`）をドラッグ＆ドロップします。
このOVRCameraRigの配下にあるLeftHandAnchor（`OVRCameraRig/TrackingSpace/LeftHandAnchor`）およびRightHandAnchorの下に、OVRHandPrefab（`Packages/Meta XR Core SDK/Prefabs/OVRHandPrefab.prefab`）をドラッグ＆ドロップします。

![](/images/enable-hand-to-throw/image2.png =300x)

RightHandAnchorの下のOVRHandPrefabの、`OVR Hand > Hand Type`を`Hand Right`に設定しておきます。

![](/images/enable-hand-to-throw/image3.png =400x)

次に、OVRCameraRigの下にOVRInteraction（`Packages/Meta XR Interaction SDK OVR Integration/Runtime/Prefabs/OVRInteraction.prefab`）をドラッグ＆ドロップします。さらに、OVRInteractionの下にOVRHands（`Packages/Meta XR Interaction SDK OVR Integration/Runtime/Prefabs/OVRHands.prefab`）をドラッグ＆ドロップします。

![](/images/enable-hand-to-throw/image4.png =300x)

このままだとOVRHandPrefabとOVRHandsの両方が手として表示されてしまうので、OVRHandPrefabによる手の表示を削除します。そこで、OVRHandPrefabの
- `OVR Skeleton Renderer`
- `OVR Mesh`
- `OVR Mesh Renderer`
- `Skinned Mesh Renderer`

を非アクティブにします。

![](/images/enable-hand-to-throw/image5.png =400x)

# 2. 手でオブジェクトを掴んで投げられるようにする
まず、HandInteractorsLeft（`OVRCameraRig/OVRInteraction/OVRHands/LeftHand/HandInteractorsLeft`）およびHandInteractorsRightの下に、HandGrabInteractor（`Packages/Meta XR Interaction SDK/Runtime/Prefabs/HandGrab/HandGrabInteractor.prefab`）をドラッグ＆ドロップします。

![](/images/enable-hand-to-throw/image6.png =300x)

HandInteractorsLeftおよびHandInteractorsRightの`Best Hover Interactor Group > Interactors`に先ほどのHandGrabInteractorを追加します。

![](/images/enable-hand-to-throw/image7.png =400x)

これで、手がオブジェクトを掴めるようになりました。

次に、LeftHand（`OVRCameraRig/OVRInteraction/OVRHands/LeftHand`）およびRightHandの下にHandVelocityCalculator（`Packages/Meta XR Interaction SDK/Runtime/Prefabs/VelocityCalculators/HandVelocityCalculator.prefab`）をドラッグ＆ドロップします。

![](/images/enable-hand-to-throw/image8.png =300x)

HandVelocityCalculatorの`Hand Pose Input Device > Hand`に`LeftHand`または`RightHand`（そのHandVelocityCalculatorの親となる手）を設定します。

![](/images/enable-hand-to-throw/image9.png =400x)

先ほどのHandGrabInteractorの`HandGrabInteractor > Optionals > Velocity Calculator`に、今ドラッグ＆ドロップしたHandVelocityCalculatorを設定します（両手のHandGrabInteractorに設定するのを忘れずに）。

![](/images/enable-hand-to-throw/image10.png =400x)

（おそらくですが）これによって手の速度が計算されるので、手がオブジェクトを離した時の速度に基づいて離された後のオブジェクトの軌道を計算できるものと思われます。

# 3. オブジェクトが掴まれるようにする
手に掴まれるオブジェクトとして、ここではCubeを作成します。作成したCubeに以下のコンポーネントを追加します。

- `Rigidbody`
- `Grabbable`
- `Physics Grabbable`
- `Hand Grab Interactable`

`Hand Grab Interactable > Optionals > Physics Grabbable`にCube自身の`Physics Grabbable`を設定します。

![](/images/enable-hand-to-throw/image11.png =400x)

# おわりに
以上の作業で、下のようにオブジェクトを手で掴んで投げられるようになります！
![](/images/enable-hand-to-throw/movie1.gif)