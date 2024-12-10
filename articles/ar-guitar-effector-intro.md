---
title: "ARギターエフェクター with Meta Quest & Ableton Live"
emoji: "🎸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ar, unity, csharp, metaquest, max]
published: true
---
# 1 はじめに

こんにちは、Ekitoです！こちらの記事は、[Iwaken Lab. Advent Calendar 2024](https://qiita.com/advent-calendar/2024/iwakenlab)の12日目の記事です。

https://qiita.com/advent-calendar/2024/iwakenlab

## 1.1 自己紹介

あらためて[Iwaken Lab.](https://www.iwakenlab.jp/)に所属しているEkitoと申します。アドベントカレンダーなるものに初めて参加しております。
私は、ARと都市をキーワードとした音楽体験を作っています。最近は、2024年10月に虎ノ門ヒルズで開催された[TOKYO NODE OPEN LAB 2024 “XR PARADE”](https://www.tokyonode.jp/lab/events/openlab2024_xr_parade/index.html)にて、街の場所に合った音楽をレコメンドするARアプリ「Walking DJ」を展示しました。

https://youtu.be/wLX4vHpueWQ

「Walking DJ」の開発記録も記事にまとめてますので、ぜひ読んでみてください！

https://zenn.dev/ekito_station/articles/tnxr-walking-dj

## 1.2 今回やりたいこと

私は普段ギターを弾いていますが、以前からギターの音をARで操ってみたいと思ってました。
今回そのファーストステップとなるイメージが浮かんだので、それを実装してみたいと思います。

仕組みとしては、ギターの先から飛ばしたレーザー^[ギターの先からレーザーを飛ばすといえば、B'zのLOVE PHANTOMのライブパフォーマンス！]がARオブジェクトに当たるとエフェクターがONになってギターの音が変化するというものです。

![](/images/ar-guitar-effector-intro/image.png =500x)
*ARギターエフェクターのイメージ*

## 1.3 ARギターエフェクター全体像

使用したデバイスやツールは以下のとおりです。

- エレキギター
- オーディオインターフェース
- Meta Quest 3
    - コントローラー
- Unity 6
    - [Meta XR All-in-One SDK](https://assetstore.unity.com/packages/tools/integration/meta-xr-all-in-one-sdk-269657)
    - [OscJack](https://github.com/keijiro/OscJack)
- Ableton Live 
    - [Max for Live](https://www.ableton.com/ja/live/max-for-live/)

システム全体像を図にすると、以下のようになります。

![](/images/ar-guitar-effector-intro/system_config.png)
*システム構成図*

1. まず、ギターをPCに接続してAbleton LiveというDAWを通して音を出します。
2. 次に、Meta Questを被りARオブジェクトに向かってレーザー（レイ）を飛ばしたら、そのことをOSC通信でAbleton Liveに通知します。
3. 通知を受け取ったAbleton LiveはエフェクターをONにします。

これからシステムの実装について、Meta Quest側とAbleton Live側に分けて説明していきたいと思います。

# 2 Meta Quest側の実装

Meta Quest 3にインストールするARアプリをUnityで実装していきます。あらかじめ[Meta XR All-in-One SDK](https://assetstore.unity.com/packages/tools/integration/meta-xr-all-in-one-sdk-269657)をインポートしておきます。

## 2.1 ギターの先からレイを飛ばす

まず、ギターのヘッドにMeta Quest 3のコントローラーを取りつけます。

![](/images/ar-guitar-effector-intro/controller_on_guitar.jpg =400x)
*といってもコントローラーの紐を巻きつけるだけ*

次に、新規シーンを作成してコントローラーからレイを飛ばすための設定をします。

:::message
最初は`[Building Block] Ray Interaction`を使うことを考えましたが、コントローラートラッキングに対応してなさそうだったので、以下の次善策を採用してます。
:::

Meta XR Interaction SDKのRay Interactionに関するサンプルシーン（`Assets/Samples/Meta XR Interaction SDK/71.0.0/Example Scenes/RayExamples.unity`）から`OVRCameraRig`を取ってきます。これによって、コントローラーからレイを飛ばせるようになります。

![](/images/ar-guitar-effector-intro/ray_examples.png)
*RayExamplesシーンのOVRCameraRig*

## 2.2 レイに当たるARオブジェクトを作る

次に、レイに当たるゲームオブジェクト（レイインタラクタブル）を作成します。以下のサイトを参考にしました。
https://developers.meta.com/horizon/documentation/unity/unity-isdk-create-ray-interactions/?locale=ja_JP

1. シーンにCubeを配置して、`Ray Interactable`コンポーネントをアタッチします。
2. Cubeに空の子オブジェクトを追加して、`Box Collider`と`Collider Surface`コンポーネントをアタッチします。
3. Cubeの`Ray Interactable`の`Surface`に、子オブジェクトの`Collider Surface`を登録します。

## 2.3 レイが当たったらOSCメッセージを送る

そして、コントローラーからのレイがレイインタラクタブルに当たったときに、OSCメッセージをPCに送る仕組みを作ります。

### 2.3.1 OSCメッセージを送るためのスクリプトを作成する

シーンに空のゲームオブジェクト（`EffectorSwitchDetector`とします）を配置します。OSCメッセージを送るためのスクリプト（`OSCSender.cs`）を作成して、`EffectorSwitchDetector`にアタッチします。

OSCメッセージを送るために、[OscJack](https://github.com/keijiro/OscJack)というライブラリを使用します。

https://github.com/keijiro/OscJack

レイがレイインタラクタブルに当たったとき、およびレイがレイインタラクタブルから外れたときにOSCメッセージを送る処理を以下のように作ります。

:::details OSCSenderの内容
```csharp:OSCSender.cs
using OscJack;
using UnityEngine;

public class OSCSender : MonoBehaviour
{
    private OscClient _client = new OscClient("PCのIPアドレス", 9000); // 9000はポート番号

    // レイがレイインタラクタブルに当たったときに呼び出す関数
    public void OnHoverEnter()
    {
        _client.Send("/unity", 1);
    }

    // レイがレイインタラクタブルから外れたときに呼び出す関数
    public void OnHoverExit()
    {
        _client.Send("/unity", 0);
    }
    
    private void OnDestroy()
    {
        _client.Dispose();
    }
}
```
:::

- レイがレイインタラクタブルに当たったとき→1（`OnHoverEnter`）
- レイがレイインタラクタブルから外れたとき→0（`OnHoverExit`）

の値をPCに送るようにします。

### 2.3.2 レイが当たったとき/外れたときに関数を呼び出す

`EffectorSwitchDetector`にさらに以下のコンポーネントをアタッチします。

- `Interactor Active State`
- `Active State Unity Event Wrapper`

#### Interactor Active State

`Interactor Active State`の`Interactor`にコントローラーの`Ray Interactor`を登録し、`Property`に`Is Hovering`を選択します。

![](/images/ar-guitar-effector-intro/interactor_active_state.png =500x)
*`Interactor Active State`のInspector*

![](/images/ar-guitar-effector-intro/controller_ray_interactor.png =500x)
*コントローラーの`Ray Interactor`の場所*

#### Active State Unity Event Wrapper

`Active State Unity Event Wrapper`の`Active State`に先ほどの`Interactor Active State`を登録し、`When Activated`イベントに先ほど作成した`OnHoverEnter`を、`When Deactivated`イベントに`OnHoverExit`を登録します。

![](/images/ar-guitar-effector-intro/active_state_unity_event_wrapper.png =500x)
*`Active State Unity Event Wrapper`のInspector*

これによって、コントローラーの`Ray Interactor`のActive State（コントローラーのレイがレイインタラクタブルにHoveringしている状態）がActivateされたときに`OnHoverEnter`が、Deactivateされたときに`OnHoverExit`が呼び出されるようになります。

# 3 Ableton Live側の実装

続いてAbleton Liveで、OSCメッセージを受け取ってエフェクターのON/OFFを切り替えるプロジェクトを作成します。このとき、Ableton Liveの拡張機能であるMax for Liveを使用します。Max for Liveは、Maxというビジュアルプログラミング言語を活用してAbleton Liveの機能を制御することができます。

Max for Liveの使い方は、[大黒淳一](https://www.youtube.com/@junichioguro)さんの動画シリーズで勉強させていただきました。大変参考になりました！

https://youtu.be/zbR2oc3VGk8?feature=shared

## 3.1 ギター用トラックを作成する

まず、Ableton Liveの新規プロジェクトを作成し、ギターを鳴らすためのトラックを作成します。

Audioトラックに、以下のデバイス（エフェクトなどのコンポーネントのこと）を追加します。

- Max Audio Effect
- Pedal（歪みエフェクター）
- Amp
- Cabinet
- Reverb

![](/images/ar-guitar-effector-intro/ableton_live.png)
*Ableton Liveのプロジェクト*

このうち、PedalのON/OFFをMax Audio Effectで制御していきます。

## 3.2 OSCメッセージを受け取る

ここから、Max Audio Effect（Max for Liveデバイス）を編集していきます。

まず、Max for Liveデバイスに`udpreceive`というOSCメッセージを受け取るためのオブジェクトを追加し、引数にポート番号を指定します。先ほど`OSCSender.cs`でポート番号に9000を指定したので、ここでも9000を指定します。

次に`OSC-route`オブジェクトを追加して、OSCメッセージをアドレスに基づいて振り分けて値を取り出します。`OSCSender.cs`から送られるOSCメッセージのアドレスはすべて`/unity`なので、アドレスが`/unity`のOSCメッセージを受け取ったらその値（0または1）を取り出す処理を作ります。

![](/images/ar-guitar-effector-intro/m4l_osc.png =250x)
*Max for LiveデバイスのOSCメッセージを受け取る部分*

## 3.3 エフェクターのON/OFFを切り替える

そして[Live API](https://docs.cycling74.com/userguide/m4l/live_api_overview/)によって、Max for LiveデバイスからAbleton Liveプロジェクトにアクセスして制御します。Ableton Liveプロジェクトの各要素には、Live Object Model（LOM）に基づいてパスが割り当てられています。

今回アクセスしたい要素は、「ギター用トラック」の「Pedalデバイス」の「ON/OFFを示すパラメーター」です。ここでは、

- 「ギター用トラック」はプロジェクトの最初のトラックなので`tracks 0`
- 「Pedalデバイス」はトラック内の2番目のデバイスなので`devices 1`
- 「ON/OFFを示すパラメーター」は`parameters 0`

となるので、「ギター用トラック」の「Pedalデバイス」の「ON/OFFを示すパラメーター」のパスは以下のようになります。

```
path live_set tracks 0 devices 1 parameters 0
```

このパスを`live.path`オブジェクトに渡してIDを発行し、そのIDを`live.object`オブジェクトに渡します。

最後に、OSCメッセージから取り出した値も`live.object`オブジェクトに渡すことで、IDが示す要素に値をセットします。これによって、0がセットされるとギター用トラックのPedalがOFFになり、1がセットされるとONになります。

![](/images/ar-guitar-effector-intro/m4l_all.png =500x)
*Max for Liveデバイス全体*

## 3.4 完成！

以上の実装によって、ギターの先からレイを飛ばすことでエフェクターのON/OFFを切り替えられるようになりました。レイがCubeに当たるとギターの音が歪み、Cubeから外れるとギターの音がクリーンに戻るようになります。

https://youtu.be/0UXjz2GF3VU

# 4 おわりに

2年ちょっと前、ARを始めたての頃に「ギターの弦を街に張り巡らせてみたら？」というアイデアを実装したことがあったので、またARとギターを組み合わせられて原点回帰できた感があります。

https://youtube.com/shorts/p2rnRnbST_o?feature=share

今回はエフェクターのON/OFFを切り替える簡単なシステムでしたけれど、Max for Liveで制御できる要素はとても多岐に渡りそうなので、今後さらに深掘りしていきたいと思います。

[Iwaken Lab. Advent Calendar 2024](https://qiita.com/advent-calendar/2024/iwakenlab)の明日の担当は[JJ](https://qiita.com/jjay)さんです。お楽しみに！
