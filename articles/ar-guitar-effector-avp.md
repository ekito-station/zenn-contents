---
title: "ARギターエフェクター Apple Vision Pro ver."
emoji: "🎸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ar, swift, applevisionpro, visionos, realitycomposerpro]
published: true
---
# 1 はじめに

この記事は、2025年3月1日に東京ポートシティ竹芝で開催された「[第16回VRフェス](https://peatix.com/event/4292335?lang=ja-jp)」にて展示した、"AR Guitar Effector【Apple Vision Pro ver.】"の開発記録をまとめたものです。

https://youtu.be/IuJFRhzgsK8

## 1.1 AR Guitar Effector (ARギターエフェクター) とは

「AR Guitar Effector」は、AR技術を活用した新しいギター演奏体験を提案するプロジェクトです。

![](/images/ar-guitar-effector-avp/image_arguitar.jpg =500x)
*AR Guitar Effectorのイメージ*

具体的な仕組みとしては、空中に浮かぶARのエフェクタースイッチに向かってギターの先端からレイを飛ばすと、ギターの音にエフェクトがかかります。色々なスイッチにギターを向けながら演奏することで、多様に変化する音を楽しむことができます。

[本プロジェクトの第1弾](https://zenn.dev/ekito_station/articles/ar-guitar-effector-intro)として昨年2024年末に、Meta Quest 3を使用したAR Guitar Effectorシステムの実装について記事にまとめました。システムの詳細については、以下の記事をご覧ください。

https://zenn.dev/ekito_station/articles/ar-guitar-effector-intro

## 1.2 Apple Vision Pro ver.

本プロジェクトの第2弾となる今回は、Apple Vision Proを使用したAR Guitar Effectorシステムを実装しました。

![](/images/ar-guitar-effector-avp/thumbnail.jpg =500x)
*AR Guitar Effector 【Apple Vision Pro ver.】*

### 1.2.1 Object Tracking機能

システムに使用するHMDをMeta Quest 3からApple Vision Proに変更したことによって生じる大きな違いは、ギターの先端のトラッキング手法です。Meta Quest 3バージョンではギターのヘッドにQuest 3のコントローラーを取り付けていましたが、Apple Vision Proには公式のコントローラーがありません。そこで、代わりに使用した機能は[Object Tracking](https://developer.apple.com/documentation/visionos/implementing-object-tracking-in-your-visionos-app/)です。

https://developer.apple.com/documentation/visionos/implementing-object-tracking-in-your-visionos-app/

Object Trackingは、Apple Vision Proが実空間上の特定の物理オブジェクトを認識すると、その物理オブジェクトの位置や方向にバーチャルオブジェクトを追従させることを可能にします。この機能を活用することで、Apple Vision Proがギターのヘッドを認識すると、その位置を起点としてギターの向いている方向にレイを飛ばす仕組みを実装することができます。

### 1.2.2 デメリット・メリット

Meta Quest 3バージョンと比較した際のApple Vision Proバージョンのデメリットは、Object Trackingのトラッキングレートつまりトラッキングの更新頻度が、(自分の試した限りでは) Quest 3のコントローラーより小さかったことです (AR FoundationのImage Trackingよりは大きかった)。

一方でメリットは、ギター自体に何かを取り付ける必要がないことです。Meta Quest 3バージョンでは、コントローラーがギターから外れたらシステム全体が機能しなくなってしまいますが、Apple Vision Proバージョンではその心配がありません。

### 1.2.3 その他追加機能

以上のようなApple Vision Proならではの変更点のほかに、以下の機能を実装しました。

- エフェクタースイッチの種類を追加した
- エフェクタースイッチを手で掴んで移動できるようにした

今回エフェクタースイッチを3種類 (Distortion, Redux, Volume Swells) 用意し、それぞれの位置を手で掴んで動かして調整できるようにしました。そのため、エフェクタースイッチを近くに並べて、それらを狙ってレイを飛ばすことで、同時に複数のエフェクトをかけることも可能になりました。実際の操作の様子は、以下の動画をご覧ください。

https://www.youtube.com/watch?v=IuJFRhzgsK8&t=28s

# 2 実装

## 2.1 システム全体像

システム構成図は以下のようになります。ギターの先端から飛ばされたレイがエフェクタースイッチに当たったり外れたりすると、Apple Vision ProからPCにOSCメッセージが送信されます。受信したOSCメッセージに基づいてPC上のAbleton Liveでギター音のエフェクトが制御されます。

![](/images/ar-guitar-effector-avp/system_config.png =500x)
*システム構成図*

## 2.2 使用ソフト・ライブラリ

使用ソフト・ライブラリは以下のとおりです。PCはM1搭載MacBook Proを使用しました。

- Xcode (Version 16.2)
    - Reality Composer Pro
    - Create ML
    - [OSCKit](https://github.com/orchetect/OSCKit)
- Ableton Live (Suite 12.1.10)
    - Max for Live

以下のセクションでは、Xcodeでの実装について解説していきます。Ableton Liveでの実装については、[Meta Quest 3バージョン](https://zenn.dev/ekito_station/articles/ar-guitar-effector-intro)とほぼ変わらないので、そちらの解説記事を参照してください。

https://zenn.dev/ekito_station/articles/ar-guitar-effector-intro

## 2.3 Xcodeでの実装の詳細

### 2.3.1 ギターのヘッドの3Dモデル作成

まず、Object Tracking機能を使って、Apple Vision Proのカメラがギターのヘッドを認識したらその位置にアンカーが配置される処理を実装していきます。

#### 2.3.1.1 ギターのヘッドを3Dスキャン

[Object Capture](https://developer.apple.com/documentation/realitykit/realitykit-object-capture/)が組み込まれたiOSアプリを使用して、ギターのヘッドを3Dスキャンして3Dモデルを作成します (Object Captureとは、フォトグラメトリを使用して3Dモデルを作成できるAPIです)。私は、「[KIRI Engine:3D Scanner & LiDAR](https://apps.apple.com/jp/app/kiri-engine-3d-scanner-lidar/id1577127142)」というアプリを使用しました。

![](/images/ar-guitar-effector-avp/scanned_data.jpg =300x)
*ギターのヘッドの3Dモデル*

:::message
3Dスキャンしたギターのヘッドのサイズは、およそ (幅) 10cm × (高さ) 20cm × (奥行き) 2cmでした。

一方、ギターのヘッドに加えてネックを含めた3Dモデルも作成してみましたが、このモデルを用いた場合、Object Trackingがうまく機能しませんでした。ギターのヘッド + ネックのサイズは、およそ (幅) 10cm × **(高さ) 60cm** × (奥行き) 2cmでした。

3Dモデルのサイズが大きくなると、トラッキング精度が低下するようです。
:::

#### 2.3.1.2 Create MLでレファレンスオブジェクト生成

Create MLというXcode内のアプリで機械学習を行うことで、3Dモデルからレファレンスオブジェクトを生成します。このレファレンスオブジェクトは、後でReality Composer Proにインポートするために使用します。

Create MLは、Xcodeから`Xcode > Open Developer Tool > Create ML`で開くことができます。

![](/images/ar-guitar-effector-avp/createml.png)
*Create MLの画面*

ビューポートに3Dモデルをドラッグ&ドロップしてから、Viewing anglesを設定します。Viewing Anglesとは物理オブジェクトをどの角度から認識し得るかの設定で、All Angles (全角度から)、Upright (下方以外から)、Front (下方・背面以外から) の中から選択できます。私の場合は、ギターのヘッドは色々な角度から見え得るので、All Anglesを選択しました。

そして、Trainボタンをクリックして機械学習を開始させます。私の場合は、完了までに22時間半ほどかかりました。

機械学習が完了したら、Outputタブに移動しGetボタンをクリックして、レファレンスオブジェクトを保存します。

### 2.3.2 Object Trackingのアンカー作成

#### 2.3.2.1 Xcodeプロジェクト作成

ここで、Xcodeで新規プロジェクトを作成します。テンプレートは、`visionOS > Application > App`を選択します。

![](/images/ar-guitar-effector-avp/xcode.png)
*Xcodeの画面*

続いて、Reality Composer Proを開きます。Xcode画面左のナビゲーターエリアから`Packages > RealityKitContent > Package`を開き、ビューポート右上のOpen in Reality Composer Proボタンをクリックします。

#### 2.3.2.2 Reality Composer Proでアンカー設定

![](/images/ar-guitar-effector-avp/rcp.png)
*Reailty Composer Proの画面*

`Immersive.usda`というシーンを開き、シーン内に空のTransformエンティティを配置します (シーンに元からあるSphere_LeftやSphere_Rightは必要なければ削除)。私の場合は、このTransformエンティティの名称を`GuitarAnchor`に変更しました。このTransformエンティティを選択し、画面右のインスペクターでAnchoringコンポーネントを追加します。

Anchoringコンポーネントにおいて、以下のように各項目を設定します。

- Target: `Object`
- Anchor Object: 先ほど生成したレファレンスオブジェクト

これで、レファレンスオブジェクトと同じ見た目の物理オブジェクトがApple Vision Proのカメラに認識されたら、その位置にアンカーが配置されるようになります。

:::message
Object TrackingはImmersiveSpaceでしか機能しないため、`Immersive`シーンを用いる必要があります。
:::

### 2.3.3 エフェクタースイッチ作成

今度は、エフェクタースイッチとなるキューブを作成していきます。シーン内にPrimitive ShapeのCubeエンティティを配置していき、それぞれ適切な名称に変更します (私の場合は`DistortionSwitch`、`ReduxSwitch`、`ViolinSwitch`)。

また、各キューブに以下のコンポーネントを追加します。

- Input Targetコンポーネント (手で掴んで移動できるように)
- Collisionコンポーネント (手で掴んで移動できるように、かつレイが当たったことを検知できるように)

#### 2.3.3.1 Shader Graph

各キューブの見た目を調整するために、Reality Composer ProのShader Graphを活用しました。

たとえば、`ReduxSwitch`に適用したシェーダーは、Shader Graphで以下のように構築しました。ReduxとはAbleton Liveに内蔵されているエフェクトの一種で、オーディオ信号のサンプリングレート・ビット解像度を下げることができます。このエフェクトを視覚的に表現するために、グラデーションの解像度を落とす、すなわちグラデーションをモザイク調にするシェーダーを作成しました。

![](/images/ar-guitar-effector-avp/shader_graph.png)
*Shader Graphの画面*

#### 2.3.3.2 キューブを手で掴んで移動できるように

各キューブを手で掴んで移動できるようにする処理を実装するために、Xcodeに戻りスクリプトを編集します。

`ImmersiveView.swift`を開くと、先ほどReality Composer Proで編集した`Immersive`シーンをRealityView内のエンティティとして読み込む処理が元から書かれています。ここに、ドラッグジェスチャーを定義してRealityViewに適用する処理を追加します。

```swift:ImmersiveView.swift
import SwiftUI
import RealityKit
import RealityKitContent

struct ImmersiveView: View {

    var body: some View {
        RealityView { content in
            // ImmersiveシーンをRealityViewに読み込む
            if let immersiveContentEntity = try? await Entity(named: "Immersive", in: realityKitContentBundle) {
                content.add(immersiveContentEntity)
            }
        }
        .gesture(dragGesture) // ドラッグジェスチャーをRealityViewに適用
    }
    // ドラッグジェスチャーを定義
    var dragGesture: some Gesture {
        DragGesture()
            .targetedToAnyEntity()
            .onChanged { value in
                value.entity.position = value.convert(value.location3D, from: .local, to: value.entity.parent!)
            }
    }
}
```

これによって、親指と人差し指でキューブをピンチして保持した状態で手を動かすことで、キューブをドラッグして移動させることができるようになります。

### 2.3.4 アンカーの位置からレイキャスト

アンカーの位置からレイを飛ばす処理を実装していきます。

#### 2.3.4.1 アンカーの位置や方向を取得

アンカーが配置された際にそのアンカーに追従させるためのエンティティ (以下、RayOrigin) を、Reality Composer Proの`Immersive`シーン内に配置しておきます。私の場合は、`RayOrigin`という名称でCubeエンティティを配置しました。

![](/images/ar-guitar-effector-avp/ray_origin.png)
*RayOrigin*

アンカーのTransformを取得するためには、あらかじめXcodeの`ImmersiveView.swift`において、[SpatialTrackingSession.Configuration](https://developer.apple.com/documentation/realitykit/spatialtrackingsession/configuration)を実行する必要があります。

そのうえで、[transformMatrix](https://developer.apple.com/documentation/realitykit/hastransform/transformmatrix(relativeto:))を用いてアンカーのTransformを取得します。そして、[setTransformMatrix](https://developer.apple.com/documentation/realitykit/hastransform/settransformmatrix(_:relativeto:))を用いてRayOriginのTransformをアンカーのTransformと揃えます。この処理を毎フレーム繰り返すことで、RayOriginがアンカーに追従するようになります。

スクリプトは以下のようになります (説明に直接関係する箇所のみ抜粋しています)。

```swift:ImmersiveView.swift
struct ImmersiveView: View {
    var body: some View {
        RealityView { content in
            if let immersiveContentEntity = try? await Entity(named: "Immersive", in: realityKitContentBundle) {
                content.add(immersiveContentEntity)

                // SpatialTrackingSession.Configurationを実行する
                let session = SpatialTrackingSession()
                let configuration = SpatialTrackingSession.Configuration(tracking: [.object, .world])
                let _ = await session.run(configuration)

                // Updateイベントをsubscribeする
                _ = content.subscribe(to: SceneEvents.Update.self) { event in
                    updateRaycast() // この関数が毎フレーム実行される
                }
            }
        }
    }
    
    func updateRaycast() {
        // アンカーのTransformを取得する
        let anchorTransform = guitarAnchor?.transformMatrix(relativeTo: nil)
        // guitarAnchorはReality Composer ProのImmersiveシーン内のGuitarAnchorを参照する変数
        
        // RayOriginのTransformをアンカーのTransformと揃える
        rayOrigin?.setTransformMatrix(anchorTransform ?? matrix_identity_float4x4, relativeTo: nil)
        // rayOriginはReality Composer ProのImmersiveシーン内のRayOriginを参照する変数
    }
}
```

:::message alert
最初は、RayOriginを介さずに直接アンカーのtranslation (位置) やrotation (向き) を用いてレイキャストを実行しようとしましたが、上手くいかず四苦八苦しました。

アンカーはシーン内の他のエンティティの座標系 (ワールド座標系) とは異なる座標系にあるようなので、[transformMatrix](https://developer.apple.com/documentation/realitykit/hastransform/transformmatrix(relativeto:))を用いてアンカーのTransformをワールド座標系におけるTransformに変換する必要がありました。また、[transformMatrix](https://developer.apple.com/documentation/realitykit/hastransform/transformmatrix(relativeto:))を用いて取得したアンカーのTransformからは、アンカーのtranslationやrotationにアクセスできませんでした。

そのため上記のように、RayOriginを介するアプローチを採用しました。
:::

#### 2.3.4.2 レイキャストの実行

[raycast](https://developer.apple.com/documentation/realitykit/scene/raycast(origin:direction:length:query:mask:relativeto:))を用いて、RayOriginの位置からRayOriginの上方に向かってレイキャストを実行します。

```swift:ImmersiveView.swift
// RayOriginの位置を取得
let originPosition = rayOrigin?.convert(transform: Transform(), to: nil).translation
// RayOriginの上方を取得
let upDirection = rayOrigin?.convert(direction: SIMD3<Float>(0, 1, 0), to: nil)

// レイキャストの実行
guard let results = rayOrigin?.scene?.raycast(origin: originPosition ?? SIMD3<Float>(0, 0, 0), direction: upDirection ?? SIMD3<Float>(0, 1, 0), length: 10) else {
    print("Error: results is nil.")
    return
}
```

[raycast](https://developer.apple.com/documentation/realitykit/scene/raycast(origin:direction:length:query:mask:relativeto:))によって飛ばされるレイ自体はシーン内に表示されません。そこで、レイが飛んでいる方向が視覚的に分かりやすいように、Reality Composer Proにおいて、アンカーの子として空のTransformエンティティを配置し、そこにParticle Emitterコンポーネントを追加します。これによって、アンカーの位置からレイが飛ぶのと同じ方向にパーティクルを飛ばします。

![](/images/ar-guitar-effector-avp/particle.png)
*パーティクルを飛ばす*

### 2.3.5 レイがキューブに当たったらOSCメッセージを送信

最後に、レイがキューブに当たったらOSCメッセージを送信する処理を実装していきます。

#### 2.3.5.1 OSCKitの追加

OSCメッセージを送信するためのライブラリとして、[OSCKit](https://github.com/orchetect/OSCKit)をXcodeのプロジェクトに追加します。

`File > Add Package Dependencies...`をクリックして表示されたウィンドウの検索バーに、`https://github.com/orchetect/OSCKit`と入力します。OSCKitが表示されたら選択してAdd Packageボタンをクリックします。

![](/images/ar-guitar-effector-avp/osckit1.png)
*OSCKitの追加 ①*

次に表示されるウィンドウのAdd to Targetの列で、プロジェクト名と同じ名称のTarget (「プロジェクト名+Tests」ではないほう) を選択してAdd Packageボタンをクリックします。

![](/images/ar-guitar-effector-avp/osckit2.png)
*OSCKitの追加 ②*

#### 2.3.5.2 OSCメッセージの送信

`ImmersiveView.swift`において、先に述べたようにレイキャストを実行し、その結果を取り出します。レイが当たったエンティティの名称がキューブの名称と一致する場合、OSCメッセージを送信する処理を実装します。

スクリプトは以下のようになります (説明に直接関係する箇所のみ抜粋しています)。この例では、レイがDistortionSwitchと当たった場合、OSCアドレス`/arguitar`に値`1`を送信しています。

```swift:ImmersiveView.swift
import OSCKit

struct ImmersiveView: View {
    private let OSC_IP_ADDRESS = "000.000.000.000" // Ableton Liveを起動させているPCのIPアドレス
    private let OSC_PORT  = 0000 // ポート番号
    private let oscClient  = OSCClient()
    
    func updateRaycast() {
        (中略)
        // レイキャストの実行
        guard let results = rayOrigin?.scene?.raycast(origin: originPosition ?? SIMD3<Float>(0, 0, 0), direction: upDirection ?? SIMD3<Float>(0, 1, 0), length: 10) else {
            print("Error: results is nil.")
            return
        }

        // レイキャスト結果の処理
        for result in results {
            if (result.entity.name == "DistortionSwitch") {
                // レイと当たったエンティティの名称が"DistortionSwitch"だったらOSCメッセージを送信
                let msg = OSCMessage("/arguitar", values: [1])
                do {
                    try oscClient.send(msg, to: OSC_IP_ADDRESS, port: UInt16(OSC_PORT))
                } catch {
                    print("Failed to send OSC message: \(error.localizedDescription)")
                }
            }
        }
    }
}
```

# おわりに

以上、"AR Guitar Effector【Apple Vision Pro ver.】"の開発記録をまとめました。[第16回VRフェス](https://peatix.com/event/4292335?lang=ja-jp)では来場者の皆さんに体験してもらったり私が演奏している様子を見てもらったりして、面白いという声をたくさんいただけて嬉しかったです。また、第16回Vアカオーディションではありがたいことに、技術的観点で高い評価だった作品に贈られる「技術賞」と、受講生間で高い評価だった作品に贈られる「Student賞」を受賞することができました！

![](/images/ar-guitar-effector-avp/vr_academy.jpg =500x)
*第16回Vアカオーディションにて技術賞受賞*

今後も本プロジェクトは発展させていきたいと思います。そして、AR Guitar Effectorを使ったライブパフォーマンスの実践として、2025/3/25にBuzzFront Yokohamaで開催される「[Iwaken Lab. Performers Fest](https://lu.ma/9o0wuz9n?locale=ja&tk=dSOAms)」に出演します！「Immersive Live with AR Guitar」という演目で、MR技術を使った没入型演出と組み合わせたギターパフォーマンスを披露する予定です。XRなどのテクノロジーを活用した音楽表現・ライブ演出にご興味がある方はぜひお越しください！

https://lu.ma/9o0wuz9n?locale=ja&tk=dSOAms

![](/images/ar-guitar-effector-avp/performers_fest.jpg)

## 参考記事

- https://1planet.co.jp/tech-blog/applevisionpro-oneplanet-visionos2-object-tracking

https://1planet.co.jp/tech-blog/applevisionpro-oneplanet-visionos2-object-tracking

- https://1planet.co.jp/tech-blog/applevisionpro-realityview-model-drag

https://1planet.co.jp/tech-blog/applevisionpro-realityview-model-drag

- https://1planet.co.jp/tech-blog/applevisionpro-oneplanet-24062601-spatial-tracking

https://1planet.co.jp/tech-blog/applevisionpro-oneplanet-24062601-spatial-tracking

- https://zenn.dev/meson_tech_blog/articles/visionos-collision-detection?redirected=1

https://zenn.dev/meson_tech_blog/articles/visionos-collision-detection?redirected=1


