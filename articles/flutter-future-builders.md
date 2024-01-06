---
title: "【Flutter】FutureBuilderを使ってFirebase Firestoreから読み込んだデータを表示させる"
emoji: "🪩"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "firebase"]
published: false
---
# はじめに
FutureBuilderは非同期処理によってWidgetを生成するWidgetです。
私は現在FlutterでSNSアプリを開発しており、非同期でFirebase Firestoreから読み込んだデータを画面に表示したい時にFutureBuilderを使っています。この記事では、そのやり方をメモしています。
# FutureBuilderを使ってFirestoreから読み込んだデータを表示させる方法
たとえば、とあるユーザのデータを表示したい場合は以下のように使います。
```dart
@override
Widget build(BuildContext context) {
    return Scaffold(
        body: FutureBuilder<DocumentSnapshot>(
            future: FirebaseFirestore.instance
                .collection('users')     // ユーザのドキュメントのコレクション
                .doc('user1')            // とあるユーザのドキュメント
                .get(),
            builder: (context, snapshot) {
                if (snapshot.connectionState == ConnectionState.waiting) {
                    return CircularProgressIndicator(); // データが読み込まれるまでローディングを表示
                }
                if (snapshot.hasError) {
                    return Text('エラーが発生しました: ${snapshot.error}');
                }
                if (!snapshot.hasData || !snapshot.data!.exists) {
                    return Text('ドキュメントが見つかりません');
                }
                // Firestoreから取得したユーザのドキュメントのデータを表示
                var data = snapshot.data!.data() as Map<String, dynamic>;
                return Column(
                    children: [
                        Text('ユーザネーム: ${data['user_name']}'),
                        Text('自己紹介: ${data['introduction']}'),
                    ],
                );
            }
        )
    );
}
```
# 複数のFutureBuilderを使う方法
ここで別のユーザのデータも表示させたい場合、Firestoreから二回データを読み込む必要がある、つまりFutureBuilderを二回使う必要があることになります。
その場合は、以下のようにFutureBuilderをネストして使います。
```dart
@override
Widget build(BuildContext context) {
    return Scaffold(
        body: FutureBuilder<DocumentSnapshot>(
            future: FirebaseFirestore.instance
                .collection('users')     // ユーザのドキュメントのコレクション
                .doc('user1')            // 1人目のユーザのドキュメント
                .get(),
            builder: (context, snapshot1) {
                return FutureBuilder<DocumentSnapshot>(
                    future: FirebaseFirestore.instance
                        .collection('users')     // ユーザのドキュメントのコレクション
                        .doc('user2')            // 2人目のユーザのドキュメント
                        .get(),
                    builder: (context, snapshot2) {
                        if (snapshot1.connectionState == ConnectionState.waiting || snapshot2.connectionState == ConnectionState.waiting) {
                            // どちらかのFutureが実行中の場合
                            return CircularProgressIndicator();
                        }
                        if (snapshot1.hasError || snapshot2.hasError) {
                            // どちらかのFutureがエラーの場合
                            return Text('エラーが発生しました');
                        }
                        if (!snapshot1.hasData || !snapshot1.data!.exists || !snapshot2.hasData || !snapshot2.data!.exists) {
                            // どちらかのドキュメントが見つからない場合
                            return Text('ドキュメントが見つかりません');
                        }

                        // Firestoreから取得したユーザのドキュメントのデータを表示
                        var data1 = snapshot1.data!.data() as Map<String, dynamic>;
                        var data2 = snapshot2.data!.data() as Map<String, dynamic>;
                        return Column(
                            children: [
                                // 1人目のユーザのデータ
                                Text('User1'),
                                Text('ユーザネーム: ${data1['user_name']}'),
                                Text('自己紹介: ${data1['introduction']}'),
                                // 2人目のユーザのデータ
                                Text('User2'),
                                Text('ユーザネーム: ${data2['user_name']}'),
                                Text('自己紹介: ${data2['introduction']}'),
                            ],
                        );
                    }
                );
            }
        )
    );
}
```