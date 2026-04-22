---
title: "Three.js Release Catch-Up - r183"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["threejs", "javascript"]
published: false
---

## 概要
リリース日: 2026/04/17
公式リリースノート: [Release r184 · mrdoob/three.js · GitHub](https://github.com/mrdoob/three.js/releases/tag/r184)
Migration-Guide: [Migration Guide · mrdoob/three.js Wiki · GitHub](https://github.com/mrdoob/three.js/wiki/Migration-Guide#183--184)

** 所感 **
:::message
- 今回のupdateについて、自分の利用範囲では更新対応は必要なさそう
:::

- gi系の更新が多い印象
- CanvasTextureどこで使おう。VRのUI的な役割しか思いつかない。。

## examplesベースの更新解説

### 1\. 3D Tiles + Clouds - examples/webgl\_loader\_3dtiles

- [https://threejs.org/examples/\#webgl\_loader\_3dtiles](https://t.co/XwAZLfm161)
- [diff (r183 → r184)](https://112ka.github.io/diff/r184_webgl_loader_3dtiles.html)
- Google Photorealistic 3D Tilesを活用し、広大な都市モデルを読み込んで表示する地理空間レンダリングのデモ。
- 今回の更新では、ボリュームレンダリングによる雲、大気散乱、および高度なポストプロセッシングが統合されました。
    \-\> 単なる「都市モデルの表示」から、時間経過や空気感までをシミュレートする「フォトリアルな地球環境レンダリング」へ進化しました。

::: details details

- **環境・大気シミュレーションの導入**

    - `@takram/three-atmosphere` による大気散乱（Aerial Perspective）を実装。距離に応じた霞みや空の色をシミュレート。
    - `CloudsEffect` を追加。3Dテクスチャ（`Data3DTexture`）を用いたボリュームレンダリングにより、動的な雲とその影を表現。
    - `getSunDirectionECEF` を使用し、指定した時刻（UTC）に基づいた正確な太陽方位を計算。

- **レンダリングパイプラインの高品位化（HDR/Post-processing）**

    - `renderer.outputBufferType` を `THREE.HalfFloatType` に設定し、HDRレンダリングに対応。
    - `AgXToneMapping` を採用し、露出（`toneMappingExposure`）を10に設定。非常に明るい太陽光下でも自然な階調を維持。
        [👉 ToneMappingについて](#tonemappingについて)
    - `postprocessing` ライブラリを導入。`SMAA`（アンチエイリアス）、`Dithering`（階調飛び防止）、`LensFlare`（レンズフレア）をスタック。

- **地理空間制御の精密化**

    - `TileCreasedNormalsPlugin` を自作。3D Tilesのメッシュ法線を `toCreasedNormals` で再計算し、タイルの継ぎ目やライティングの質感を向上。
        [👉 toCreasedNormalsについて](#tocreasednormalsについて)
    - 初期表示位置を東京（北緯35.68, 東経139.80）に設定。`tiles.ellipsoid.getObjectFrame` を用いて、地表に対するカメラの向きと高度を正確に配置。

- **UX/パフォーマンスの調整**

    - `UpdateOnChangePlugin` による変更時のみのレンダリング最適化を継続しつつ、雲の移動などの動的要素をサポート。
    - `adjustHeight`（カメラの地面追従）による読み込み時のガタつきを防ぐため、初回ユーザー操作まで機能を無効化するワークアラウンドを実装。
:::

### 2\. SSGI Ball Pool - examples/webgpu\_postprocessing\_ssgi\_ballpool.html

- [https://threejs.org/examples/#webgpu_postprocessing_ssgi_ballpool](https://t.co/OWMAF8MsDb)
- インタラクティブな物理演算（ボールプール）に、リアルタイムの間接照明（SSGI）を組み合わせた高度なWebGPU新規デモ。
    \-\> WebGPUの計算能力を最大限に活かし、従来は静的だった「光のバウンス」を、物理演算で激しく動く物体に対してもリアルタイムで適用可能にした。

::: details details

- **次世代ポストプロセス・パイプライン (RenderPipeline)**

    - 従来の `EffectComposer` に代わり、TSLベースの `RenderPipeline` を使用。ノードベースでシェーダーを構築するため、パス間のデータの受け渡しが非常に柔軟になっています。
    - `scenePass.setMRT(...)` により、法線や速度ベクトル（Velocity）を一括でレンダリングし、後続のSSGIやTRAAで再利用しています。

- **SSGI (Screen Space Global Illumination)**

    - `ssgi( scenePassColor, scenePassDepth, sceneNormal, camera )` ノードにより、画面内の情報から間接照明を計算。
    - ボールが壁に近づいた際、そのボールの色が壁に「移る」表現（カラーブリーディング）が物理演算と同期してリアルタイムに描写されます。

- **TRAA (Temporal Re-projected Anti-Aliasing)**

    - 過去のフレームと現在のフレームを `velocity`（速度バッファ）を用いて合成。
    - 通常のアンチエイリアス（MSAA）では難しい、SSGI由来のノイズ抑制とエッジの滑らかさを同時に実現しています。

- **最適化技術**

    - `UnsignedByteType` へのバッファ型指定：拡散色や法線情報の精度を必要十分に抑えることで、メモリ帯域幅を最適化しています。
    - `InstancedMesh` の活用：大量の物理演算ボールを1回のドローコールで効率的に描画しています。
:::

-----

### 3\. Light Probe Volume (Sponza) - `examples/webgl_lightprobes_sponza.html`

- [https://threejs.org/examples/#webgl_lightprobes_sponza](https://t.co/FPukaQaeJE)
- [diff (r183 → r184)](https://112ka.github.io/diff/r184_webgl_lightprobes_sponza.html)
- 有名な「Sponza」の建築モデルを使用し、広範囲な空間に対して\*\*ライトプローブのグリッド（LightProbeGrid）\*\*を配置することで、リアルタイムな間接照明（グローバル・イルミネーション）の効果をシミュレートするデモ。
- 新しいアドオン `LightProbeGrid` および `LightProbeGridHelper` の導入。
    \-\> **静的なライトマップに頼らず、実行時にプローブを格子状に自動配置・計算することで、広大な空間内のどこでも一貫した間接照明を享受できるようになった。**
    [👉 LightProbeGridとは](#lightprobegridとは)

::: details details

- **Lighting & Rendering**

    - **`LightProbeGrid` の活用**: 従来の `LightProbe` は単一地点の情報を保持するものでしたが、これを `sizeX/Y/Z`（空間サイズ）と `countX/Y/Z`（密度）に基づいて格子状に一括管理できるようになりました。
    - **動的なベイク (`probes.bake`)**: シーンのライト状況に合わせて `probes.bake(renderer, scene, ...)` を呼び出すことで、その場のライティング情報をキューブマップ経由でキャプチャし、空間内の間接照明を更新します。
    - **ACESFilmicToneMapping**: 近年のThree.jsの標準に合わせ、高輝度な屋外光と日陰のコントラストを美しく表現するために採用されています。

- **Asset Management**

    - **外部モデルローダーの高度化**: `MODEL_INDEX_URL` から動的に Sponza モデルを検索し、環境に最適なバリアント（glTF-Binary等）を選択して読み込む実装が追加されました。これにより、サンプルのリソース管理がより柔軟になっています。

- **Helper & Debugging**

    - **`LightProbeGridHelper`**: 空間内に配置された多数のプローブ（球体）を視覚化し、それぞれの地点でどのような光がキャプチャされているかをデバッグ可能にしています。
:::


### 4\. Light Probe Volume - examples/webgl\_lightprobes

- [https://threejs.org/examples/\#webgl\_lightprobes](https://t.co/y5O0BRF6mm)
- [diff (r183 → r184)](https://112ka.github.io/diff/r184_webgl_lightprobe.html)
- コーネルボックス内でL1球面調和関数（SH）プロローブのグリッドを使用して、位置依存の間接拡散照明（Global Illumination）をシミュレートするデモ。
- 今回の更新で、新しく `LightProbeGrid` と `LightProbeGridHelper` が追加され、シーン全体のGIをベイク・可視化することが可能になりました。
    \-\> 単なる「静的な環境光」から「空間の位置に応じたリアルな色被り（カラーブリーディング）を含むGI表現」が可能になりました。

::: details details

- **GI（グローバルイルミネーション）システムの導入**

    - `LightProbeGrid` を使用して、指定した解像度（グリッド数）で空間内のライティング情報をベイク。
    - L1球面調和関数（SH）を用いることで、少ないデータ量で位置に応じた拡散反射光の変化をリアルタイムに計算。
    - 
- **ベイク・プロセスの追加**

    - `probes.bake(renderer, scene, ...)` により、実行時にシーンのライティング情報をサンプリング。
    - 動的に解像度（Resolution）を変更して再ベイクする機能を実装し、精度とパフォーマンスのトレードオフを確認可能に。

- **デバッグ・可視化機能**

    - `LightProbeGridHelper` を導入。空間に配置された各プロブがどのような光情報を保持しているかを球体として可視化。
    - 左の壁が赤、右の壁が緑のコーネルボックス内で、物体が壁に近づくとその色が反射して映る「カラーブリーディング」を明確に再現。

- **レンダリング設定の最適化**

    - `ACESFilmicToneMapping` を採用し、HDRなライティング環境をシリアスかつフォトリアルな階調で描画。
:::

### 5\. Watch (Rolex) - examples/webgl\_watch

- [https://threejs.org/examples/\#webgl\_watch](https://t.co/IYWSdsQ3FJ)
- [diff (r183 → r184)](https://112ka.github.io/diff/r184_webgl_watch.html)
- 高級時計（Rolex）の3Dモデルを表示し、環境マップによる反射、AOマップ（Ambient Occlusion）による陰影、そしてリアルタイムの時刻同期を実演するデモ。
- 今回の更新では、ライティング設定の微調整とトーンマッピングの変更が行われ、質感がより実物に近いニュートラルな表現に調整されました。

::: details details

- **トーンマッピングの刷新**

    - `renderer.toneMapping` を `ACESFilmicToneMapping` から `NeutralToneMapping` に変更。
    - Khronosグループが推奨する最新の「Neutral」設定を採用することで、色の彩度やコントラストが過度に強調されるのを抑え、アーティストが意図した通りの色味を再現。
    [👉 ToneMappingについて](#tonemappingについて)

- **ライティングと影の品質向上**

    - `DirectionalLight` の位置を調整（`-0.1` → `0.2`）し、時計の文字盤やケースへの光の当たり方を最適化。
    - シャドウマップの解像度を `1024` から `2048` へ倍増。エッジのギザギザを減らし、より精細な影を描画。
    - `VSMShadowMap` 設定の削除など、レンダラーの標準仕様に合わせたクリーンアップ。

- **マテリアルとメッシュ処理の改善**

    - 風防（ガラス）の不透明度（`opacity`）を `0.4` から `0.8` へ引き上げ、ガラスの存在感を強調。
    - 同じ名前のマテリアルが重複して生成されないよう、読み込み時のキャッシュ処理を追加。メモリ効率と一貫性が向上。
    - 床（floor）メッシュに対して不要な影のキャスト/レシーブ設定を除外する判定を追加。

- **情報の更新**

    - モデルのデータ量（またはテクスチャを含む総量）の表記を `610 ko` から `1200 ko` に更新し、現在のリソース状況を正確に反映。
:::

### 6\. HTML Texture - examples/webgl\_materials\_texture\_html

  - [https://threejs.org/examples/\#webgl\_materials\_texture\_html](https://t.co/uGLpn7HK4g)
  - HTML要素（DOM）をテクスチャとして3Dオブジェクトに貼り付け、WebGL空間内で描画・更新する新しい機能のデモ。

::: details details

- **`HTMLTexture` の新規導入**

    - `new THREE.HTMLTexture(element)` を用いることで、DOM要素を自動的にCanvasへレンダリングし、WebGLのテクスチャとして同期。
    - CSSアニメーション（`@keyframes`）や、画像・SVGの混在したリッチなUIをマテリアルとして利用可能。

- **HTML-in-Canvas API とポリフィルの活用**

    - 将来的なブラウザ標準機能である `HTML-in-Canvas API` (WICG) を先取り。
    - 未対応ブラウザ向けに `three-html-render/polyfill` を導入し、広範な互換性を確保。

- **インタラクションの統合**

    - `InteractionManager` を使用して、3D空間上のメッシュに対するマウスイベントを背後のHTML要素へ伝達。
    - 3Dオブジェクト上に描画されたボタンをクリックして中身を書き換えるといった、双方向のUI操作を実現。
    - 
- **レンダリング品質の維持**

    - `RoundedBoxGeometry` を使用し、角丸のボックスにテクスチャを貼り付け。
    - `NeutralToneMapping` と `RoomEnvironment` を組み合わせることで、HTML由来の色彩を保ちつつ、リアルな環境反射を両立。
:::

## Key Terms & Decisions

### toCreasedNormalsについて
![](/images/toCreasedNormals.webp)
*Image generated by AI*

この処理を行うことで、ローポリゴンのモデルでも、機械的なカッチリとした見た目を効率的に表現できるようになります。

#### 左（Original Mesh）:
急な角度（Sharp Angle）を持つエッジでも、1つの頂点（Shared Vertex）に対して 1つの平均化された法線（Normal） しか持っていません。この状態だと、ライティングが滑らかになりすぎてしまい、角が「溶けた」ような見た目になります。

#### 中央（toCreasedNormals ACTION）:
処理が実行されます。システムが特定の角度（例：45度）を超えるエッジを検出し、共有されていた法線をそれぞれの面に対して分割（Splitting） します。

#### 右（Resulting Mesh）:
結果です。見た目は1つの頂点ですが、論理的には 分割された頂点（Split Vertices A1, A2） となり、それぞれが独自の 独立した法線（Normal A, B） を持っています。これにより、光の当たり方がエッジでパキッと分かれ、鋭い角（ハードエッジ）が表現されます。



### ToneMappingについて

3DCGのレンダリングにおける**トーンマッピング（Tone Mapping）**は、計算上の広大な光の範囲（HDR）を、ディスプレイの表示可能範囲（SDR）へ最適に圧縮する工程です。

以下、主要な手法についてです。

#### 1. AgX Tone Mapping（現代のスタンダード）
Blender 4.0以降でデフォルト採用され、Three.jsでも推奨され始めている最新の手法です。

- 特徴: 非常に明るい部分（ハイライト）が、白飛びする前に**「自然に彩度が落ちていく」**フィルムのような挙動をします。デジタル特有の「不自然な色の変化（青が水色に化ける等）」が起きません。
- 用途: 最高品質のフォトリアルなレンダリング。
- 印象: 高級感がある、落ち着いている、写実的。

#### 2. ACES Filmic Tone Mapping（映画的・ドラマチック）
映画業界の標準（Academy Color Encoding System）に基づいた手法です。

- 特徴: S字カーブが強く、コントラストと彩度がはっきり出ます。暗い部分はより暗く、明るい部分は鮮やかに強調されます。
- 用途: 映画、ゲーム、没入感重視のコンテンツ。
- 印象: パキッとしている、カッコいい、ドラマチック。

#### 3. Neutral Tone Mapping（正確・製品重視）
Khronos GroupがglTFの表示基準として策定した手法です。

- 特徴: 意図的なコントラスト調整を最小限に抑え、入力されたテクスチャの色味を最大限に尊重します。明るい部分も色相を保ったまま滑らかに処理します。
- 用途: ECサイト（商品の色確認）、建築ビジュアライゼーション、正確な色再現が必要な場合。
- 印象: 素直、クリア、誠実。

### LightProbeGridとは？

従来のThree.jsで間接照明を扱う場合、`LightProbe` を手動で配置するか、高価な `PMREM` 等を用いた環境マップに依存していました。
今回導入された **`LightProbeGrid`** は、指定したバウンディングボックス内を3Dのグリッドで分割し、各格子点で光の情報を球面調和関数（Spherical Harmonics）として保持します。

**メリット:**

1.  **動的な物体への対応**: 空間内を移動するキャラクターや物体が、その場所の光の色（例えば、赤い壁の近くなら赤っぽくなるなど）を正しく反映できます。
2.  **パフォーマンス**: フルスペックのリアルタイムGI（レイトレーシング等）に比べ、計算負荷が非常に低く、モバイル環境でも動作可能です。
3.  **利便性**: 手動でプローブを並べる手間がなくなり、`bake()` メソッド一つで環境をキャプチャできます。

この進化により、Three.jsにおける「空間のライティング」の質が一段階引き上げられたと言えます。