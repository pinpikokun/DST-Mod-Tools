# AGENTS.md — DST-Mod-Guide 共通ルール

このファイルは AI エージェント（Claude Code、GitHub Copilot、その他）が自動的に読み込む共通ルールである。

## 前提：このリポジトリの目的と意義

DST-Mod-Tools は、Don't Starve Together (DST) の MOD 開発に必要なツール群と、その使い方ガイドを提供するリポジトリである。

**最終目標**: プログラミング経験がなくても、PC 操作に不慣れでも、このリポジトリのガイドだけを読めば誰でもキャラクター MOD を完成させて Steam Workshop に公開できる状態を維持すること。

ガイドは「MOD 初心者」かつ「PC 初心者」の両方を想定読者としている。専門知識を前提とせず、すべての手順を省略なく記述する。

---

## 用語の統一

以下の用語は全ガイドで表記を統一すること。

| 用語 | 正しい表記 | NG 例 |
|---|---|---|
| ポートレート画像 | ポートレート | 肖像画、portrait |
| MOD | MOD（半角大文字、前後にスペース） | mod、Mod |
| DST | DST（初出時に正式名称を併記） | dst |
| ktools | ktools（小文字） | keytool、Ktools |
| 各辺が 2 の累乗 | 各辺が 2 の累乗（例: 256 x 512 は OK） | 正方形でなければならない |
| キャラクター MOD のマップアイコン | `images/map_icons/` | `images/minimap/` |
| 建造物のマップアイコン | `images/minimap/` | `images/map_icons/` |
| スクリプトの場所 | `data\databundles\scripts.zip` を展開 | `data\scripts\prefabs\` に直接ある |

## ポートレート画像サイズ（全ガイド共通の正値）

| 用途 | サイズ |
|---|---|
| saveslot_portraits | 128 x 128 |
| selectscreen_portraits | 256 x 512 |
| bigportraits | 512 x 512 以上 |
| avatar | 64 x 64 |
| map_icons | 64 x 64 |

---

## リポジトリ構成

```
DST-Mod-Tools/
├── README.md / .html              ← メインガイド（はじめてのキャラクター MOD）
├── AGENTS.md                      ← 本ファイル（共通ルール）
├── CLAUDE.md                      ← Claude Code 固有の設定
├── DST-Spriter/
│   ├── README.md / .html          ← アニメーション総合ガイド
│   ├── README-character.md / .html ← キャラクターアニメーション編
│   ├── README-item-equipment.md / .html ← アイテム・武器防具編
│   ├── README-effect.md / .html   ← エフェクト編
│   └── sample/                    ← サンプルプロジェクト
├── DST-TextureConverter/
│   └── README.md / .html          ← PNG → TEX 変換ガイド
├── DST-ktools/
│   └── README.md / .html          ← krane/ktech デコンパイルガイド
├── DST-Lua-Scripting/
│   ├── README.md / .html          ← Lua スクリプティング総合ガイド
│   ├── README-DST-teemo.md / .html ← Captain Teemo MOD 実践解説
│   ├── README-speech.md / .html   ← キャラクターセリフ（Speech）作成ガイド
│   └── README-modinfo-config.md / .html ← modinfo コンフィグオプション詳細ガイド
├── DST-AI-Agent/
│   └── README.md / .html          ← AI エージェント活用ガイド
├── DST-Workshop-Upload/
│   └── README.md / .html          ← Steam ワークショップ公開ガイド
├── DST-FMOD-Designer/
│   └── README.md / .html          ← キャラクターボイス作成ガイド
└── DST-Chatterbox/
    └── README.md / .html          ← AI 音声合成ガイド
```
