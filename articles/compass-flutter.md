---
title: "スマホがある地点の方角を向いた時にWidgetを表示させる（Flutter）"
emoji: "🧭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, dart, 個人開発]
published: true
---
# はじめに
Flutterを使って、スマホがある目標地点の方角を向いた時にWidgetを表示させるアプリケーションを作りました。
![アプリケーションのイメージ](/images/compass-flutter/exp.png)
*▲ アプリケーションのイメージ*

# コード全体
[こちらのGitHub](https://github.com/ekito-station/compass-flutter/blob/main/lib/main.dart)でも確認できます。
:::details コード全体
```dart:main.dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:flutter_compass/flutter_compass.dart';
import 'package:geolocator/geolocator.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Compass Flutter',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const MyHomePage(title: 'Compass Flutter'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  // ユーザの位置
  Position myPosition = Position(
    // 初期座標（ひとまず渋谷駅の座標）
    latitude: 35.658199,
    longitude: 139.701625,
    timestamp: DateTime.now(),
    altitude: 0,
    accuracy: 0,
    heading: 0,
    speed: 0,
    speedAccuracy: 0,
    floor: null,
  );
  late StreamSubscription<Position> myPositionStream;
  late double? deviceDirection;

  // 目標地点の位置
  Position markerPosition = Position(
    // （例）新宿駅の座標
    latitude: 35.689702,
    longitude: 139.700560,
    timestamp: DateTime.now(),
    altitude: 0,
    accuracy: 0,
    heading: 0,
    speed: 0,
    speedAccuracy: 0,
    floor: null,
  );
  late double markerDirection;
  double directionTolerance = 5.0;

  final LocationSettings locationSettings = const LocationSettings(
    accuracy: LocationAccuracy.high,
    distanceFilter: 10,
  );

  // startPositionから見たendPositionの方角を計算
  double calcDirection(Position startPosition, Position endPosition) {
    double startLat = startPosition.latitude;
    double startLng = startPosition.longitude;
    double endLat = endPosition.latitude;
    double endLng = endPosition.longitude;
    double direction =
        Geolocator.bearingBetween(startLat, startLng, endLat, endLng);
    if (direction < 0.0) {
      direction += 360.0;
    }
    return direction;
  }

  // direction1とdirection2の差が閾値未満か確認
  bool checkTolerance(double direction1, double direction2) {
    if ((direction1 - direction2).abs() < directionTolerance) {
      return true;
    } else if ((direction1 - direction2).abs() > 360 - directionTolerance) {
      return true;
    } else {
      return false;
    }
  }

  @override
  void initState() {
    super.initState();

    // 位置情報サービスが許可されていない場合は許可をリクエストする
    Future(() async {
      LocationPermission permission = await Geolocator.checkPermission();
      if (permission == LocationPermission.denied) {
        await Geolocator.requestPermission();
      }
    });

    // ユーザの現在位置を取得し続ける
    myPositionStream =
        Geolocator.getPositionStream(locationSettings: locationSettings)
            .listen((Position position) {
      myPosition = position;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      // FlutterCompassのイベントをlistenする
      body: StreamBuilder<CompassEvent>(
          stream: FlutterCompass.events,
          builder: (context, snapshot) {
            if (snapshot.hasError) {
              return Text('Error reading heading: ${snapshot.error}');
            }
            if (snapshot.connectionState == ConnectionState.waiting) {
              return const Center(
                child: CircularProgressIndicator(),
              );
            }

            // デバイスの向いている方角を取得
            deviceDirection = snapshot.data?.heading;
            if (deviceDirection == null) {
              return const Center(
                child: Text("Device does not have sensors."),
              );
            }

            // ユーザの現在位置から見た目標地点の方角を計算
            markerDirection = calcDirection(myPosition, markerPosition);

            return Center(
              child: Opacity(
                // デバイスの向いている方角と目標地点の方角の差が閾値未満の場合
                // opacityを1にしてwidgetを表示させる
                opacity: checkTolerance(deviceDirection!, markerDirection)
                    ? 1.0
                    : 0.0,
                child: const Icon(
                  Icons.expand_less,
                  size: 100,
                ),
              ),
            );
          }),
    );
  }
}
```
:::

# 1. 事前準備
## 1-1. パッケージインストール
[geolocator](https://pub.dev/packages/geolocator)と[flutter_compass](https://pub.dev/packages/flutter_compass)というパッケージをインストールします。
```sh
flutter pub add geolocator
flutter pub add flutter_compass
```

## 1-2. 位置情報サービスに関する権限設定
### iOSの場合
`ios/Runner/Info.plist`を下記のように編集します。
```diff plist:Info.plist
...
 <dict>
+	<key>NSLocationWhenInUseUsageDescription</key>
+	<string>Your location is required for this app.</string>
...
```
### Androidの場合
`android/app/src/main/AndroidManifest.xml`を下記のように編集します。
```diff xml:AndroidManifest.xml
 <manifest xmlns:android="http://schemas.android.com/apk/res/android"
     package="com.example.compass_flutter">
+    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" /> 
...
```

# 2. 実装
## 2-1. ユーザの現在位置を継続的に取得する
ユーザの現在位置を格納する変数を設定します。
```dart
// ユーザの位置
Position myPosition = Position(
    // 初期座標（ひとまず渋谷駅の座標）
    latitude: 35.658199,
    longitude: 139.701625,
    timestamp: DateTime.now(),
    altitude: 0,
    accuracy: 0,
    heading: 0,
    speed: 0,
    speedAccuracy: 0,
    floor: null,
);
```
その他、以下のような必要な変数を設定します。
```dart
late StreamSubscription<Position> myPositionStream;
final LocationSettings locationSettings = const LocationSettings(
    accuracy: LocationAccuracy.high,
    distanceFilter: 10,
);
```
`Geolocator.getPositionStream`でユーザの現在位置を取得し続けます。
```dart
@override
void initState() {
    ...
    // ユーザの現在位置を取得し続ける
    myPositionStream =
        Geolocator.getPositionStream(locationSettings: locationSettings)
            .listen((Position position) {
        myPosition = position;
    });
}
```

## 2-2. ユーザの現在位置から見た目標地点の方角を計算する
目標地点の位置を格納する変数を設定します。
```dart
// 目標地点の位置
Position markerPosition = Position(
    // （例）新宿駅の座標
    latitude: 35.689702,
    longitude: 139.700560,
    timestamp: DateTime.now(),
    altitude: 0,
    accuracy: 0,
    heading: 0,
    speed: 0,
    speedAccuracy: 0,
    floor: null,
);
```
とある地点から見た、他のとある地点の方角を計算する関数を作ります。
```dart
// startPositionから見たendPositionの方角を計算する関数
double calcDirection(Position startPosition, Position endPosition) {
    double startLat = startPosition.latitude;
    double startLng = startPosition.longitude;
    double endLat = endPosition.latitude;
    double endLng = endPosition.longitude;

    double direction =
        Geolocator.bearingBetween(startLat, startLng, endLat, endLng);

    if (direction < 0.0) {
        direction += 360.0;
    }
    return direction;
}
```
上記の関数を使って、ユーザの現在位置から見た目標地点の方角を計算します。
```dart
late double markerDirection;
// ユーザの現在位置から見た目標地点の方角を計算
markerDirection = calcDirection(myPosition, markerPosition);
```

## 2-3. スマホの向いている方角を継続的に取得する
StreamBuilderを使ってFlutterCompassのイベントを継続的に監視し、snapshotからスマホの向いている方角を取得します。
```dart
late double? deviceDirection;

@override
Widget build(BuildContext context) {
    return Scaffold(
        ...
        body: StreamBuilder<CompassEvent>(
            stream: FlutterCompass.events,
            builder: (context, snapshot) {
                ...
                // デバイスの向いている方角を取得
                deviceDirection = snapshot.data?.heading;
        ...
```

## 2-4. スマホの向いている方角と目標地点の方角がほぼ同じだったらWidgetを表示させる
2つの方角の差が閾値未満だったらtrueを返す関数を作ります。
```dart
// ここでは、方角の差が5度未満だったらWidgetを表示させることに
double directionTolerance = 5.0;

// direction1とdirection2の差が閾値未満か確認する関数
bool checkTolerance(double direction1, double direction2) {
    // 差が5度未満ならtrueを返す
    if ((direction1 - direction2).abs() < directionTolerance) {
        return true;
    // 差が355度より大きくてもtrueを返す
    } else if ((direction1 - direction2).abs() > 360 - directionTolerance) {
        return true;
    // それ以外ならfalseを返す
    } else {
        return false;
    }
}
```
上記の関数を使って、Widgetのopacity（透明度）を変化させます。
opacity=1.0ならWidgetが表示され、opacity=0.0ならWidgetが透明になります。
```dart
@override
Widget build(BuildContext context) {
    return Scaffold(
        ...
        body: StreamBuilder<CompassEvent>(
            stream: FlutterCompass.events,
            builder: (context, snapshot) {
                ...
                return Center(
                    child: Opacity(
                        opacity: checkTolerance(deviceDirection!, markerDirection)
                            ? 1.0   // trueならopacity=1.0に
                            : 0.0,  // falseならopacity=0.0に
                        // 以下のWidget（矢印のアイコン）のopacityが1.0か0.0になる
                        child: const Icon(
                            Icons.expand_less,
                            size: 100,
                        ),
                    ),
                );
            }),
    );
}
```

# 3. おわりに
スマホを目標地点の方角に向けると、以下のように矢印が表示されるようになります。
![スクリーンショット](/images/compass-flutter/screenshot.jpg =250x)