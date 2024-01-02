---
title: "【Flutter】Firebase Storageから画像をダウンロードして表示する"
emoji: "🖼️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "firebase"]
published: true
---
Flutterでアプリ開発するなかで、Firebase Storageに保存した画像をダウンロードしてアプリ内に表示させる方法を考えたのでメモします。
Firebaseプロジェクトの作成方法については割愛します。

# 画像をダウンロードする関数を用意する
まず、Firebase Storageから画像をダウンロードする関数を用意します。
ここで[cached_network_image](https://pub.dev/packages/cached_network_image)パッケージを活用して、インターネットから取得した画像をキャッシュして表示を高速化させます。
https://pub.dev/packages/cached_network_image

```dart
// cached_network_imageパッケージをインポート
import 'package:cached_network_image/cached_network_image.dart';
```
このパッケージも活用して以下のような_downloadImageという関数を作成します。
```dart
ImageProvider? _image;
bool _isImageLoaded = false; // 画像をすでにダウンロードしたかどうかの判定用

Future<void> _downloadImage(String url) async {
    if (!_isImageLoaded) {　// 画像をまだダウンロードしていない場合に処理を実行
        // Firebase Storage上の画像のURLを取得
        String downloadURL =
            await FirebaseStorage.instance.ref(url).getDownloadURL();
        // URLから画像を取得
        final imgProvider = CachedNetworkImageProvider(downloadURL);
        setState(() {
            _image = imgProvider;
            _isImageLoaded = true;
        });
    }
}
```

# ダウンロードした画像を表示する
以上の関数を用いて、画面上に画像を表示させます。
このとき、画像が読み込まれるまで（つまり_imageがnullになっている間）CircularProgressIndicatorを表示させるようにします。
```dart
@override
Widget build(BuildContext context) {
    if (!_isImageLoaded) { // 画像をまだダウンロードしていない場合に関数を実行
        // 引数にFirebase Storage上の画像のURL（例: 'Images/sample.png'）を渡す
        _downloadImage('画像のURL');
    }
    return Scaffold(
        body: Center(
            child: _image == null
                ? CircularProgressIndicator() // _imageがnullになっている間はグルグルを表示
                : Container(
                    decoration: BoxDecoration(
                        image: DecorationImage(
                            fit: BoxFit.cover,
                            image: _image!
                        )
                    )
                ),
        ),
    );
}
```