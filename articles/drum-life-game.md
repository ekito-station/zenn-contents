---
title: "Drum Life Game開発メモ"
emoji: "🥁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity", "csharp", "個人開発"]
published: true
---
# はじめに
Drum Life Gameという音楽演奏アプリを制作しました。ライフゲームのルールに基づいてドラムの演奏パターンが変化します。本記事では、アプリの開発過程についてメモします（説明のために一部コードを変更しています）。
https://youtu.be/IZWCfksDZnE

### Drum Life Game
iOS ver.
@[card](https://apps.apple.com/jp/app/drum-life-game/id1669654073)
Android ver.
@[card](https://play.google.com/store/apps/details?id=com.ekito_station.DrumLifeGame)

# 開発過程

### 1. セルを並べる
下図のように、セルの行列を作ります。行はドラムの各パート（キック・スネアなど）、列は各拍（例: BEAT0 = 1拍目）を意味します。
![](/images/drum-life-game/image1.png)

セルにはImageを使用し、デフォルトではColorを黒色に設定します。セルにAudioSourceをアタッチし、セルが存在する行に応じて該当するパートの音をAudioClipに設定します。

あとで列ごとに音を再生させるために下図のように、1列分のセルをまとめて1つの空のGameObject（下図ではBEAT0）の子オブジェクトに設定します。
![](/images/drum-life-game/image2.png)

### 2. クリックされたセルのON/OFFを切り替える
セルにはONの状態とOFFの状態があり、ONの状態のセルから音が鳴ります。
セルの色が黒の場合はOFFの状態で、白の場合はONの状態を意味します。

セルがクリックされると、黒色（OFFの状態）のセルは白色（ONの状態）に、白色のセルは黒色になるようにします。
以下のスクリプト（NoteController）をセルにアタッチします。
```csharp:NoteController.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class NoteController : MonoBehaviour
{
    public Image image;             // セルにアタッチされているImageコンポーネント
    [System.NonSerialized]
    public bool isPlayed = false;   // trueならONの状態、falseならOFFの状態

    public void OnCellClicked()     // セルがクリックされたときに実行される関数
    {
        if (isPlayed)   // 元々ONの状態の場合
        {
            image.color = Color.black;  // 黒色にする         
            isPlayed = false;           // OFFの状態にする
        }
        else            // 元々OFFの状態の場合
        {
            image.color = Color.white;  // 白色にする           
            isPlayed = true;            // ONの状態にする
        }
    }
}
```
そして、セルにButtonコンポーネントをアタッチし上記のOnCellClicked()をOnClickに登録します。

### 3. 左の列から順番に音を再生させる
画面下のPLAYボタンが押されると、左の列から右に向かって順番に音が再生されます。
列の中で白色のセルからのみ音が鳴ります。
![](/images/drum-life-game/image3.png)

まず、NoteControllerに以下の関数を追加します。
自らが存在する列の再生される順番が来たときに、OFFの状態のセルは茶色になり、ONの状態のセルはベージュ色になるとともに音を鳴らします。次の列の再生される順番になったら（0.3秒後）、色を元に戻します。
```csharp:NoteController.cs
    public AudioSource audioSource;
    public AudioClip audioClip;

    public void PlayNote()
    {
        if (isPlayed)   // ONの状態なら
        {
            // 音を鳴らす
            audioSource.PlayOneShot(audioClip);
            // 色を変える
            Color _colorPlayed;
            if (ColorUtility.TryParseHtmlString("#eedcb3", out _colorPlayed))
                image.color = _colorPlayed;     // ベージュ色にする
        }
        else    // OFFの状態なら
        {
            // 色を変える
            Color _colorNotPlayed;
            if (ColorUtility.TryParseHtmlString("#6c3524", out _colorNotPlayed))
                image.color = _colorNotPlayed;  // 茶色にする
        }
        Invoke("ReturnColor", 0.3f);    // 0.3秒後に色を戻す
    }

    private void ReturnColor()
    {
        if (isPlayed)　image.color = Color.white;   // ONの状態なら白色に戻す
        else　image.color = Color.black;            // OFFの状態なら黒色に戻す
    }
```
次に、各列でセルのPlayNote()を同時に実行するために、UnityEventを使用します。
各列のセルをまとめている空のGameObject（1で説明したBEAT0のようなGameObject）に以下のスクリプト（BeatController）をアタッチします。
```csharp:BeatController.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

public class BeatController : MonoBehaviour
{
    public UnityEvent noteEvent = new UnityEvent();

    public void PlayNotes()
    {
        noteEvent.Invoke();
    }
}
```
インスペクタ上で、UnityEventにその列の各セルのPlayNote()を追加します。
![](/images/drum-life-game/image4.png)

そして、シーン上に新たに作成した空のGameObjectに、以下のスクリプト（LifeGameManager）をアタッチします。
```csharp:LifeGameManager.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class LifeGameManager : MonoBehaviour
{
    public BeatController[] beats;

    private IEnumerator PlayBeats() // 列を順番に再生するコルーチン
    {
        while (true)
        {
            foreach (BeatController beat in beats)
            {
                beat.PlayNotes();                       // 1つの列を再生する
                yield return new WaitForSeconds(0.3f);  // 次の列の再生まで0.3秒待つ
            }
        }
    }
}
```
インスペクタ上で、beatsというリストに各列のBeatControllerを追加します。
![](/images/drum-life-game/image5.png)

これによって、LifeGameManagerのPlayBeats()を実行すると以下のような流れで音が再生されるようになります。
1. 各列のBeatControllerのPlayNotes()が実行される
2. その列の各セルのNoteControllerのPlayNote()が実行される
3. そのセルの状態に応じてセルの色が変化し音が鳴る

### 4. ライフゲームの要領で、セルのON/OFFのパターンを変化させる
ここで、ライフゲームのルールについて簡単に説明します。
![](/images/drum-life-game/image6.png)
・（誕生）ONの状態のセル3つに囲まれたOFFの状態のセルは、ONの状態に切り替わります。
・（維持）ONの状態のセル2つor3つに囲まれたONの状態のセルは、ONの状態のままです。
・それ以外のセルは、OFFの状態になります。

このルールに基づいて、列の再生が一周したら各セルのON/OFFの状態を切り替えるようにします。
まず、各列のBeatControllerにNoteControllerのリストを作り、その列のセルのNoteControllerをすべて追加します。
```csharp:BeatController.cs
    public NoteController[] notes;
```
![](/images/drum-life-game/image7.png)
次に、各セルのNoteControllerに、周囲にあるONの状態のセルの数を保存するための変数（count）を作ります。
```csharp:NoteController.cs
    [System.NonSerialized]
    public int count = 0;
```
そして、LifeGameManagerに以下の関数を追加します。
```csharp:LifeGameManager.cs
    private IEnumerator PlayBeats()
    {
        while (true)
        {
            foreach (BeatController beat in beats)
            {
                beat.PlayNotes();
                yield return new WaitForSeconds(0.3f);
            }
            // 以下の処理を追加
            NextLife(); // 列の再生が一周したら実行
        }
    }
    public void NextLife()
    {
        CountLife();    // 各セルの周囲にあるONの状態のセルを数える関数
        ChangeLife();   // 各セルのON/OFFの状態を切り替える関数
    }
```
CountLife()では、周囲にあるONの状態のセルの数をセルごとに計算してcountに保存します。
```csharp:LifeGameManager.cs
    private void CountLife()
    {
        for (int x = 0; x < beats.Length; x++)
        {
            if (x == 0) // 一番左の列（BEAT0）
            {
                NoteController[] _notes = beats[x].notes;       // その列（BEAT0）のNoteControllerのリスト
                NoteController[] _nextNotes = beats[x+1].notes; // 右の列（BEAT1）のNoteControllerのリスト

                for (int y = 0; y < _notes.Length; y++)
                {
                    // 周囲のONの状態のセルを数える
                    if (_nextNotes[y].isPlayed) _notes[y].count++;          // 右のセルがONの状態ならcountを1増やす 

                    if (y != 0) // 一番上の行以外のセルなら
                    { 
                        if (_notes[y-1].isPlayed) _notes[y].count++;        // 上のセルがONの状態ならcountを1増やす
                        if (_nextNotes[y-1].isPlayed) _notes[y].count++;    // 右上のセルがONの状態ならcountを1増やす
                    if (y != _notes.Length - 1) // 一番下の行以外のセルなら
                    { 
                        if (_notes[y+1].isPlayed) _notes[y].count++;        // 下のセルがONの状態ならcountを1増やす
                        if (_nextNotes[y+1].isPlayed) _notes[y].count++;    // 右下のセルがONの状態ならcountを1増やす
                    }                                       
                }                
            }
            else if (x == beats.Length - 1) // 一番右の列（BEAT15）
            {
                NoteController[] _prevNotes = beats[x-1].notes; // 左の列（BEAT14）のNoteControllerのリスト
                NoteController[] _notes = beats[x].notes;       // その列（BEAT15）のNoteControllerのリスト

                for (int y = 0; y < _notes.Length; y++)
                {
                    // 周囲のONの状態のセルを数える
                    if (_prevNotes[y].isPlayed) _notes[y].count++;          // 左のセルがONの状態ならcountを1増やす
                    
                    if (y != 0) // 一番上の行以外のセルなら
                    {
                        if (_prevNotes[y-1].isPlayed) _notes[y].count++;    // 左上のセルがONの状態ならcountを1増やす
                        if (_notes[y-1].isPlayed) _notes[y].count++;        // 上のセルがONの状態ならcountを1増やす
                    }
                    if (y != _notes.Length - 1) // 一番下の行以外のセルなら
                    {
                        if (_prevNotes[y+1].isPlayed) _notes[y].count++;    // 左下のセルがONの状態ならcountを1増やす  
                        if (_notes[y+1].isPlayed) _notes[y].count++;        // 下のセルがONの状態ならcountを1増やす
                    }                                       
                }                
            }
            else    // BEAT0・BEAT15以外の列
            {
                NoteController[] _prevNotes = beats[x-1].notes; // 左の列のNoteControllerのリスト
                NoteController[] _notes = beats[x].notes;       // その列のNoteControllerのリスト
                NoteController[] _nextNotes = beats[x+1].notes; // 右の列のNoteControllerのリスト

                for (int y = 0; y < _notes.Length; y++)
                {
                    // 周囲のONの状態のセルを数える
                    if (_prevNotes[y].isPlayed) _notes[y].count++;          // 左のセルがONの状態ならcountを1増やす
                    if (_nextNotes[y].isPlayed) _notes[y].count++;          // 右のセルがONの状態ならcountを1増やす

                    if (y != 0) // 一番上の行以外のセルなら
                    {
                        if (_prevNotes[y-1].isPlayed) _notes[y].count++;    // 左上のセルがONの状態ならcountを1増やす  
                        if (_notes[y-1].isPlayed) _notes[y].count++;        // 上のセルがONの状態ならcountを1増やす
                        if (_nextNotes[y-1].isPlayed) _notes[y].count++;    // 右上のセルがONの状態ならcountを1増やす
                    }
                    if (y != _notes.Length - 1) // 一番下の行以外のセルなら
                    {
                        if (_prevNotes[y+1].isPlayed) _notes[y].count++;    // 左下のセルがONの状態ならcountを1増やす
                        if (_notes[y+1].isPlayed) _notes[y].count++;        // 下のセルがONの状態ならcountを1増やす
                        if (_nextNotes[y+1].isPlayed) _notes[y].count++;    // 右下のセルがONの状態ならcountを1増やす
                    }                                       
                }
            }
        }
    }
```
最後にChangeLife()では、セルごとにcountの値に基づいてON/OFFの状態を決定します。
```csharp:LifeGameManager.cs
    private void ChangeLife()
    {
        for (int i = 0; i < beats.Length; i++)
        {
            NoteController[] _notes = beats[i].notes;

            for (int j = 0; j < _notes.Length; j++)
            {
                NoteController _note = _notes[j];

                // セルが元々OFFの状態かつ周囲にあるONの状態のセルが3つの場合、ONの状態にする
                if (!_note.isPlayed && _note.count == 3)
                {
                    _note.isPlayed =true;
                    _note.image.color = Color.white;
                }
                // セルが元々ONの状態かつ周囲にあるONの状態のセルが2つでも3つでもない場合、OFFの状態にする
                else if (_note.isPlayed && _note.count !=2 && _note.count != 3)
                {
                    _note.isPlayed = false;
                    _note.image.color = Color.black;
                }

                // countの値をリセットする
                _note.count = 0;
            }
        }
    }
```

# おわりに
Drum Life GameのARバージョンも制作しました。ARを活用することで、元々のアプリ体験を空間的に拡張しています。
https://youtube.com/shorts/xgSqND1ucXU
### Drum Life Game AR
iOS ver.
@[card](https://apps.apple.com/app/drum-life-game-ar/id6449294573)
Android ver.
@[card](https://play.google.com/store/apps/details?id=com.ekito_station.DrumLifeGameAR)