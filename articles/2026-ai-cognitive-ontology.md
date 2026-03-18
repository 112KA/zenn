---
title: 人の認知機能分類
emoji: 🍣
type: ideas
topics: ["ai"]
published: false
---

## はじめに
AI機能は人の認知能力の代替である。
AIの性質を理解するために、AIの力を借りて認知機能を構造化・分類する。
そしてそれを、今後も利用できる参照用コンテキストとして、この記事を残す。
これに至るまでも何度も構造変更をした。利用目的変化や思考の深化によって今後も構造が変わる可能性が大いにある事に留意しておきたい。

## Schema
```yaml
stages:
  - input
  - process
  - output

layers:
  - perception
  - compression
  - organization
  - reasoning
  - creation
  - decision

primitives:
  - detect
  - retrieve
  - select
  - compress
  - compare
  - group
  - label
  - infer
  - score
  - estimate
  - generate
  - modify
  - choose
  - commit

stage_layer_map:
  input:
    - perception
  process:
    - compression
    - organization
    - reasoning
  output:
    - creation
    - decision

primitive_layer_map:
  detect: [perception, reasoning]
  retrieve: [perception]
  select: [compression, creation]
  compress: [compression]
  compare: [organization, reasoning]
  group: [organization, decision]
  label: [organization]
  infer: [reasoning]
  score: [reasoning, decision]
  estimate: [reasoning, decision]
  generate: [creation]
  modify: [creation]
  choose: [decision]
  commit: [decision]
```
```yaml
steps:

  extraction:
    layer: perception
    primitives:
      - detect
      - select

  summarization:
    layer: compression
    primitives:
      - select
      - compress

  classification:
    layer: organization
    primitives:
      - compare
      - group
      - label

  structuring:
    layer: organization
    primitives:
      - group
      - label

  transformation:
    layer: creation
    primitives:
      - select
      - modify

  diagnosis:
    layer: reasoning
    primitives:
      - detect
      - compare
      - infer

  evaluation:
    layer: reasoning
    primitives:
      - compare
      - score

  prioritization:
    layer: decision
    primitives:
      - score
      - compare
      - group

  recommendation:
    layer: creation
    primitives:
      - compare
      - estimate
      - generate

  planning:
    layer: decision
    primitives:
      - estimate
      - group
      - choose

  validation:
    layer: reasoning
    primitives:
      - compare
      - detect

  monitoring:
    layer: perception
    primitives:
      - detect
      - compare
```
```yaml
controls:

  decide:
    role: 最終判断・責任確定
    actor: human
    description: 人が意思決定する

  branch:
    role: 条件分岐
    actor: system
    description: 条件によりフローを分ける

  loop:
    role: 再試行・反復
    actor: system
    description: 条件を満たすまで繰り返す

  escalate:
    role: 人へ引き上げ
    actor: human
    description: AI処理を中断して人に委譲

  terminate:
    role: 終了
    actor: system
    description: フローを終了する
```
## 各階層について
### Meta Layer (3) - Cognitive Flow
Input, Process, Output
### Layer (6) - Cognitive Stage
> [!NOTE] 用途
> 概念理解・説明
#### Input
- Perception(知覚・取得) - 外部情報を取得
- Compression(要約・凝縮) - 情報を圧縮して本質に絞る
#### Process
- Organization(整理・構造化)  - 情報を整理して構造をつくる
- Reasoning(推論) - 情報から意味や関係を導く
#### Output
- Creation(生成) - 新しい情報やアウトプットをつくる
- Decision(判断) - 行動・結論を決める
### Atom (14) - Cognitive Type
> [!NOTE] 用途
> 分析

#### Perception
- search
- extract
- detect
#### Compression
- summarize
- tag
#### Organization
- classify
- structure
#### Reasoning
- compare
- evaluate
- predict
#### Creation
- generate
- recommend
#### Decision
- decide
### Primitive (60) - Operation
> [!NOTE] 用途
> 完全分解