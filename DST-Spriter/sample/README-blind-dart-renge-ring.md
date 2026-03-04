# DST-Spriter ブラインドダートの射程範囲リングの作成について

Don't Starve Together (DST) の MOD 開発において、**Spriter** を使ってゲーム内アニメーションを作成する際のサンプルプロジェクトと手順書です。

## 前提・背景

本リポジトリでは、DST Teemo MOD で実際に作成したサンプル（`DST-teemo-blind-dart-renge-ring/`）を使って、Spriter によるアニメーション作成手順を説明します。

- **サンプル内容**: ブラインドダート（Blind Dart）を装備した際に表示される **射程範囲リング（円）** のアニメーション
- **使用ツール**: [Spriter](https://brashmonkey.com/)（BrashMonkey 製 2D アニメーションツール）
- **目的**: Spriter でリング型アニメーションを作る方法を、他の MOD 開発者も参照できる形でまとめたもの

> **Tips**: Spriter は Steam ライブラリ → フィルタで「ツール」を選択 → 「Don't Starve Mod Tools」をインストールすると同梱されており、無料で利用できます。

## フォルダ構成

```
DST-Spriter/
├── README.md
└── DST-teemo-blind-dart-renge-ring/   # サンプルのSpriterプロジェクトフォルダ
    ├── blind-dart-renge-ring.scml     # Spriterプロジェクトファイル
    └── targetring01/                  # 画像素材
        ├── targetring01-11.png        # 点線リング用パーツ
        ├── targetring01-12.png        # 枠付き実線リング用パーツ
        ├── targetring01-13.png        # 枠なし実線リング用パーツ
        ├── targetring01-14.png        # 実線リング用パーツ
        └── targetring01-15.png        # 枠なし円リング用パーツ
```

## 含まれるアニメーション一覧

| アニメーション名 | 使用画像 | 説明 |
|---|---|---|
| `renge-ring-dotted-line` | targetring01-11 | 点線で描画されたリング |
| `renge-ring-line` | targetring01-14 | 実線で描画されたリング |
| `renge-ring-line-frameless` | targetring01-13 | 枠なしの実線リング |
| `renge-ring-line-frame` | targetring01-12 | 枠付きの実線リング |
| `renge-ring-circle-frameless` | targetring01-15 | 枠なしの円形リング |

各アニメーションは、1枚の画像パーツを **30° 間隔で 12 枚配置** して円を構成しています。

> **採用アニメーション**: 複数のバリエーションを試作した結果、元となった League of Legends（LoL）のアタックムーブで表示される射程範囲のイメージに最も近かった **`renge-ring-circle-frameless`**（枠なしの円形リング）を Teemo MOD で採用しました。

---

## Spriter での射程リング作成手順（手動）

### 1. アニメーションの長さを設定

画面下部のタイムラインで以下の値を入力：

- **開始**: `0`
- **長さ**: `134`

> **補足**: 長さ `134` は、DST 本体に含まれる同様の円型アニメーションが採用している値に合わせたものです。なぜ `134` なのかの理由は不明ですが、既存のアニメーションに倣うのが安全です。

### 2. 画像のピボット（回転軸）を設定

右ペインで画像をクリックし、以下の値に変更：

- **pivot_x**: `18.682900`
- **pivot_y**: `0.531640`

> ピボットは画像の回転中心点です。この値により、画像の上端付近を軸にして回転します。

### 3. キャンバス上の配置位置を設定

左ペインで画像の位置を以下に設定：

- **x**: `0`
- **y**: `0`

### 4. 画像を 12 枚配置し、各パーツに角度を設定

同じ画像を 12 枚配置し、それぞれ以下の角度を入力して円を形成します：

| # | 角度 |
|---|------|
| 1 | 0° |
| 2 | 30° |
| 3 | 60° |
| 4 | 90° |
| 5 | 120° |
| 6 | 150° |
| 7 | 180° |
| 8 | 210° |
| 9 | 240° |
| 10 | 270° |
| 11 | 300° |
| 12 | 330° |

---

## Spriter → DST 用アニメーション ZIP のビルド手順

### 前提条件

- **DST Mod Tools** がインストール済みであること（インストール方法は [初心者向け注意点・Tips > 環境準備](#環境準備) を参照）
- パス: `C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\`

### 手順

#### 0. ターミナルを起動する

以降のコマンドは **Git Bash** で実行します（bash 構文を使用しているため、Windows のコマンドプロンプトでは動作しません）。

> **Note（Windows）**: Git Bash は [Git for Windows](https://gitforwindows.org/) をインストールすると付属します。右クリックメニューの「Git Bash Here」からも起動できます。

#### 1. シェル変数を設定する

後続のコマンドで繰り返し使うパスや名前を変数に設定します。**手順 2 以降で使うため、必ず最初に実行してください。**

```bash
MOD_TOOLS="C:/Program Files (x86)/Steam/steamapps/common/Don't Starve Mod Tools/mod_tools"
PROJECT_NAME="<プロジェクト名>"  # .scmlファイルの拡張子なしの名前
```

> **Note**: これは環境変数ではなく、このターミナルセッション内でのみ有効な一時的なシェル変数です。ターミナルを閉じると消えるので、再度実行する場合はもう一度設定してください。

#### 2. Spriter プロジェクトを `exported/` にコピー

```bash
cp -r "<Spriterプロジェクトのフォルダ>" "$MOD_TOOLS/exported/$PROJECT_NAME"
```

> **Note（Windows）**: コピー先は `C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\exported\` です。「Spriterプロジェクトのフォルダ」とは、`.scml` ファイルと画像フォルダが入っているフォルダのことです（本サンプルでは `DST-teemo-blind-dart-renge-ring/`）。

#### 3. `scml.exe` で中間 ZIP を生成

`animation.xml`、`build.xml`、PNG が含まれる中間 ZIP を生成します。

```bash
cd "$MOD_TOOLS"
./scml.exe "exported/$PROJECT_NAME/$PROJECT_NAME.scml" "exported/$PROJECT_NAME"
```

#### 4. `buildanimation.py` で最終 ZIP を生成

`anim.bin`、`build.bin`、`atlas-0.tex` が含まれる DST 用の最終 ZIP を生成します。

```bash
./buildtools/windows/Python27/python.exe tools/scripts/buildanimation.py \
  "exported/$PROJECT_NAME/$PROJECT_NAME.zip" \
  --outputdir="exported/$PROJECT_NAME" --force --ignoreexceptions
```

#### 5. 生成された ZIP を MOD にコピー

```bash
cp "$MOD_TOOLS/exported/$PROJECT_NAME/anim/$PROJECT_NAME.zip" "<MODフォルダ>/anim/"
```

---

## Lua 側の設定

```lua
-- Asset登録
Asset("ANIM", "anim/<ファイル名>.zip")

-- AnimState設定
fx.AnimState:SetBank("<scmlのエンティティ名>")
fx.AnimState:SetBuild("<scmlのファイル名（拡張子なし）>")
fx.AnimState:PlayAnimation("<アニメーション名>", true)
```

| 設定項目 | 対応する値 | 例（本サンプルの場合） |
|---|---|---|
| **Bank** | scml 内の `<entity name="...">` の値 | `blind-dart-renge` |
| **Build** | scml のファイル名（拡張子なし） | `blind-dart-renge-ring` |
| **Animation** | scml 内の `<animation name="...">` の値 | `renge-ring-dotted-line` |

---

## 初心者向け注意点・Tips

### 環境準備

- **Spriter** は無料版（Spriter B5）で十分です。Steam からもインストールできます
- **DST Mod Tools** は Steam のツールから無料でインストールできます
  - Steam ライブラリ → フィルタで「ツール」を選択 → 「Don't Starve Mod Tools」を検索

### 命名規則に注意

DST のアニメーションシステムでは、**ファイル名・フォルダ名がそのまま Bank 名や Build 名になります**。

- ファイル名・フォルダ名には **英小文字・数字・ハイフン(`-`)・アンダースコア(`_`)** のみ使用してください
- **スペースや日本語は使わないでください** — ビルド時にエラーになるか、ゲーム内で正しく読み込まれません
- scml ファイル名とプロジェクトフォルダ名は **一致させる** のが安全です
- PNG 画像のファイル名には特に命名規則はありません。自由に名前を付けられます

### Bank・Build・Animation の違い

初心者が最も混乱しやすいポイントです。

| 用語 | 何を指すか | 決まり方 |
|---|---|---|
| **Bank** | アニメーションのグループ名 | scml 内の Entity 名（`<entity name="...">`） |
| **Build** | 見た目（テクスチャ）のセット名 | scml のファイル名（拡張子なし） |
| **Animation** | 個々のアニメーション名 | scml 内の Animation 名（`<animation name="...">`） |

> 同じ Bank に対して Build を差し替えることで、**アニメーションの動きはそのままに見た目だけ変える**ことができます。DST のキャラクタースキンなどもこの仕組みを利用しています。

### ピボット（pivot）の重要性

- ピボットは画像の **回転中心点** です。この値がずれると、回転した画像が円の形にならず **バラバラに散らばって** しまいます
- Spriter の GUI 上で手動設定するとズレやすいため、**数値入力で正確に設定する** ことをおすすめします

### ビルドがうまくいかないとき

- `scml.exe` の実行でエラーが出る場合、**scml ファイルのパスにスペースや日本語が含まれていないか** 確認してください
- `buildanimation.py` でエラーが出る場合は `--ignoreexceptions` オプションを付けると詳細を無視して続行できますが、**まずはエラー内容を確認する** ことを推奨します
- 生成された ZIP が空、または `anim/` フォルダ内に ZIP が見つからない場合は、**プロジェクトフォルダ名と scml ファイル名が一致しているか** 再確認してください

### ゲーム内で確認する方法

MOD にアニメーション ZIP を配置したら、ゲーム内のコンソール（`~` キー）で動作確認できます：

```lua
-- コンソールでアニメーションをテスト表示する例
local fx = SpawnPrefab("yourprefab")
fx.AnimState:SetBank("blind-dart-renge")
fx.AnimState:SetBuild("blind-dart-renge-ring")
fx.AnimState:PlayAnimation("renge-ring-circle-frameless", true)
```

### 参考リンク

- [Don't Starve Together Modding Guide（Klei Forums）](https://forums.kleientertainment.com/forums/topic/59174-tutorial-the-artists-guide-to-dont-starve-together-modding/)
- [Spriter 公式サイト](https://brashmonkey.com/)
- [DST Mod Tools（Steam）](https://store.steampowered.com/app/250820/Dont_Starve_Mod_Tools/)

---

## ライセンス

このドキュメントは自由に利用・改変・再配布できます。DST MOD 制作にお役立てください。

---

<sub>Author: [pinpikokun](https://steamcommunity.com/profiles/76561198076111536/)</sub><br>
<sub>[![GitHub](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/pinpikokun/DST-teemo) [![Steam Workshop](https://img.shields.io/badge/Steam%20Workshop-Subscribe-blue?logo=steam)](https://steamcommunity.com/sharedfiles/filedetails/?id=390684095)</sub>
