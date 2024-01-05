---
title: "【Flutter】ボトムバーを常に表示させる"
emoji: "🔻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter"]
published: true
---
# はじめに
Flutterアプリ開発において、画面下にボトムバーを常に表示させたかったため[persistent_bottom_nav_bar](https://pub.dev/packages/persistent_bottom_nav_bar)というパッケージを導入しました。
![](/images/flutter-persistent-bottom-nav-bar/image1.jpg =250x)
パッケージのインストール方法はこちらのサイトに記載されています。
https://pub.dev/packages/persistent_bottom_nav_bar/install

# ボトムバーを配置する
まずパッケージをインポートします。
```dart
import 'package:persistent_bottom_nav_bar/persistent_tab_view.dart';
```
次に、Scaffoldのbodyプロパティに`PersistentTabView`を指定します。
```dart
@override
Widget build(BuildContext context) {
    return Scaffold(
        body: PersistentTabView(
            context,
            screens: ,
            items: []
        )
    );
}
```
PersistentTabViewのscreensプロパティに、ボトムバーを使って表示を切り替えたい画面のリストを指定します。
```dart
var _screens = [
    Page1(),
    Page2(),
    Page3()
];

@override
Widget build(BuildContext context) {
    return Scaffold(
        body: PersistentTabView(
            context,
            screens: _screens, // 画面のリストを指定
            items: []
        )
    );
}
```
itemsプロパティには、`PersistentBottomNavBarItem`のリストを指定します。このPersistentBottomNavBarItemがボトムバーに配置される各アイコンボタンになり、アイコン（`icon`）や選択時の色（`activeColorPrimary`）、非選択時の色（`inactiveColorPrimary`）を指定できます。
ちなみに`title`プロパティにテキストを指定すると、選択時にアイコン横にそのテキストが表示されるようになります。
```dart
@override
Widget build(BuildContext context) {
    return Scaffold(
        body: PersistentTabView(
            context,
            screens: _screens,
            items: [
                // Page1へのボタン
                PersistentBottomNavBarItem(
                icon: Icon(Icons.home),             // アイコン
                activeColorPrimary: Colors.white,   // 選択時の色
                inactiveColorPrimary: Colors.black, // 非選択時の色
                title: 'Home'                       // 選択時にアイコン横に表示されるテキスト
                ),
                // Page2へのボタン
                PersistentBottomNavBarItem(
                icon: Icon(Icons.mic),
                activeColorPrimary: Colors.white,
                inactiveColorPrimary: Colors.black,
                title: 'Record'
                ),
                // Page3へのボタン
                PersistentBottomNavBarItem(
                icon: Icon(Icons.person),
                activeColorPrimary: Colors.white,
                inactiveColorPrimary: Colors.black,
                title: 'Account'
                ),
            ],
        ),
    );
}
```
これで、常に表示されるボトムバーが配置できました。

# 画面遷移時にボトムバーを消す
ただ、画面遷移する際にボトムバーの表示を消したいシチュエーションもあると思います。その際は`PersistentNavBarNavigator.pushNewScreen`というメソッドを使います。このとき`withNavBar`という引数にfalseを指定することで、画面遷移時にボトムバーを消すことができます。
```dart
PersistentNavBarNavigator.pushNewScreen(
    context,
    screen: Page4(), // 遷移先の画面を指定
    withNavBar: false // falseに指定
);
```
