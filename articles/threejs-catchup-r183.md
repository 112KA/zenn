---
title: Three.js Release Catch-Up - r183
emoji: 🍣
type: tech
topics: ["threejs", "javascript"]
published: true
---

## はじめに
いつの間にか知らない機能が追加されていたり書き方が変わっているthreejsをキャッチアップする目的で、AI要約でexamplesの更新を追い、不明点を調査しつつ、バージョンアップの移行詳細を記録する。

https://x.com/threejs/status/2024146551279394933

## 概要
リリース日: 2026/02/19
公式リリースノート: [Release r183 · mrdoob/three.js · GitHub](https://github.com/mrdoob/three.js/releases/tag/r183)
Migration-Guide: [Migration Guide · mrdoob/three.js Wiki · GitHub](https://github.com/mrdoob/three.js/wiki/Migration-Guide#182--183)

:::message
特に以下2点はmigration時にコードの修正箇所が多くなりそう。
:::

- `Clock` から より高機能な `Timer` クラスに置き換わる。
    - [👉 Timer機能（Clockとの違い）](#timer機能（clockとの違い）)
- WebGPU/Nodeベースのポストプロセス管理クラスが`PostProcessing` から `RenderPipeline`に名称変更。
    - Three.jsのレンダリング基盤が単なるポストプロセスから、複数のレンダリングパスや履歴バッファを含むパイプライン全体を管理する設計へ進化しようとしていることを象徴しています。

## examplesベースの更新解説
### 1. Sky, Water Shader改善 - examples/webgl_shaders_ocean
- [https://threejs.org/examples/#webgl_shaders_ocean](https://t.co/SnmmjUo8El)
- [diff (r182 → r183)](https://112ka.github.io/diff/r183_webgl_shaders_ocean.html)
- リアルな波の質感を表現し、空の反射や光の屈折をシミュレーションして、美しい水面を描画するシェーダーのデモ。
- r183での主な変更は、HDR対応、bloom追加、sky shader改良です。
  -> 単なる「静的な水面shader＋空」から「より映画的なシーン」へ表現が進化しました。

::: details details
- **描画品質/HDRパイプラインの強化**
    - `WebGLRenderer` が `outputBufferType: THREE.HalfFloatType` になり、明るさ情報の表現力が向上。
      - [👉 何故HalfFloatTypeか](#何故halffloattypeか)
    - `UnrealBloomPass` を導入
- **露出（Exposure）の調整性向上**
    - `toneMappingExposure` を `0.5 -> 0.1` に変更し、白飛びしにくい初期値へ。
    - GUIに `exposure` スライダー追加。
- **空表現の拡張**
    - `Sky` の uniform に `cloudCoverage / cloudDensity / cloudElevation` を追加。
    - 毎フレーム `sky.material.uniforms['time'].value = time;` で更新。空（雲）の時間変化表現に対応。

:::

---

### 2. New TRAA利用のVolumetric Lighting - examples/webgpu_volume_lighting_traa

- [https://threejs.org/examples/#webgpu_volume_lighting_traa](https://t.co/sKEPxqG2ux)
- r183で追加された**TRAAノードとボリューメトリック照明**を組み合わせた新規デモ。
    [👉 TRAAとは](#traaとは)

::: details details
- ボリューム本体は createTexture3Dで生成した `Data3DTexture` を使い、`VolumeNodeMaterial` の `scatteringNode` で密度をサンプリング。
- 描画は `RenderPipeline` で分離し、以下の順で合成。
    1. `depthPass`（不透明物の深度）
    2. pass + mrt(output, velocity（カラー+モーションベクトル）
    3. traa
- 時間方向の安定化として、halton 由来の32サンプルオフセットを毎フレーム更新し、TRAAの蓄積と整合させています。
  [👉 Haltonとは](#haltonとは)

:::

---

### 3. 改良版SSR Node - examples/webgpu_postprocessing_ssr

- [https://threejs.org/examples/#webgpu_postprocessing_ssr](https://t.co/iyruSamhrx) 
- [diff(r182 → r183)](https://112ka.github.io/diff/r183_webgpu_postprocessing_ssr.html)
- SSR(Screen Space Reflection)を紹介するデモ。
- 主な改善点は、**SSRの見え方の安定化**と**r183系APIへの追従**。

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

### 4. New PostProcess Godray - examples/webgl_postprocessing_godrays

- [https://threejs.org/examples/#webgl_postprocessing_godrays](https://t.co/doP3IjfRzv)
- 点光源からのゴッドレイ（体積光っぽい光条）をポストプロセスで表現する新規デモ。
- 外部ライブラリの以下を利用
    - `postprocessing`
    - `three-good-godrays`
    
::: details details

- 処理の流れ

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

- Godraysパラメータの意味（主要）

    - `density`: レイの密度（細かさ）
    - `maxDensity`: 光の最大濃度
    - `edgeStrength`, `edgeRadius`: エッジ付近の強調
    - `distanceAttenuation`: 距離減衰
    - `color`: 光の色
    - `raymarchSteps`: サンプリング回数（品質/負荷トレードオフ）
    - `blur`: ぼかし有無
    - `gammaCorrection`: ガンマ補正

:::

---
 
### 5. New RetroPassNode - examples/webgpu_postprocessing_retro

- [https://threejs.org/examples/#webgpu_postprocessing_retro](https://t.co/t13VJbZEyl) 
- WebGPU + TSL で、**PS1風のレトロ画面**をポストプロセスで作る新規デモ。
- シーン自体（モデル/煙/ライト）を描いたあと、画面全体に retro 系エフェクトを段階適用する。

::: details details

- 構成の流れ（実装順）

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

        **r183追加要素として中心的**なのは以下です。

        - **Retro系TSLポストプロセスノード**
            - `retroPass`（`three/addons/tsl/display/RetroPassNode.js`）
        - **CRT系TSLユーティリティ**
            - `scanlines`, `vignette`, `colorBleeding`, `barrelUV`（`three/addons/tsl/display/CRT.js`）
        - **形状マスク系TSL**
            - `circle`（`three/addons/tsl/display/Shape.js`）を歪み補正量に利用

        加えて、既存系と組み合わせて完成させています。

        - `bayerDither`（Bayerディザ）[👉 Bayer Ditherとは](#bayer-ditherとは)
        - `posterize`（色段階化）
        - `replaceDefaultUV`（パイプラインUV差し替え）
        - `screenSize`（走査線密度を画面解像度基準に）
    6. **Inspector GUI**
        
        - 各 uniform をリアルタイム調整
        - `Retro Pipeline` ON/OFF で `defaultPass` と切替

:::

---

## Key Terms & Decisions
### Timer機能（Clockとの違い）
- `.stop()` で進行を完全に止められる（Clockは止められない）
- `.setTimescale(n)` でスローや早送りが可能
- `timer.connect( document )`で**Page Visibility API** と連動させてユーザーがタブを切り替えたり、ブラウザを最小化したりした際に、**自動的にタイマーを一時停止**
- Timerは`update()`で内部状態を更新するので1フレーム内でgetDelta()などを複数回呼んでも値が変わらない。(ClockはgetDelta()を呼ぶ毎に違う値を取得)

---

### 何故HalfFloatTypeか
HDRレンダリングの条件（1.0以上の値、諧調がなめらか、メモリ消費抑制）をバランスよく満たす

| **フォーマット**           | **HDR対応** | **メモリ負荷** | **特徴**                   |
| -------------------- | --------- | --------- | ------------------------ |
| **UnsignedByteType** | ×         | 低         | 一般的な画像用。1.0以上は切り捨てられる。   |
| **HalfFloatType**    | **○**     | **中**     | **HDRの標準。十分な階調と現実的な負荷。** |
| **FloatType**        | ○         | 高         | 物理シミュレーション等、超高精度が必要な場合用。 |

---

### TRAAとは
Temporal Reprojection Anti-Aliasing
過去のフレームを再利用することで低負荷ながら強力なAA手法。DLSS等の超解像技術の基盤にもなっている。微細なノイズが発生しやすい現代のディファードレンダリングにおいて不可欠な標準技術となっています。

### Haltonとは
Halton数列は、TRAAで各フレームの描画位置を微小にずらす（ジッター）際、サンプリング点を均一に分散させて効率よくジャギーを抑制するために用いられる低不規則性数列です。

---

### Bayer Ditherとは
![](/images/BayerDither.webp)
*Image generated by AI*
規則的なしきい値行列（Bayer行列）を用いて、各ピクセルの明るさをドットの密度パターンに変換することで、限られた色数で擬似的な階調を高速に再現する手法

## 所感
- Inspector知らない間に便利になってる!?使っていきたい。
- できればversion upごとに追っていきたいけど、気持ちが続くかどうか。。