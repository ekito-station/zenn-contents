---
title: "【Flutter】Containerの中央にテキスト、右端にボタンを表示させる"
emoji: "🔘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter"]
published: true
---
# はじめに
現在FlutterでSNSアプリを作っています。
アカウントページを作成する際に、
ユーザネームを表示するヘッダーを下図のように作成しました。
![](/images/flutter-center-title-right-icon/image1.jpg)
このようなヘッダーを作成するために、
Containerの中に以下のようにWidgetを配置する方法をまとめます。
- 中央にText（ユーザネーム）
- 右端にIconButton（設定ボタン）
- 左端には何もなし

# Rowを用いる
まず、各Widgetを横に並べるためにRowを用います。
Widgetを均等に並べるために、
`mainAxisAlignment`プロパティには`MainAxisAlignment.spaceEvenly`を指定します。
これによって、Widget間にスペースが均等に配置されます。
```dart
Row(
    mainAxisAlignment: MainAxisAlignment.spaceEvenly,
    children: [
        // 各Widgetを配置
    ]
)
```
# onPressedがnullのIconButtonを配置する
次にRowの`children`の中に、各Widgetを配置します。
ここでTextを中央に配置するためには、左端にも右端のIconButtonと同じ大きさのWidgetを配置しなければなりません。
そこで、`onPressed`プロパティをnullにしてボタンとしての機能を無効化したIconButtonを左端にも配置します。これで、左端にIconButtonと同じ大きさの空白が生まれます。
```dart
IconButton(
    icon: Icon(Icons.settings), // 適当なアイコン
    onPressed: null, // これによってボタンが無効化
),
```
# おわりに
全体のコードは以下のようになります。
```dart
Container(
    child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceEvenly,
        children: [
            // 左端の空白
            IconButton(
                icon: Icon(Icons.settings), // 適当なアイコン
                onPressed: null, // これによってボタンが無効化
            ),
            // 中央に配置するText
            Text('Text'),
            // 右端に配置するIconButton
            IconButton(
                icon: Icon(Icons.settings),
                onPressed: () {
                    // ボタンを押した際の処理
                },
            )
        ],
    )
),
```