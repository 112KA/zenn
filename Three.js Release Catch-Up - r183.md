いつの間にか知らない機能が追加されていたり書き方が変わっているthreejsをキャッチアップする目的で、主にexamplesの更新分についてAIの力を借りた要約＆不明な用語・判断を調べながら各詳細を記録していく。
## Overview
リリース日: 2026/02/19
公式リリースノート: [Release r183 · mrdoob/three.js · GitHub](https://github.com/mrdoob/three.js/releases/tag/r183)
Migration-Guide: [Migration Guide · mrdoob/three.js Wiki · GitHub](https://github.com/mrdoob/three.js/wiki/Migration-Guide#182--183)

> [!NOTE] Point
> 特に以下2点はmigration時にコードの修正箇所が多くなりそう。
- `Clock` から より高機能な `Timer` クラスに置き換わります。
	- [[#Timerの機能（Clockとの違い）]]
- WebGPU/Nodeベースのポストプロセス管理クラスが`PostProcessing` から `RenderPipeline`に名称変更。

## New Featured Example Note
### Updated List
#### 1. [https://threejs.org/examples/#webgl_shaders_ocean](https://t.co/SnmmjUo8El)
リアルな波の質感を表現し、空の反射や光の屈折をシミュレーションして、美しい水面を描画するシェーダーのデモ。
r183で「静的な水面＋空」から「HDR＋ブルーム＋雲制御＋時間変化を持つ、より映画的なシーン」へ表現が拡張された。
::: details details
- **描画品質/HDRパイプラインの強化**
    - `WebGLRenderer` が `outputBufferType: THREE.HalfFloatType` になり、明るさ情報の表現力が向上。
	    - [[#何故HalfFloatTypeか]]
    - `UnrealBloomPass` を導入
- **露出（Exposure）の調整性向上**
    - `toneMappingExposure` を `0.5 -> 0.1` に変更し、白飛びしにくい初期値へ。
    - GUIに `exposure` スライダー追加。
- **空表現の拡張**
    - `Sky` の uniform に `cloudCoverage / cloudDensity / cloudElevation` を追加。
    - 毎フレーム `sky.material.uniforms['time'].value = time;` で更新。空（雲）の時間変化表現に対応。

:::

---

#### 2. [https://threejs.org/examples/#webgpu_volume_lighting_traa](https://t.co/sKEPxqG2ux)
r183で追加された**TRAAノードとボリューメトリック照明 ** を組み合わせたデモ。
[[#TRAAとは]]

::: details details
- ボリューム本体は createTexture3Dで生成した `Data3DTexture` を使い、`VolumeNodeMaterial` の `scatteringNode` で密度をサンプリング。
- 描画は `RenderPipeline` で分離し、以下の順で合成。
    1. `depthPass`（不透明物の深度）
    2. pass + mrt(output, velocity（カラー+モーションベクトル）
    3. traa
- 時間方向の安定化として、halton 由来の32サンプルオフセットを毎フレーム更新し、TRAAの蓄積と整合させています。
	[[#Haltonとは]]

:::

---

#### 3. [https://threejs.org/examples/#webgpu_postprocessing_ssr](https://t.co/iyruSamhrx) 

SSR(Screen Space Reflection)を紹介するデモ。
主な改善点は、**SSRの見え方の安定化**と**r183系APIへの追従**です。

::: details details

- **SSR合成の修正**
    
    - `blendColor( scenePassColor, ssrPass )` をやめて  
        `scenePassColor.add( ssrPass.rgb )` に変更。
    - コメントにもある通り、**SSRがpremultiplied colorを出す前提**での加算合成にしており、反射の不自然な暗化/色ズレを避けやすくなっています。
- **PostProcessing → RenderPipeline**
    
    - `THREE.PostProcessing` から `THREE.RenderPipeline` に移行。
    - `outputNode` の切替や `needsUpdate`、`render()` 呼び出し先も統一され、現在のWebGPUノード構成に合わせた実装になっています。
- **SSR確認しやすいシーン構成に改善**
    
    - カメラ下に反射面（円形メッシュ）を追加して、SSR効果の確認がしやすくなっています。
- **SSRパラメータのチューニング**
    
    - `blurQuality: 2 -> 1`
    - `maxDistance: 0.5 -> 1`
    - `thickness: 0.015 -> 0.03`
    - 反射の届く範囲やヒット判定の安定性を上げつつ、コストを抑える方向の調整です。
- **環境マップ生成の調整**
    
    - `pmremGenerator.fromScene( environment, 0.04 )` に変更し、環境反射の粗さ/拡散感を微調整しています。

:::

---

#### 4. [https://threejs.org/examples/#webgl_animation_keyframes](https://t.co/WGAagzP4DU) 

見た目の質向上
THREE.WebGLRendererにtoneMapping = THREE.ACESFilmicToneMapping を設定し、HDR環境光の見え方を改善。
環境ライティングの刷新
Skyを使って空から環境テクスチャを生成し、THREE.PMREMGenerator経由で scene.background / scene.environment に適用。
旧 RoomEnvironment より、シーン雰囲気を調整しやすい構成。
時間管理の更新
THREE.ClockからTHREE.Timerへ移行し、animateでtimer.update() + timer.getDelta() を利用。
timer.connect( document ) により、タブ状態などの影響を受ける実運用向け挙動に寄せている。
操作感の微調整
OrbitControls のtarget を (0, 0.7, 0) に変更、enableDamping を維持してより自然なカメラ操作に。
不要処理の整理
初期化順・改行・不要設定を整理し、読みやすさを改善。

r183 新規機能への言及
r183リリース文脈でこのパッチが直接活用しているのは、主に以下です。

THREE.Timer ベースの更新ループ
Skyの拡張ユニフォーム（例: cloudElevation）を使った環境生成

---

#### 5. [https://threejs.org/examples/#webgpu_parallax_uv](https://t.co/yeVDHHSVBG) 

環境ライティング品質の向上
キューブマップから HDR equirectangular に変更し、反射の一貫性を改善。scene.backgroundBlurriness も追加され、背景と被写界感のバランス調整が可能。
関連: THREE.EquirectangularReflectionMapping scene, backgroundBlurriness, HDRLoader

パララックス表現の制御性アップ
scaleUV と parallaxScale を uniform 化し、UVタイル密度と視差量をリアルタイム調整可能に。
関連: uniform, parallaxUV, uv

ノーマル処理のNode化
material.normalMap から material.normalNode = normalMap(...) へ移行し、TSLノードパイプラインに統一。roughness も同じ scaledUV を使用して質感整合性が向上。
関連: normalMap, THREE.MeshStandardNodeMaterial

テクスチャ設定の明確化
top/roughness/normal/displacement すべてに RepeatWrapping を設定し、UVスケール変更時の破綻を防止。
関連: THREE.RepeatWrapping

シーン構成の意図明確化
ジオメトリを Box から Circle に変更し、地面材質デモとして視認性を改善。
関連: THREE.CircleGeometry

デバッグ/調整UIの追加
Inspector を導入し、背景ぼかし・視差量・UVスケールをGUIで即時調整可能。
関連: Inspector, renderer.inspector

r183 の新規機能への言及（この差分で使っているもの）
Inspector アドオン活用（WebGPU系サンプルでの新しい調整フロー）
renderer.inspector と createParameters() を使ったランタイムUI追加は、r183系のサンプル方針に沿った更新です。
関連: Inspector

TSL/Nodeマテリアル運用の強化方向
normalNode + normalMap() + uniform() を使う形は、r183で進んだWebGPU/TSL中心の記述スタイルに合致しています。

---

#### 6. [https://threejs.org/examples/#webgl_postprocessing_godrays

- **点光源からのゴッドレイ（体積光っぽい光条）**をポストプロセスで表現するデモです。
- three.js本体だけでなく、外部ライブラリの
    - `postprocessing`（`EffectComposer`, `RenderPass`）
    - `three-good-godrays`（`GodraysPass`）  
        を使っています。

##### 処理の流れ

1. **シーン準備**
    
    - `GLTFLoader` で `godrays_demo.glb` を読み込み。
    - `concrete`, `base` のマテリアルを差し替えて暗めに統一。
    - `setupBackdrop()` で周囲に大きい面を配置し、光の見え方を安定化。
2. **ライトと影**
    
    - PointLightを高強度で配置し、`castShadow = true`。
    - シーン内メッシュを走査して castShadow/receiveShadow を有効化。
    - 発光源の可視化用に小さい白球を置いています。
3. **ポストプロセス**
    
    - `EffectComposer` を作成（`frameBufferType: HalfFloatType`）。
    - `RenderPass` で通常レンダリング結果を作成。
    - `GodraysPass` を追加して光条を合成。
    - `renderToScreen = true` で最終出力。
4. **操作と描画**
    
    - `OrbitControls` でカメラ操作。
    - `animate()` で `composer.render()` を毎フレーム実行。

##### Godraysパラメータの意味（主要）

- `density`: レイの密度（細かさ）
- `maxDensity`: 光の最大濃度
- `edgeStrength`, `edgeRadius`: エッジ付近の強調
- `distanceAttenuation`: 距離減衰
- color: 光の色
- `raymarchSteps`: サンプリング回数（品質/負荷トレードオフ）
- `blur`: ぼかし有無
- `gammaCorrection`: ガンマ補正

##### r183で新規追加された機能の利用について

- このファイルで**r183新規機能として明確に使っているのは「このデモ自体の追加」**です。
- 実装の中心である `GodraysPass` は **three.js本体の新APIではなく外部拡張** です。
- `GLTFLoader` / `OrbitControls` / `WebGLRenderer` / PointLight などは既存機能の利用です。

---

#### 7. [https://threejs.org/examples/#webgl_effects_anaglyph](https://t.co/FDIuWc0rcP) 

この差分の改善点は、**`AnaglyphEffect` のステレオ設定を明示的に行っていること**です。

- `effect.eyeSep = 0.064;`
    
    - 両眼間距離（IPD）を 64mm 相当に固定。
    - 観察者差の影響を減らし、デフォルト任せより**意図した立体感**を作りやすくなります。
- `effect.planeDistance = 3;`
    
    - ゼロ視差面（スクリーン奥行きに見える基準面）をカメラ配置に合わせて指定。
    - この例ではシーン中心をスクリーン面に近づける意図で、**立体の飛び出し/引っ込み量を制御**しやすくなっています。

##### r183 新機能との関係

この変更は、r183で使える `AnaglyphEffect` の新しい調整項目（`eyeSep` / `planeDistance`）を活用したものです。  
そのため、r183を前提にすると次の利点があります。

- 立体感の強さを定量的にチューニング可能
- 視差の基準面を明示でき、見え方の破綻を減らせる
- 「物理的に妥当な値」に寄せた設定が可能
 
#### 8. [https://threejs.org/examples/#webgpu_postprocessing_retro](https://t.co/t13VJbZEyl) 

- WebGPU + TSL で、**PS1風のレトロ画面**をポストプロセスで作るデモです。
- シーン自体（モデル/煙/ライト）を描いたあと、画面全体に retro 系エフェクトを段階適用します。

###### 構成の流れ（実装順）

1. **シーン描画**
    
    - THREE.WebGPURenderer
    - THREE.RenderPipeline
    - `pass( scene, camera )` で通常レンダーパス
2. **背景ノード**
    
    - `scene.backgroundNode` に Fn() で生成した空グラデーション＋疑似星空を設定
    - `normalWorld` / atan / `smoothstep` / `fract` などで手続き生成
3. **被写体**
    
    - GLTF を切替ロード（Coffee Mug / Damaged Helmet）
    - Mug のときだけ煙メッシュを表示
    - Helmet のときは HDR 環境マップを読み込み
4. **煙（NodeMaterial）**
    
    - `MeshBasicNodeMaterial` の `positionNode` でノイズ＋回転でゆらぎ変形
    - `colorNode` でアルファ/色をノイズ由来で生成
5. **Retro ポストプロセス**
    
    - `retroPass( scene, camera, { affineDistortion } )` を起点に、
        - UV歪み
        - 色にじみ
        - Bayerディザ
        - ポスタライズ
        - ビネット
        - 走査線  
            を合成して `renderPipeline.outputNode` に設定
6. **Inspector GUI**
    
    - 各 uniform をリアルタイム調整
    - `Retro Pipeline` ON/OFF で `defaultPass` と切替

##### r183新規機能の言及（この demo で使っているもの）

このデモで **r183追加要素として中心的**なのは以下です。

- **Retro系TSLポストプロセスノード**
    - `retroPass`（`three/addons/tsl/display/RetroPassNode.js`）
- **CRT系TSLユーティリティ**
    - `scanlines`, `vignette`, `colorBleeding`, `barrelUV`（`three/addons/tsl/display/CRT.js`）
- **形状マスク系TSL**
    - `circle`（`three/addons/tsl/display/Shape.js`）を歪み補正量に利用

加えて、既存系と組み合わせて完成させています。

- `bayerDither`（Bayerディザ）
- `posterize`（色段階化）
- `replaceDefaultUV`（パイプラインUV差し替え）
- `screenSize`（走査線密度を画面解像度基準に）

---

#### 9. [https://openusd.org/release/dl_kitchen_set.html](https://t.co/CqCgWMVi0p)

---

#### 10. [https://threejs.org/editor/](https://t.co/rqspz5lp0g)

**Editorの大規模刷新**。  
添付パッチは「UI再設計 + WebGPU対応 + レンダリング機能強化 + 読み込み導線改善」が中心です。

##### 1) UI/レイアウト刷新

- フォーム系CSSをタグセレクタからクラスベースへ整理（`.Button`, `.Input`, `.TextArea`, `.Select`）。
    - main.css
- メニューバー高さを `32px → 36px` に統一、ツールバー位置を下部中央寄せに変更。
- 新規ダイアログUI（`.Dialog`）追加。
- アニメーションパネル領域と専用リサイザ追加（下部タイムライン）。

関連実装:

- Animation in Animation.js
- AnimationResizer in AnimationResizer.js
- index.html, Resizer.js

##### 2) Renderer/環境設定の強化（WebGL + WebGPU）

- プロジェクト設定にレンダラー種別を追加:
    - `project/renderer/type` (WebGLRenderer / `WebGPURenderer`)
    - Config.js, Sidebar.Project.Renderer.js
- Editorのimport mapに `"three/webgpu"` を追加。
    - index.html
- Viewport側で WebGLRenderer と `WebGPURenderer` を切替可能化。`PMREMGenerator` も分岐。
    - Viewport.js

##### 3) 背景/環境ライトの扱い変更

- Scene背景・環境タイプが `Default` を持つ設計に変更。
- `backgroundType` / `environmentType` をEditor状態・JSONに保存。
    - Editor.js, Sidebar.Scene.js, Viewport.js

##### 4) Renderメニュー新設（画像/動画出力）

- Sidebarの画像・動画出力を廃止し、メニューバーに移動。
    - MenubarRender in [editor/js/Menubar.Render.js)
    - Menubar.js
- 画像出力:
    - Solid / Realistic(Pathtracer)対応
- 動画出力:
    - ffmpeg CDN依存を削除し、`VideoEncoder` + 簡易MP4 muxへ移行
    - index.html, Menubar.Render.js
##### 5) ローダー改善（glTF/USD）

- glTFインポート時に「オブジェクト追加」か「ルートシーン置換」かを選べるダイアログ追加。
    - GLTFImportDialog in [editor/js/GLTFImportDialog.js)
    - Loader.js
- `usda/usdc/usdz` の処理整理、`USDLoader(manager)` 利用。
- 画像拡張子は「他ローダー側でテクスチャとして扱う」分岐を明示。

##### 6) Geometry/Animation編集の実用機能追加

- GeometryパネルにMorph Targets編集UI追加。
    - Sidebar.Geometry.js
- BufferGeometry属性表示改善（Interleaved stride表示など）。
    - Sidebar.Geometry.BufferGeometry.js
- 旧 `Sidebar.Object.Animation` は削除し、下部アニメーションパネルへ移行。
    - editor/js/Sidebar.Object.Animation.js（削除）

##### 7) その他の重要変更

- デフォルトカメラnear/farが大幅拡張 (`0.001` ～ `1e10`)。
    - Editor.js
- Playerロードを `async` 化（起動時の非同期レンダラー初期化に対応）。
    - Player.js, app.js
- Service Workerは実質停止方向（登録解除＆キャッシュ削除）へ。
    - sw.js

---

##### r183新機能の利用に関する言及

このパッチで、r183系の新要素として特に利用しているのは次です。

- **`three/webgpu` エントリの利用**
    
    - import map追加と `WebGPURenderer` 切替実装
    - index.html, Sidebar.Project.Renderer.js, Viewport.js
- **EditorワークフローのWebGPU対応強化**
    
    - Project出力時にレンダラー別import mapを書き分け
    - Sidebar.Project.App.js, index.html
- **トーンマッピング既定値の更新（Neutral）**
    
    - 設定既定値が `NoToneMapping` から `NeutralToneMapping` へ
    - Config.js, Sidebar.Project.Renderer.js

---

### Key Terms & Decisions
#### Timerの機能（Clockとの違い）
- `.stop()` で進行を完全に止められる（Clockは止められない）
- `.setTimescale(n)` でスローや早送りが可能
- `timer.connect( document )`で**Page Visibility API** と連動させてユーザーがタブを切り替えたり、ブラウザを最小化したりした際に、**自動的にタイマーを一時停止**
- Timerは`update()`で内部状態を更新するので1フレーム内でgetDelta()などを複数回呼んでも値が変わらない。(ClockはgetDelta()を呼ぶ毎に違う値を取得)
#### 何故HalfFloatTypeか
HDRレンダリングの条件（1.0以上の値、諧調がなめらか、メモリ消費抑制）をバランスよく満たす

| **フォーマット**           | **HDR対応** | **メモリ負荷** | **特徴**                   |
| -------------------- | --------- | --------- | ------------------------ |
| **UnsignedByteType** | ×         | 低         | 一般的な画像用。1.0以上は切り捨てられる。   |
| **HalfFloatType**    | **○**     | **中**     | **HDRの標準。十分な階調と現実的な負荷。** |
| **FloatType**        | ○         | 高         | 物理シミュレーション等、超高精度が必要な場合用。 |
#### TRAAとは
Temporal Reprojection Anti-Aliasing
過去のフレームを再利用することで低負荷ながら強力なAA手法。DLSS等の超解像技術の基盤にもなっている。微細なノイズが発生しやすい現代のディファードレンダリングにおいて不可欠な標準技術となっています。
#### Haltonとは
Halton数列は、TRAAで各フレームの描画位置を微小にずらす（ジッター）際、サンプリング点を均一に分散させて効率よくジャギーを抑制するために用いられる低不規則性数列です。