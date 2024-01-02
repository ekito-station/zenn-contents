---
title: "【Flutter】ページを戻る時にデータを渡す"
emoji: "⏪"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter"]
published: true
---
# （通常の）ページを戻るやり方
Flutterにおいて、ページ遷移してから前のページに戻るコードは通常以下のようになります。
```dart
// ページ遷移する処理
Navigator.push(context, MaterialPageRoute(
    builder: (context) => NextPage()
));
```
```dart
// 前のページに戻る処理
Navigator.pop(context);
```
# データを渡しながらページを戻るやり方
前のページに戻る際に遷移元のページにデータを渡したい時は、以下のようにコードを修正します。
```dart
// ページ遷移する処理＆戻ってきた後の処理
Navigator.push(context, MaterialPageRoute(
    builder: (context) => NextPage()
)).then((value) {
    // 遷移先のページからのデータがvalueに渡される
    if (value != null) {
        // 遷移先のページからのデータ（value）を使った処理
        ...
    }
})
```
```dart
// データを渡しながら前のページに戻る処理
var value;  // 渡したいデータ
Navigator.pop(context, value);
```

# 複数のデータを渡しながらページを戻るやり方
複数のデータを渡したい時は、Mapを使用します。
```dart
// ページ遷移する処理＆戻ってきた後の処理
Navigator.push(context, MaterialPageRoute(
    builder: (context) => NextPage()
)).then((values) {
    // 遷移先のページからのデータがvaluesに渡される
    if (values != null) {
        // 遷移先のページからのデータ（values）を使った処理
        dynamic value1 = values['key1'];
        dynamic value2 = values['key2'];
        ...
    }
})
```
```dart
// データを渡しながら前のページに戻る処理
Map<String, dynamic> values = {
  'key1': value1,
  'key2': value2,
};
Navigator.pop(context, values);
```