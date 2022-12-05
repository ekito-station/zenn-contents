---
title: "LineRendererで引いた線にBoxColliderを設置する（3D）"
emoji: "📊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity", "csharp"]
published: true
---
# 要約
1. LineRendererで原点からX軸方向に線を引く
2. BoxColliderを設置する
3. 線を本来の方向・位置に回転・移動させる
# コード全体
```csharp:LineRendererAndBoxCollider.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class LineRendererAndBoxCollider : MonoBehaviour
{
    void Start()
    {
        Vector3 startPos = new Vector3(1.0f, 1.0f, 5.0f);  // 始点
        Vector3 endPos = new Vector3(5.0f, 5.0f, 1.0f);    // 終点
        Vector3 lineVec = endPos - startPos;
        float dist = lineVec.magnitude;
        Vector3 lineX = new Vector3(dist, 0f, 0f);        

        // LineRendererで原点からX軸方向に線を引く
        GameObject line = new GameObject("Line");
        LineRenderer lineRenderer = line.AddComponent<LineRenderer>();
        lineRenderer.useWorldSpace = false;
        Vector3[] linePositions = new Vector3[] {Vector3.zero, lineX};
        lineRenderer.SetPositions(linePositions);

        // BoxColliderを設置する
        BoxCollider col = line.AddComponent<BoxCollider>();

        // 線を本来の方向・位置に回転・移動させる
        line.transform.rotation = Quaternion.FromToRotation(lineX, lineVec);
        line.transform.position += startPos;   
    }
}
```
# 記事の目的
　Unityの3D空間にLineRendererで線を引いて、その線にBoxColliderを設置する方法を解説します。試しに、LineRendererで引いた線にそのままBoxColliderを設置してみます。
```csharp
Vector3 startPos = new Vector3(1.0f, 1.0f, 5.0f);  // 始点
Vector3 endPos = new Vector3(5.0f, 5.0f, 1.0f);    // 終点

GameObject line = new GameObject("Line");
LineRenderer lineRenderer = line.AddComponent<LineRenderer>();        
Vector3[] linePositions = new Vector3[] {startPos, endPos};
lineRenderer.SetPositions(linePositions);

BoxCollider col = line.AddComponent<BoxCollider>(); // BoxColliderを設置
```
すると、BoxColliderの形は下図の緑部分のようになってしまいます（Physics DebuggerでBoxColliderを可視化しています）。BoxColliderは向きを指定できないため、回転しないまま線全体を包もうとして、無駄に大きくなってしまいます。
![](/images/linerenderer-boxcollider/image1.png)
　BoxColliderを回転させて線にフィットするように設置するためには、工夫が必要です。具体的には、まずX軸方向に線を引いてBoxColliderを設置した後で、本来の方向に回転させるという方法でやります。
# 1. LineRendererで原点からX軸方向に線を引く
　最初に、引きたい線の長さを取得します。
```csharp
Vector3 startPos = new Vector3(1.0f, 1.0f, 5.0f);  // 始点
Vector3 endPos = new Vector3(5.0f, 5.0f, 1.0f);    // 終点
Vector3 lineVec = endPos - startPos;
float dist = lineVec.magnitude; // 引きたい線の長さ
```
その長さだけX軸方向に伸びているベクトルを作成します。
```csharp
Vector3 lineX = new Vector3(dist, 0f, 0f);
```
次に、LineRendererを用意します。
```csharp
GameObject line = new GameObject("Line");   // 空のゲームオブジェクトを作成
LineRenderer lineRenderer = line.AddComponent<LineRenderer>();  // LineRendererをアタッチ
```
LineRendererで原点からX軸方向に、本来引きたい線の長さだけ伸びる線を引きます。
```csharp
lineRenderer.useWorldSpace = false; // ローカル座標系に切り替える
Vector3[] linePositions = new Vector3[] {Vector3.zero, lineX};
lineRenderer.SetPositions(linePositions);
```
useWorldSpaceをfalseにしてLineRendererの座標系をローカル座標系に切り替えているのは、あとで線を移動させるためです。もしtrueのまま（=ワールド座標系のまま）だと、LineRendererをアタッチしたオブジェクトのTransformを変化させても、SetPositionsで設定した線の頂点の位置は変化しません。つまり、線を回転・移動させられなくなってしまいます。
# 2. BoxColliderを設置する
　このタイミングで、線にBoxColliderを設置します。すると、線の方向・位置はともかく、線にフィットするようにBoxColliderが設置されます。
![](/images/linerenderer-boxcollider/image2.png)
# 3. 線を本来の方向・位置に回転・移動させる
　そして、原点からX軸方向に伸びていた線を、本来の始点から本来の方向に伸びるように直します。FromToRotationを使って線をX軸方向から本来の方向へ回転させ、原点（現時点での線の始点）が本来の始点と重なるように線全体を移動させます。
```csharp
line.transform.rotation = Quaternion.FromToRotation(lineX, lineVec);    // 本来の方向に
line.transform.position += startPos;    // 本来の位置に
```
これによって、BoxColliderも線と一緒に回転・移動します。以上の手順で、LineRendererで引いた線にフィットするBoxColliderを設置することができました！
![](/images/linerenderer-boxcollider/image3.png)