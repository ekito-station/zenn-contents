---
title: "影だけ表示される透明なオブジェクトをつくる"
emoji: "🫥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity"]
published: true
---
# はじめに
バーチャル空間に透明なオブジェクトと動き回る光を配置し、オブジェクトの影を表示させる映像を作りました。

https://www.youtube.com/watch?v=Lw1pKM1sac8

この記事では、このようにオブジェクトを透明にしてその影だけ表示させる方法をまとめます。

# 1. Cast ShadowsをShadows Onlyに設定する
1番簡単な方法はこちらになります。
![Cast Shadows](/images/transparent-with-shadow/image1.png =500x)
オブジェクトの`Mesh Renderer > Lighting > Cast Shadows`を`Shadows Only`に設定することで、そのオブジェクトを透明にする＆影だけを表示させることができます。

# 2. 透明になるシェーダを作る
こちらの方法は、オブジェクトを透明にするだけでなく半透明にすることもできるので、より応用が利く方法です。

こちらの記事が参考になります。

https://nn-hokuson.hatenablog.com/entry/2016/10/05/201022

Surface Shaderを使用して、次のようなシェーダを作成します。

```hlsl:OnlyShadow.shader
Shader "Custom/OnlyShadow" {
	SubShader {
		Tags { "Queue" = "Transparent" } // Transparentを指定
		LOD 200

		CGPROGRAM
		#pragma surface surf Standard alpha:fade // alpha:fadeを追加 
		#pragma target 3.0

		struct Input {
			float2 uv_MainTex;
		};

		void surf (Input IN, inout SurfaceOutputStandard o) {
			o.Alpha = 0.0f;	// 透明度を指定
		}
		ENDCG
	}
	FallBack "Diffuse" // 影を描画
}
```
o.Alphaに値を代入することで、透明度を指定できます。0にすれば完全に透明になります。
Fallbackに"Diffuse"を指定することで、影は描画してくれるようになります。

シェーダについてはまだまだ勉強を始めたばかりなので、不正確な記述があればご指摘いただけると幸いです。


