# DST-ktools ガイド — ゲームアセットのデコンパイル・変換マニュアル（krane / ktech）

Don't Starve Together (DST) の **アニメーションアセット（.zip）** を、編集可能な **Spriter プロジェクト（.scml）+ PNG 画像** にデコンパイルする手順を、初心者向けにまとめたガイドです。

> **HTML 版**: [README.html](https://pinpikokun.github.io/DST-Mod-Guide/DST-ktools/README.html)

---

## 目次

1. [はじめに — ktools とは？](#はじめに--ktools-とは)
2. [ktools のダウンロードと配置](#ktools-のダウンロードと配置)
3. [krane — アニメーション ZIP のデコンパイル](#krane--アニメーション-zip-のデコンパイル)
4. [ktech — TEX ↔ PNG 画像変換](#ktech--tex--png-画像変換)
5. [実践例: 既存アニメーションを参考にオリジナルを作る](#実践例-既存アニメーションを参考にオリジナルを作る)
6. [コマンドリファレンス](#コマンドリファレンス)
7. [よくあるエラーと対処法](#よくあるエラーと対処法)
8. [Tips](#tips)

---

## はじめに — ktools とは？

DST のゲームアセット（キャラクター・アイテム・エフェクトのアニメーション）は、`.zip` ファイルに圧縮された **独自のバイナリ形式** で格納されています。

```
blow_dart.zip（ゲームのアニメーション）
├── anim.bin      ← アニメーション定義（バイナリ）
├── build.bin     ← ビルド情報（バイナリ）
└── atlas-0.tex   ← スプライトシート（TEX 形式）
```

これらのファイルはそのままでは中身を見ることも編集することもできません。**ktools** は、このバイナリ形式を **Spriter プロジェクト + 個別の PNG 画像** にデコンパイルしてくれるツールです。

```
blow_dart.zip
    │
    │  krane でデコンパイル
    ↓
blow_dart_decompiled/
├── blow_dart.scml     ← Spriter プロジェクト（アニメーション編集可能）
├── dart/
│   ├── dart-0.png     ← 個別スプライト画像
│   ├── dart-1.png
│   └── ...
├── flametail/
│   ├── flametail-0.png
│   └── ...
└── ...
```

### なぜデコンパイルが必要？

- **既存アニメーションの構造を理解する** — 公式キャラクターやアイテムの作りを参考にできる
- **スプライト画像を確認する** — パーツの大きさ・向き・枚数を把握できる
- **カスタムアニメーションのベースにする** — 既存アセットをコピーして改造できる

> **注意**: デコンパイルしたアセットを **そのまま再配布** することは、ゲームの利用規約に反する場合があります。あくまで **自分の MOD 開発の参考** として使用してください。

### ktools に含まれるツール

| ツール | 用途 |
|---|---|
| **krane** | アニメーション ZIP をデコンパイル → Spriter プロジェクト + PNG 画像 |
| **ktech** | TEX ↔ PNG の相互変換 |

> **ktech と TextureConverter の違い**:
> - **TextureConverter**（Don't Starve Mod Tools 同梱）は **PNG → TEX 変換** に特化しており、MOD 制作時に画像を TEX に変換するために使います
> - **ktech** は **TEX → PNG の逆変換** にも対応しており、ゲーム内の TEX ファイルを PNG に戻して中身を確認できます
> - どちらも PNG → TEX 変換ができますが、MOD 制作時の PNG → TEX 変換には公式の TextureConverter を使うのが無難です

---

## ktools のダウンロードと配置

### ダウンロード

ktools は GitHub で公開されています。

```
https://github.com/nsimplex/ktools/releases
```

上記ページから **最新バージョンの Windows 用バイナリ**（`ktools-*-win32.zip`）をダウンロードしてください。

### 配置

1. ダウンロードした ZIP を **任意の場所** に展開します

   おすすめの配置先:
   ```
   C:\Users\<ユーザー名>\Desktop\DST-ktools\ktools-4.4.4\
   ```

2. 展開すると以下のファイルが含まれています:

   ```
   ktools-4.4.4/
   ├── ktech.exe          ← TEX ↔ PNG 変換ツール
   ├── krane.exe          ← アニメーション ZIP デコンパイラ
   ├── coder.xml          ← ImageMagick 設定ファイル
   ├── colors.xml         ← 色設定ファイル
   └── *.dll              ← 動作に必要なライブラリ（すべて必要）
   ```

   > **重要**: `.exe` ファイルだけを別の場所にコピーしても動きません。`.dll` ファイルと `.xml` ファイルが同じフォルダに必要です。

### 動作確認

Git Bash を開いて以下のコマンドを実行します:

```bash
"/c/Users/<ユーザー名>/Desktop/DST-ktools/ktools-4.4.4/krane.exe" --version
```

バージョン番号が表示されれば OK です。

> **初心者向け Tip**: パスは自分の配置場所に合わせて変更してください。以降のガイドでは、パスを変数に保存して使う方法を説明します。

---

## krane — アニメーション ZIP のデコンパイル

krane は DST のアニメーション ZIP ファイルを **Spriter プロジェクト（.scml）+ 個別 PNG 画像** にデコンパイルするツールです。

### デコンパイルの手順

ここでは、DST 本体に含まれる `blow_dart.zip`（吹き矢のアニメーション）をデコンパイルする例で説明します。

#### ステップ 1: アニメーション ZIP を用意する

DST のアニメーションファイルは以下の場所にあります:

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\data\anim\
```

> **このフォルダには大量の ZIP ファイルがあります**。エクスプローラーの検索バーにファイル名を入力すると素早く見つけられます。

デコンパイルしたい ZIP ファイルを **作業フォルダにコピー** してください。

```
作業フォルダの例:
C:\Users\<ユーザー名>\Desktop\DST-ktools\
└── blow_dart.zip   ← ここにコピー
```

> **注意**: ゲーム本体のフォルダ内で直接作業しないでください。誤って元ファイルを壊すとゲームが起動しなくなる可能性があります。

#### ステップ 2: ZIP を展開する

コピーした ZIP ファイルを **通常の ZIP として展開** します（右クリック →「すべて展開」）。

```
DST-ktools/
├── blow_dart.zip
└── blow_dart/           ← 展開されたフォルダ
    ├── anim.bin         ← アニメーション定義
    ├── build.bin        ← ビルド情報
    └── atlas-0.tex      ← スプライトシート画像
```

> **初心者向け Tip**: ZIP を展開すると中に `.bin` と `.tex` ファイルが入っています。これらは DST 独自のバイナリ形式で、テキストエディタでは開けません。

#### ステップ 3: krane でデコンパイルする

展開したフォルダがある場所で **Git Bash** を開き、以下のコマンドを実行します:

```bash
# krane.exe のパスを変数に保存（自分の配置場所に合わせて変更）
KRANE="/c/Users/<ユーザー名>/Desktop/DST-ktools/ktools-4.4.4/krane.exe"

# デコンパイル実行
"$KRANE" blow_dart blow_dart_decompiled
```

- 第 1 引数: 展開した ZIP の中身があるフォルダ（`anim.bin` と `build.bin` が入っているフォルダ）
- 第 2 引数: 出力先フォルダ名（自動的に作成される）

#### ステップ 4: 結果を確認する

デコンパイルが成功すると、出力先フォルダに Spriter プロジェクトと個別の PNG 画像が生成されます:

```
blow_dart_decompiled/
├── blow_dart.scml              ← Spriter プロジェクトファイル
├── dart/                       ← ダート（矢）のスプライト
│   ├── dart-0.png
│   ├── dart-1.png
│   └── ...（9 枚）
├── flametail/                  ← 炎の尾エフェクト
│   ├── flametail-0.png
│   └── ...（5 枚）
├── grass/                      ← 筒の装飾
│   └── grass-0.png
├── gun/                        ← 筒本体
│   ├── gun-0.png
│   └── gun-1.png
├── pipe/                       ← パイプ
│   └── pipe-0.png
└── swap_blowdart_pipe/         ← 手持ち差し替え用
    ├── swap_blowdart_pipe-0.png
    └── swap_blowdart_pipe-2.png
```

生成された `.scml` ファイルを **Spriter** で開くと、アニメーションの構造（タイムライン、キーフレーム、各パーツの位置・回転・スケール）を確認できます。

> **Spriter の場所**:
> ```
> C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\Spriter\Spriter.exe
> ```

### Bank 名と Build 名を確認する

デコンパイル時に `-v`（verbose）オプションを付けると、Bank 名と Build 名が表示されます:

```bash
"$KRANE" -v blow_dart blow_dart_decompiled
```

> **Bank と Build とは？**
> - **Bank**: アニメーションの定義名。どのアニメーション（idle、attack など）が使えるかを決める
> - **Build**: 見た目の定義名。どのスプライト画像を使うかを決める
>
> Lua コードでは `AnimState:SetBank("bank名")` と `AnimState:SetBuild("build名")` で指定します。既存アセットの Bank/Build 名を確認することで、自分の MOD で正しい値を設定できます。

### Bank と Build の名前を変更する

デコンパイル時に Bank 名や Build 名を変更することもできます:

```bash
# Build 名を変更してデコンパイル
"$KRANE" --rename-build my_custom_dart blow_dart blow_dart_decompiled

# Bank 名を変更してデコンパイル
"$KRANE" --rename-bank my_custom_dart blow_dart blow_dart_decompiled
```

---

## ktech — TEX ↔ PNG 画像変換

ktech は DST の TEX 形式と PNG の相互変換を行うツールです。

### TEX → PNG 変換（ゲーム内画像の確認）

ゲーム内の TEX ファイルを PNG に変換して中身を確認できます。

```bash
# ktech.exe のパスを変数に保存
KTECH="/c/Users/<ユーザー名>/Desktop/DST-ktools/ktools-4.4.4/ktech.exe"

# TEX → PNG 変換
"$KTECH" atlas-0.tex atlas-0.png
```

> **どんなときに使う？**
> - ゲーム内のスプライトシート（`atlas-0.tex`）の中身を目視で確認したいとき
> - 既存 MOD の TEX ファイルがどんな画像か確認したいとき
> - デコンパイル前に、アトラス画像の全体像を見たいとき

### PNG → TEX 変換

```bash
"$KTECH" my_image.png my_image.tex
```

> **注意**: MOD 制作で使う PNG → TEX 変換は、公式の **TextureConverter** を使うのが推奨です（[DST-TextureConverter ガイド](../DST-TextureConverter/README.md) を参照）。ktech による変換はデバッグや確認用途に向いています。

### TEX ファイルの情報を表示する

```bash
"$KTECH" -i atlas-0.tex
```

TEX ファイルのサイズ、圧縮形式、ミップマップ情報などが表示されます。変換せずにファイルの仕様だけ確認したいときに便利です。

---

## 実践例: 既存アニメーションを参考にオリジナルを作る

ここでは、**既存の吹き矢アニメーションをデコンパイルして、オリジナルの飛翔体アニメーションを作る** という実践的なワークフローを紹介します。

### 全体の流れ

```
┌─────────────────────────────────────────────────────┐
│                                                       │
│  1. ゲームから参考アセットをコピー                      │
│       ↓                                               │
│  2. krane でデコンパイル                               │
│       ↓                                               │
│  3. Spriter でアニメーション構造を確認                  │
│       ↓                                               │
│  4. スプライト画像のサイズ・枚数・配置を把握            │
│       ↓                                               │
│  5. オリジナルのスプライト画像を描く                    │
│       ↓                                               │
│  6. Spriter で新しい .scml プロジェクトを作成          │
│       ↓                                               │
│  7. autocompiler でビルド → anim/*.zip                 │
│       ↓                                               │
│  8. MOD に組み込んでテスト                             │
│                                                       │
└─────────────────────────────────────────────────────┘
```

### ステップ 1: 参考アセットをデコンパイルする

```bash
KRANE="/c/Users/<ユーザー名>/Desktop/DST-ktools/ktools-4.4.4/krane.exe"

# 吹き矢の ZIP を展開済みのフォルダに対してデコンパイル
"$KRANE" blow_dart blow_dart_decompiled
```

### ステップ 2: Spriter で構造を確認する

`blow_dart_decompiled/blow_dart.scml` を Spriter で開きます。

確認するポイント:
- **アニメーション名**: タイムラインのドロップダウンからアニメーション一覧を確認（例: `idle`、`dart_pipe`）
- **パーツ構成**: どのフォルダにどんなスプライトがあるか
- **キーフレーム**: 各スプライトがどのタイミングで切り替わるか
- **位置・回転・スケール**: 各パーツのオフセットやサイズ変更

### ステップ 3: スプライト画像の仕様を把握する

デコンパイルされた PNG を画像エディタで開いて確認します:

| 確認項目 | 例（blow_dart の場合） |
|---|---|
| 画像サイズ | dart: 66×146 px、flametail: 約 110-141×55-65 px |
| 枚数 | dart: 9 枚、flametail: 5 枚 |
| 向き | 右向き |
| 背景 | 透過 PNG |

### ステップ 4: オリジナル画像を描く

参考アセットと **同じようなサイズ・向き** でオリジナルのスプライト画像を描きます。

> **ポイント**: 既存アセットと同じフレーム数・同じファイル命名規則にすると、Spriter プロジェクトの修正が最小限で済みます。

### ステップ 5: Spriter で新しいプロジェクトを作成

デコンパイルした `.scml` をコピーして、スプライト画像を差し替えることで効率的に新しいプロジェクトを作れます。

### ステップ 6: autocompiler でビルド

完成した Spriter プロジェクトを `exported` フォルダに配置して autocompiler でビルドします（詳しくは [メインガイドの Step 7](../README.md) を参照）。

---

## コマンドリファレンス

### krane

```bash
krane [オプション] <入力パス> <出力ディレクトリ>
```

| オプション | 説明 |
|---|---|
| `--build <name>` | 指定した Build 名のみを選択してデコンパイル |
| `--bank <name>` | 指定した Bank 名のアニメーションのみを選択（複数指定可） |
| `--rename-build <name>` | Build 名を変更してデコンパイル |
| `--rename-bank <name>` | Bank 名を変更してデコンパイル |
| `--mark-atlases` | アトラス画像をクリップ領域付きで PNG 保存（デバッグ用） |
| `--check-animation-fidelity` | Spriter 表現の忠実度をチェック |
| `-v` | 詳細な出力を表示（Bank/Build 名の確認に便利） |
| `-q` | 出力を抑制 |
| `--version` | バージョン表示 |
| `-h`, `--help` | ヘルプ表示 |

> **入力パスについて**: 入力パスがフォルダの場合、その中の `build.bin` と `anim.bin` が自動的に使用されます。アトラス画像（`atlas-*.tex`）の場所も build ファイルから自動判定されるため、引数に指定する必要はありません。

### ktech

```bash
ktech [オプション] <入力ファイル> <出力先>
```

#### TEX → PNG 変換時のオプション

| オプション | 説明 |
|---|---|
| `-Q <0-100>` | 出力画像の品質。デフォルト: 100 |
| `-i`, `--info` | 変換せずに TEX ファイルの情報を表示 |

#### PNG → TEX 変換時のオプション

| オプション | 説明 |
|---|---|
| `--atlas <path>` | 生成するアトラス（XML）のパス |
| `-c <形式>` | 圧縮形式: `dxt1`, `dxt3`, `dxt5`, `rgb`, `rgba`。デフォルト: `dxt5` |
| `-f <フィルター>` | ミップマップ生成フィルター: `lanczos`, `mitchell`, `bicubic`, `catrom`, `cubic`, `box`。デフォルト: `lanczos` |
| `-t <type>` | テクスチャタイプ: `1d`, `2d`, `3d`, `cube`。デフォルト: `2d` |
| `--no-premultiply` | アルファ事前乗算を行わない |
| `--no-mipmaps` | ミップマップを生成しない |

#### 共通オプション

| オプション | 説明 |
|---|---|
| `--width <px>` | 出力の幅を固定（高さ未指定ならアスペクト比維持） |
| `--height <px>` | 出力の高さを固定（幅未指定ならアスペクト比維持） |
| `--pow2` | 幅と高さを 2 のべき乗に切り上げ |
| `--square` | 正方形に変換（幅と高さの大きい方に合わせる） |
| `--extend` | リサイズではなく余白を追加して拡張 |
| `--extend-left` | 余白を左側に追加（`--extend` を含む） |
| `-v` | 詳細な出力を表示 |
| `-q` | 出力を抑制 |

---

## よくあるエラーと対処法

### krane 実行時のエラー

| 症状 | 原因 | 解決法 |
|---|---|---|
| `Could not find build.bin` | 入力フォルダに `build.bin` がない | ZIP をきちんと展開したか確認。展開後のフォルダを指定する |
| `Could not find anim.bin` | 入力フォルダに `anim.bin` がない | 同上 |
| 出力フォルダに何も生成されない | DLL ファイルが見つからない | `krane.exe` と同じフォルダにすべての `.dll` があるか確認 |
| 文字化けしたエラーメッセージ | パスに日本語が含まれている | 日本語を含まないパスに配置する |

### ktech 実行時のエラー

| 症状 | 原因 | 解決法 |
|---|---|---|
| 変換後の PNG が真っ黒 | TEX ファイルが壊れている | 別の TEX ファイルで試す |
| `Not a TEX file` | 入力ファイルが TEX 形式ではない | ファイルの拡張子と中身を確認 |

### 共通の注意点

- **パスに日本語やスペースを含む場合** はパスを `"` で囲んでください
- **DLL がすべて揃っていることを確認** — `krane.exe` や `ktech.exe` は単体では動作しません
- **Git Bash で実行する場合** はパスのバックスラッシュ `\` をスラッシュ `/` に変換してください

---

## Tips

### よく使うコマンドをまとめる

毎回パスを入力するのは面倒なので、作業開始時にまず変数を設定しておくと便利です:

```bash
# 作業開始時にこの 2 行を実行しておく
KRANE="/c/Users/<ユーザー名>/Desktop/DST-ktools/ktools-4.4.4/krane.exe"
KTECH="/c/Users/<ユーザー名>/Desktop/DST-ktools/ktools-4.4.4/ktech.exe"

# あとは変数で呼び出せる
"$KRANE" input_folder output_folder
"$KTECH" input.tex output.png
```

### ゲームアセットの場所

DST のアニメーション ZIP ファイルは以下にあります:

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\data\anim\
```

よく参考にされるアセット:

| ファイル名 | 内容 |
|---|---|
| `wilson.zip` | ウィルソン（キャラクター） |
| `blow_dart.zip` | 吹き矢（武器 + 飛翔体） |
| `spear.zip` | 槍（武器） |
| `axe.zip` | 斧（ツール） |
| `torchfire.zip` | たいまつの炎（エフェクト） |

### アトラス画像を直接確認する

krane でフルデコンパイルする前に、アトラス画像だけを PNG で見たい場合:

```bash
# アトラスにクリップ領域をマークした画像を出力
"$KRANE" --mark-atlases blow_dart atlas_preview
```

これにより、スプライトシートのどの領域がどのパーツに対応しているかを視覚的に確認できます。

### TEX ファイルの情報だけ確認する

変換せずに TEX ファイルの仕様を確認したいとき:

```bash
"$KTECH" -i atlas-0.tex
```

---

## 参考リンク

- [ktools GitHub リポジトリ](https://github.com/nsimplex/ktools) — ソースコードとリリース
- [DST-Spriter ガイド](../DST-Spriter/README.md) — アニメーション作成の詳細
- [DST-TextureConverter ガイド](../DST-TextureConverter/README.md) — PNG → TEX 変換（MOD 制作用）
- [メインガイド（はじめてのキャラクター MOD）](../README.md)

---

## ライセンス

このドキュメントは自由に利用・改変・再配布できます。DST MOD 制作にお役立てください。

---

<sub>Author: [pinpikokun](https://steamcommunity.com/profiles/76561198076111536/)</sub><br>
<sub>[![GitHub](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/pinpikokun/DST-teemo) [![Steam Workshop](https://img.shields.io/badge/Steam%20Workshop-Subscribe-blue?logo=steam)](https://steamcommunity.com/sharedfiles/filedetails/?id=390684095)</sub>
