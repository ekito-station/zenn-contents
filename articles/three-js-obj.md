---
title: "Three.jsを使ってOBJファイルを表示させる"
emoji: "🚥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["threejs"]
published: true
---
# はじめに
Three.jsを使って、以下のようにwebブラウザでOBJファイルを表示させる方法についてメモ。
![Demo Movie](/images/three-js-obj/image1.gif)

# コード全体
[こちらのGitHubのレポジトリ](https://github.com/ekito-station/building-viewer-threejs)でもコードを確認できます。
```html:index.html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <script src="https://unpkg.com/three@0.147.0/build/three.min.js"></script>
        <script src="https://unpkg.com/three@0.147.0/examples/js/controls/OrbitControls.js"></script>
        <script src="https://unpkg.com/three@0.147.0/examples/js/loaders/OBJLoader.js"></script>
        <script src="index.js"></script>
    </head>
    <body>
        <canvas id="myCanvas"></canvas>
    </body>
</html>
```
```js:index.js
window.addEventListener("DOMContentLoaded", init);

function init() {
    const width = 960;
    const height = 540;

    // レンダラーを作成
    const canvasElement = document.querySelector('#myCanvas');
    const renderer = new THREE.WebGLRenderer({
        antialias: true,
        canvas: canvasElement,
    });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(width, height);

    // シーンを作成
    const scene = new THREE.Scene();

    // カメラを作成
    const camera = new THREE.PerspectiveCamera(45, width / height, 1, 10000);
    camera.position.set(0, 0, 600);

    // カメラコントローラーを作成
    const controls = new THREE.OrbitControls(camera, canvasElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.2;

    // 環境光源を作成
    const ambientLight = new THREE.AmbientLight(0xffffff);
    ambientLight.intensity = 0.5;
    scene.add(ambientLight);

    // 平行光源を作成
    const directionalLight = new THREE.DirectionalLight(0xffffff);
    directionalLight.intensity = 1;
    directionalLight.position.set(1, 3, 1);
    scene.add(directionalLight);

    // 3Dモデルの読み込み
    const objLoader = new THREE.OBJLoader();
    objLoader.load(
        'Building.obj',
        function (obj) {
            scene.add(obj);
            obj.position.x = -50;
            obj.position.y = -100;
        },
    );

    tick();

    function tick() {
        renderer.render(scene, camera); // レンダリング
        requestAnimationFrame(tick);
    }
}
```

# 1. HTMLファイルを作成
HTMLファイル内で、以下の外部スクリプトを読み込みます。
- three.min.js
- OrbitControls.js（カメラを制御するためのライブラリ）
- OBJLoader.js（OBJファイルを読み込むためのライブラリ）
- index.js（これから作成するJavaScriptファイル）
```html
<script src="https://unpkg.com/three@0.147.0/build/three.min.js"></script>
<script src="https://unpkg.com/three@0.147.0/examples/js/controls/OrbitControls.js"></script>
<script src="https://unpkg.com/three@0.147.0/examples/js/loaders/OBJLoader.js"></script>
<script src="index.js"></script>
```
また、OBJファイルを表示させる描画エリアとなる`canvas`を用意します。
```html
<body>
    <canvas id="myCanvas"></canvas>
</body>
```
# 2. JavaScriptファイルを作成
## 2-1. 描画に必要な要素（レンダラーなど）を作成
3Dモデルの描画に必要なレンダラー、シーン、カメラなどを作成していきます。
マウス操作でカメラを制御できるカメラコントローラーについては、`enableDamping`や`dampingFactor`を以下のように設定することで、カメラの動きを滑らかにできます。
```js
const controls = new THREE.OrbitControls(camera, canvasElement);
controls.enableDamping = true;
controls.dampingFactor = 0.2;
```
また、自分は3Dモデルを全体的に明るく見せつつ立体感も出したかったので、環境光源と平行光源の両方を作成しました。環境光源だけだと3D空間全体に均等に光が当たりますが、陰影が生まれません。
```js
// 環境光源を作成
const ambientLight = new THREE.AmbientLight(0xffffff);
ambientLight.intensity = 0.5;
scene.add(ambientLight);

// 平行光源を作成
const directionalLight = new THREE.DirectionalLight(0xffffff);
directionalLight.intensity = 1;
directionalLight.position.set(1, 3, 1);
scene.add(directionalLight);
```
## 2-2. OBJファイルの読み込み
OBJLoaderを使用して、OBJファイルを読み込みます。以下の例における`'Building.obj'`の箇所にOBJファイルのパスを指定します。
```js
const objLoader = new THREE.OBJLoader();
objLoader.load(
    'Building.obj',
    function (obj) {
        scene.add(obj);
        obj.position.x = -50;
        obj.position.y = -100;
    },
);
```
また、3Dモデルの位置（`obj.position`）あるいはカメラの位置（`camera.position`）を調整することで、webページを最初開いた時に3Dモデルをどこからどの角度で見せたいかを指定します。

## 2-3. アニメーションを実装
最後に、フレームごとにレンダリングを行う処理を追加することで、カメラから見た3Dモデルの様子が常に更新されるようになります。これによって、マウス操作でカメラの位置を動かすと3Dモデルが動いて見えるようになります。
```js
tick(); // 初回実行

function tick() {
    renderer.render(scene, camera); // レンダリング
    requestAnimationFrame(tick);    // 毎フレーム実行
}
```

