---
title: "DOTweenでTextを点滅させる"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity", "csharp", "dotween"]
published: true
---
# はじめに
初DOTweenということで、Textを点滅させる方法についてメモ。
# 1. DOTweenのセットアップ
アセットストアからDOTween (HOTween v2)を入手。
https://assetstore.unity.com/packages/tools/animation/dotween-hotween-v2-27676?locale=ja-JP
こちらの記事を参考にして、プロジェクトにインポート。
https://zenn.dev/ohbashunsuke/books/20200924-dotween-complete/viewer/dotween-03

# 2. Textを点滅（フェードアウト/フェードイン）させる
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;   // 追加
using DG.Tweening;      // 追加

public class TextManager : MonoBehaviour
{
    public Text dotweenText;
    public float dotweenInterval;

    void Start()
    {
            dotweenText.DOFade(0.0f, dotweenInterval)   // アルファ値を0にしていく
                       .SetLoops(-1, LoopType.Yoyo);    // 行き来を無限に繰り返す
    }
}
```
DoFadeを使うことで、Textのアルファ値を(第一引数)に向かって(第二引数)秒かけて変化させます。たとえばDoFade(0.0f, 1.5f)とすると、1.5秒かけてアルファ値0(=透明)に変化していきます。これによって、Textをフェードアウトさせることができます。

さらにSetLoopsを使うことで、変化を繰り返すことができます。第一引数には繰り返す回数(-1にすると無限に繰り返す)、第二引数には繰り返し方を指定します。たとえばSetLoops(-1, LoopType.Yoyo)とすると、ヨーヨーのように行ったり来たりする変化を無限に繰り返します。

LoopTypeについては、こちらの記事が参考になります。
https://zenn.dev/ohbashunsuke/books/20200924-dotween-complete/viewer/dotween-09

DoFadeとSetLoopsを組み合わせることで、Textのアルファ値の変化を行ったり来たりさせることができます。つまりフェードアウトとフェードインを繰り返すことができるのです。
![](/images/dotween-text/image1.gif)

# 3. Textの色を変化させる
ついでに、Textの色を変化させる処理を加えてみました。
```csharp
    void Start()
    {
            // Sequenceのインスタンスを作成
            var sequence = DOTween.Sequence();
            // Sequenceに処理を追加していく
            sequence.AppendInterval(4f);                          // 4秒待つ
            sequence.Append(dotweenText.DOColor(Color.red, 5f));  // 5秒かけて赤に
            sequence.Append(dotweenText.DOColor(Color.blue, 5f)); // 5秒かけて青に
            // Sequenceを実行
            sequence.Play();

            dotweenText.DOFade(0.0f, dotweenInterval)   // アルファ値を0にしていく
                       .SetLoops(-1, LoopType.Yoyo);    // 行き来を無限に繰り返す
    }
```
Sequenceを使うことで、処理を連続で行うことができます。Sequenceのインスタンスを作成し、そこに処理を追加していきます。上の例では、まず4秒待ってから5秒かけてTextの色を赤に変え、その後に5秒かけて青に変える処理を追加しています。最後にPlayで一連の処理を実行しています。
![](/images/dotween-text/image2.gif)