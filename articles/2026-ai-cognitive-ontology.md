---
title: 人の認知機能分類
emoji: 🍣
type: idea
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
```
```yaml
patterns:

  extraction:
    primitives:
      - detect
      - select

  summarization:
    primitives:
      - select
      - compress

  classification:
    primitives:
      - compare
      - group
      - label

  structuring:
    primitives:
      - group
      - label

  transformation:
    primitives:
      - select
      - modify

  diagnosis:
    primitives:
      - detect
      - compare
      - infer

  evaluation:
    primitives:
      - compare
      - score

  prioritization:
    primitives:
      - score
      - compare
      - group

  recommendation:
    primitives:
      - compare
      - estimate
      - generate

  planning:
    primitives:
      - estimate
      - group
      - choose

  validation:
    primitives:
      - compare
      - detect

  monitoring:
    primitives:
      - detect
      - compare
```

## 各階層についての説明
### Stage (3) - 処理の流れ
Input, Process, Output
### Layer (6) - 認知の種類
> [!NOTE] 用途
> 概念理解・説明

- patternの「性質」
- どの種類の思考か
#### Input
- Perception: 知覚（取得・検知）
#### Process
- Compression: 圧縮（削減・要約）
- Organization: 整理（構造化）
- Reasoning: 推論（意味づけ）
#### Output
- Creation: 生成（新規作成・変更）
- Decision: 判断（選択・確定）
### Patterns (12) - 認知パターン
> [!NOTE] 用途
> 分析

- primitiveの組み合わせ
- 意味を持つ
- 人が機能を理解しやすい

### Primitive (14) - Operation
> [!NOTE] 用途
> 分解

- 最小単位
- 文脈を持たない