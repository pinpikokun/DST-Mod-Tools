# DST-Spriter: Don't Starve Together MOD アニメーション作成ガイド

Don't Starve Together（DST）の MOD を作ると、**自分だけのキャラクター・武器・エフェクト** をゲームに追加できます。

本ガイドでは、DST のアニメーション作成ツール **Spriter** の使い方から、MOD として動かすまでの手順を、初心者にもわかりやすく解説します。

> **HTML 版**: [README.html](https://pinpikokun.github.io/DST-Mod-Guide/DST-Spriter/README.html)

---

## このガイドで学べること

このガイドは **全体共通の基礎知識**（このファイル）と、**カテゴリ別の詳細ガイド** で構成されています。

| ガイド | 内容 |
|---|---|
| **README.md**（このファイル） | DST アニメーションの基礎、環境準備、フォルダ構成、ビルド手順、Lua 設定、画像変換 |
| [README-character.md](README-character.md) | キャラクターアニメーション（メインキャラ・ゴースト・スキン） |
| [README-item-equipment.md](README-item-equipment.md) | アイテム・武器防具アニメーション（地面の見た目・装備時の見た目） |
| [README-effect.md](README-effect.md) | エフェクトアニメーション（視覚効果・投射物・範囲表示） |

> **Tips**: まずはこのファイルを最後まで読んで全体像をつかみ、それから興味のあるカテゴリの詳細ガイドに進んでください。

---

## DST のアニメーション（Spriter）とは

DST に登場するキャラクター・アイテム・エフェクトは、すべて **2D のスプライトアニメーション** で表現されています。これらのアニメーションは **Spriter** というツールで作られています。

### 全体の流れ

```
PNG 画像を用意する
    ↓
Spriter で画像を配置してアニメーションを作る（.scml ファイル）
    ↓
DST Mod Tools でビルドする（.zip ファイルに変換）
    ↓
MOD の anim/ フォルダに配置する
    ↓
Lua コードで読み込んでゲーム内に表示する
```

### Bank・Build・Animation — 3 つの重要な概念

DST のアニメーションシステムには、3 つのキーワードがあります。これを理解することが MOD 作成の第一歩です。

| 用語 | 一言で言うと | 決まり方 | 例 |
|---|---|---|---|
| **Bank** | アニメーションの「動き」のグループ | Spriter の Entity 名 | `my_sword` |
| **Build** | アニメーションの「見た目」のセット | Spriter のファイル名（拡張子なし） | `my_sword` |
| **Animation** | 個々のアニメーション名 | Spriter の Animation 名 | `idle` |

> **Tips**: Bank は「動きの型」、Build は「見た目の着せ替え」と考えるとわかりやすいです。同じ Bank（動きの型）に対して Build（見た目）を差し替えれば、動きはそのままに見た目だけ変えられます。DST のキャラクタースキンもこの仕組みを利用しています。

---

## 環境準備

MOD のアニメーションを作るには、以下の 3 つのツールが必要です。

### 1. DST Mod Tools（必須）

Spriter 本体やビルドツールがすべて含まれています。

**インストール手順:**
1. Steam を起動する
2. ライブラリを開き、上部のフィルタで「ツール」を選択する
3. 「Don't Starve Mod Tools」を検索してインストールする

> **Note**: インストール先はデフォルトで `C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\` です。

### 2. Spriter（DST Mod Tools に同梱）

DST Mod Tools をインストールすると、Spriter も一緒にインストールされます。追加の操作は不要です。

### 3. Git Bash（ビルドコマンド実行用）

ビルド手順で使う bash コマンドを実行するために必要です。

**インストール手順:**
1. [Git for Windows](https://gitforwindows.org/) をダウンロードしてインストールする
2. インストール後、フォルダを右クリック →「Git Bash Here」で起動できる

> **Note**: Windows のコマンドプロンプト（cmd）では本ガイドのコマンドは動作しません。必ず Git Bash を使ってください。

---

## MOD のフォルダ構成

DST MOD は決まったフォルダ構成を持っています。アニメーションに関連する部分を中心に説明します。

```
<MOD名>/
├── modmain.lua                     ← MOD のメインスクリプト（Asset 宣言など）
├── modinfo.lua                     ← MOD の情報（名前・説明・設定）
├── modicon.tex / .xml              ← Steam ワークショップのアイコン
│
├── anim/                           ← ★ Spriter アニメーション ZIP を置く場所
│   ├── キャラクター名.zip
│   ├── アイテム名.zip
│   └── エフェクト名.zip
│
├── images/                         ← 画像ファイル（TEX + XML 形式）
│   ├── inventoryimages/            ←   インベントリアイコン（アイテム欄の画像）
│   ├── avatars/                    ←   チャット欄のアバター
│   ├── hud/                        ←   HUD（クラフトタブ等）
│   ├── map_icons/                  ←   マップ上のアイコン
│   ├── saveslot_portraits/         ←   セーブスロットのポートレート
│   └── selectscreen_portraits/     ←   キャラクター選択画面
│
├── bigportraits/                   ← キャラクター選択時の大きいポートレート
│
├── scripts/
│   ├── prefabs/                    ← プレハブ（キャラ・アイテム等の定義）
│   ├── components/                 ← コンポーネント（機能の定義）
│   └── widgets/                    ← UI ウィジェット
│
└── sound/                          ← サウンドファイル（FMOD 形式）
```

> **Tips**: アニメーション ZIP は必ず `anim/` フォルダに配置してください。それ以外の場所に置いてもゲームは読み込みません。

---

## アニメーションの種類一覧

DST MOD で使われるアニメーションは、大きく分けて以下の種類があります。

| 種類 | 内容 | ZIP の例 | 詳細ガイド |
|---|---|---|---|
| **キャラクター** | メインキャラの全ポーズ（idle, walk, attack 等） | `my_character.zip` | [README-character.md](README-character.md) |
| **ゴースト** | 死亡時の幽霊の見た目 | `ghost_my_character_build.zip` | [README-character.md](README-character.md) |
| **アイテム** | 地面に置いたときの見た目 | `my_sword.zip` | [README-item-equipment.md](README-item-equipment.md) |
| **装備スワップ** | キャラが手に持ったときの見た目 | `swap_my_sword.zip` | [README-item-equipment.md](README-item-equipment.md) |
| **エフェクト** | 視覚効果（バフ・デバフ・爆発等） | `my_effect.zip` | [README-effect.md](README-effect.md) |

> **Note**: すべての種類を作る必要はありません。自分の MOD に必要なものだけ作ればOKです。たとえばエフェクト MOD ならキャラクターアニメーションは不要です。

---

## ビルド手順（Spriter → DST 用 ZIP）

Spriter で作ったアニメーションを、ゲームが読み込める `.zip` に変換する手順です。すべてのアニメーション種類で共通の手順です。

> **autocompiler との違い**: [メインガイド](../README.md) では `autocompiler.exe` を使う方法を紹介しています。autocompiler は内部的にここで説明する `scml.exe` + `buildanimation.py` と同じ処理を行っています。**初心者は autocompiler を使うのが簡単です**。ここでは Git Bash を使った手動ビルド手順を説明しており、エラー時の原因特定や細かい制御をしたい場合に役立ちます。

### 0. Git Bash を起動する

フォルダを右クリック →「Git Bash Here」を選択するか、スタートメニューから Git Bash を起動してください。

### 1. シェル変数を設定する

> **なぜ「シェル変数」？**: シェルとは、Git Bash のような **コマンドを入力する画面（ターミナル）そのもの** のことです。シェル変数とは、その画面の中だけで使える **一時的なメモ書き** のようなものです。長いパスに短い名前を付けておくことで、後のコマンドで何度も長いパスを打ち直す手間を省けます。

後続のコマンドで使う変数を設定します。**必ず最初に実行してください。**

```bash
MOD_TOOLS="C:/Program Files (x86)/Steam/steamapps/common/Don't Starve Mod Tools/mod_tools"
PROJECT_NAME="<プロジェクト名>"  # .scmlファイルの拡張子なしの名前
```

> **Note**: これは環境変数ではなく、このターミナルセッション内でのみ有効な一時的なシェル変数です。ターミナルを閉じると消えるので、再度実行する場合はもう一度設定してください。

### 2. Spriter プロジェクトを `exported/` にコピー

```bash
cp -r "<Spriterプロジェクトのフォルダ>" "$MOD_TOOLS/exported/$PROJECT_NAME"
```

> **Note**: コピー先は `C:\Program Files (x86)\Steam\...\mod_tools\exported\` です。「Spriter プロジェクトのフォルダ」とは、`.scml` ファイルと画像フォルダが入っているフォルダのことです。

### 3. `scml.exe` で中間 ZIP を生成

```bash
cd "$MOD_TOOLS"
./scml.exe "exported/$PROJECT_NAME/$PROJECT_NAME.scml" "exported/$PROJECT_NAME"
```

### 4. `buildanimation.py` で最終 ZIP を生成

```bash
./buildtools/windows/Python27/python.exe tools/scripts/buildanimation.py \
  "exported/$PROJECT_NAME/$PROJECT_NAME.zip" \
  --outputdir="exported/$PROJECT_NAME" --force --ignoreexceptions
```

> **Note**: Python を別途インストールする必要はありません。DST Mod Tools に Python 2.7 が同梱されており（`buildtools/windows/Python27/python.exe`）、上記コマンドはそれを直接使用しています。

### 5. 生成された ZIP を MOD にコピー

```bash
cp "$MOD_TOOLS/exported/$PROJECT_NAME/anim/$PROJECT_NAME.zip" "<MODフォルダ>/anim/"
```

---

## Lua 側の基本設定

ビルドした ZIP をゲーム内で使うには、Lua コードでの設定が必要です。

### Asset 宣言

アニメーション ZIP はゲーム起動時に読み込む必要があります。宣言する場所は 2 箇所あります。

**prefab ファイル内で宣言する場合（推奨）:**

> **Note**: Prefab（プレハブ）とは、DST におけるゲーム内オブジェクトの定義ファイルです。キャラクター、アイテム、エフェクトなど、ゲーム内に存在するものはすべて Prefab として定義されます。`scripts/prefabs/` フォルダに Lua ファイルとして配置します。

```lua
local assets = {
    Asset("ANIM", "anim/my_sword.zip"),
}
```

**modmain.lua で宣言する場合（キャラクター等）:**

```lua
Assets = {
    Asset("ANIM", "anim/my_character.zip"),
    Asset("ANIM", "anim/ghost_my_character_build.zip"),
}
```

### AnimState の基本操作

ゲーム内でアニメーションを表示・制御するには `AnimState` を使います。これは Prefab の Lua ファイル内（`scripts/prefabs/` 配下）に記述します。

```lua
-- 基本の3行セット
inst.AnimState:SetBank("my_sword")           -- Bank を指定
inst.AnimState:SetBuild("my_sword")          -- Build を指定
inst.AnimState:PlayAnimation("idle")         -- アニメーションを再生

-- ループ再生（エフェクト等に使う）
inst.AnimState:PlayAnimation("open")              -- まず1回再生
inst.AnimState:PushAnimation("idle_loop", true)   -- 続けてループ再生

-- 装備スワップ（キャラが武器を手に持つ）
owner.AnimState:OverrideSymbol("swap_object", "swap_my_sword", "swap_my_sword")

-- 地面に表示するエフェクト（向きの設定）
inst.AnimState:SetOrientation(ANIM_ORIENTATION.OnGround)
```

### Bank / Build / Animation の典型的なパターン

| 種類 | Bank | Build | 主なアニメーション | 備考 |
|---|---|---|---|---|
| キャラクター | 独自 or `wilson` | 独自 | idle, walk, attack 等 | `wilson` Bank を流用すれば動きは自動 |
| ゴースト | `ghost`（流用） | 独自 | idle 等 | DST 本体の Bank を流用 |
| アイテム | 独自 | 独自 | idle | 静止画 1 枚で OK |
| 装備スワップ | `swap_object`（流用） | `swap_<名前>` | — | DST 本体の Bank を流用 |
| エフェクト | `forcefield` 等（流用） | 独自 | open, idle_loop | 既存 Bank を流用可能 |
| 投射物 | `blow_dart` 等（流用） | 独自 | idle_pipe, dart_pipe | 既存 Bank を流用可能 |

> **Tips**: Bank に `ghost`、`swap_object`、`forcefield`、`poopcloud` など DST 本体の Bank 名を使う場合があります。これは DST 本体の「動きの型」をそのまま流用し、見た目（Build）だけ自分のものに差し替えるテクニックです。

---

## 命名規則

DST のアニメーションシステムでは、ファイル名やフォルダ名がそのまま Bank 名や Build 名になります。命名には注意が必要です。

**使える文字:** 英小文字、数字、ハイフン（`-`）、アンダースコア（`_`）

> **重要**: **スペースや日本語は使わないでください。** ビルド時にエラーになるか、ゲーム内で正しく読み込まれません。

**その他のルール:**
- scml ファイル名とプロジェクトフォルダ名は **一致させる** のが安全です
- PNG 画像のファイル名には特に命名規則はありません（自由に付けられます）

---

## Appendix A: インベントリアイコン（PNG → TEX + XML）

アイテムをインベントリ（持ち物欄）に表示するには、アニメーション ZIP とは別に **TEX + XML 形式のアイコン画像** が必要です。PNG 画像を DST 専用の TEX 形式に変換し、対応する XML アトラスファイルとセットで MOD に配置します。

| 項目 | 概要 |
|---|---|
| PNG サイズ | **64 x 64 px**（正方形・2 のべき乗。それ以外はクラッシュ） |
| 変換ツール | `TextureConverter.exe`（DST Mod Tools に同梱） |
| 出力ファイル | `.tex`（テクスチャ）+ `.xml`（アトラス定義） |
| 配置先 | `<MOD名>/images/inventoryimages/` |
| Asset 宣言 | `Asset("IMAGE", "...tex")` + `Asset("ATLAS", "...xml")` を modmain.lua に追加 |

詳しい変換コマンド・オプション・XML の書き方については、専用ガイドを参照してください。

→ [DST-TextureConverter](../DST-TextureConverter/) — PNG → TEX + XML 変換の詳細ガイド

---

## Appendix B: その他の画像アセット

キャラクター MOD を作る場合、インベントリアイコン以外にも画像が必要です。すべて PNG → TEX + XML 変換の手順は同じです。

| 画像の種類 | 配置先 | 推奨サイズ | 用途 |
|---|---|---|---|
| インベントリアイコン | `images/inventoryimages/` | 64 x 64 | アイテム欄の画像 |
| アバター | `images/avatars/` | 64 x 64 | チャット欄のキャラアイコン |
| マップアイコン | `images/map_icons/` | 64 x 64 | マップ上の表示 |
| HUD アイコン | `images/hud/` | 128 x 128 | クラフトタブ等 |
| セーブスロットポートレート | `images/saveslot_portraits/` | 128 x 128 | セーブ画面 |
| キャラ選択画面ポートレート | `images/selectscreen_portraits/` | 256 x 512 | キャラ選択 |
| 大きいポートレート | `bigportraits/` | 512 x 512 以上 | キャラ選択（詳細） |
| MOD アイコン | ルート直下 | 128 x 128 | Steam ワークショップ |

> **Note**: すべて各辺が 2 のべき乗サイズにしてください（正方形でなくても各辺が 2 のべき乗なら OK。例: 256 x 512）。サイズが合わないとクラッシュします。

---

## サンプルプロジェクト

本リポジトリには以下のサンプルが含まれています。

| サンプル | 内容 |
|---|---|
| [README-blind-dart-renge-ring.md](sample/README-blind-dart-renge-ring.md) | 射程範囲リング作成の詳細ガイド |
| `sample/DST-teemo-blind-dart-renge-ring/` | 上記ガイドの Spriter プロジェクトファイル |

---

## 参考リンク

| リンク | 内容 |
|---|---|
| [DST Modding Guide（Klei Forums）](https://forums.kleientertainment.com/forums/topic/59174-tutorial-the-artists-guide-to-dont-starve-together-modding/) | 公式フォーラムのアーティスト向けガイド |
| [Spriter 公式サイト](https://brashmonkey.com/) | BrashMonkey 製 2D アニメーションツール |
| [DST Mod Tools（Steam）](https://store.steampowered.com/app/250820/Dont_Starve_Mod_Tools/) | ビルドツール一式（無料） |
| [Git for Windows](https://gitforwindows.org/) | Git Bash 同梱 |

---

## ライセンス

このドキュメントは自由に利用・改変・再配布できます。DST MOD 制作にお役立てください。

---

<sub>Author: [pinpikokun](https://steamcommunity.com/profiles/76561198076111536/)</sub><br>
<sub>[![GitHub](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/pinpikokun/DST-teemo) [![Steam Workshop](https://img.shields.io/badge/Steam%20Workshop-Subscribe-blue?logo=steam)](https://steamcommunity.com/sharedfiles/filedetails/?id=390684095)</sub>
