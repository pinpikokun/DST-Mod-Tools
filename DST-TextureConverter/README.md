# DST TextureConverter ガイド — PNG to TEX 変換マニュアル

Don't Starve Together (DST) の MOD 開発で必要になる **PNG → TEX 変換** の手順を、初心者向けにまとめたガイドです。

> **HTML 版**: [README.html](https://pinpikokun.github.io/DST-Mod-Guide/DST-TextureConverter/README.html)

---

## 目次

1. [はじめに — TEX ファイルとは？](#はじめに--tex-ファイルとは)
2. [Don't Starve Mod Tools のインストール](#dont-starve-mod-tools-のインストール)
3. [PNG 画像の準備](#png-画像の準備)
4. [Git Bash について](#git-bash-について)
5. [PNG → TEX 変換](#png--tex-変換)
6. [XML アトラスファイルの作成](#xml-アトラスファイルの作成)
7. [MOD フォルダへの配置](#mod-フォルダへの配置)
8. [Lua ソースでの使い方（サンプルコード）](#lua-ソースでの使い方サンプルコード)
9. [よくあるエラーと対処法](#よくあるエラーと対処法)
10. [Tips](#tips)
11. [付録: 武器・アイテム MOD に必要な画像の全体像](#付録-武器アイテム-mod-に必要な画像の全体像)

---

## はじめに — TEX ファイルとは？

DST はゲーム内で **PNG をそのまま使うことができません**。代わりに Klei 独自の `.tex` 形式と、テクスチャの領域情報を持つ `.xml`（アトラスファイル）をセットで使います。

```
PNG画像 ──(TextureConverter)──> .tex ファイル
                                  +
                                .xml ファイル（手動作成）
                                  ↓
                            MOD から参照して使う
```

MOD で画像を追加したい場合は、必ずこの **TEX + XML** のペアを用意する必要があります。

---

## Don't Starve Mod Tools のインストール

TextureConverter は **Don't Starve Mod Tools** に同梱されています。Steam から無料でインストールできます。

### 手順

1. **Steam を開く**
2. **ライブラリ** を選択
3. 検索バーに `Don't Starve Mod Tools` と入力（見つからない場合は、検索バーの横にある絞り込みで「ツール」にチェックを入れてください）
4. 表示された **Don't Starve Mod Tools** をクリックし、**インストール**

### インストール先の確認

デフォルトでは以下のパスにインストールされます:

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\
```

> **注意:** Steam のライブラリフォルダを変更している場合は、パスが異なります。
> Steam → 設定 → ストレージ からインストール先を確認してください。

TextureConverter の実行ファイルは以下にあります:

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\tools\bin\TextureConverter.exe
```

> **初心者向けTip:** このパスは長いので、後の作業でコピー＆ペーストしやすいようにメモ帳などに控えておくと便利です。

---

## PNG 画像の準備

TEX に変換する PNG 画像には **守らなければならないルール** があります。ここを間違えると、変換はできてもゲームがクラッシュします。

### 必須ルール: サイズは「2のべき乗」の正方形

| OK | NG |
|---|---|
| 32x32, 64x64, 128x128, 256x256, 512x512 | 100x100, 64x128, 50x50, 200x200 |

> **なぜ正方形・2のべき乗でなければならないの？**
>
> GPU のテクスチャ処理が2のべき乗サイズに最適化されているためです。DST は起動時にテクスチャサイズをチェックし、条件を満たさない場合は **テクスチャアサーションエラー** を出してクラッシュします。

### 用途別の推奨サイズ

| 用途 | 推奨サイズ | 備考 |
|------|-----------|------|
| インベントリアイコン | **64x64** | DST 公式アイコンと同じサイズ。十分鮮明 |
| マップアイコン | 64x64 | ミニマップに表示するアイコン |
| キャラクター選択画面の大きいポートレート | 256x256 ~ 512x512 | 大きいほどファイルサイズ・VRAM消費が増える |
| HUD / UI 要素 | 用途に応じて | 2のべき乗正方形であれば自由 |

> **初心者向けTip:** 迷ったら **64x64** で作ってください。インベントリアイコンは DST 公式も 64x64 です。128x128 にしてもゲーム内での表示サイズは変わらず、ファイルサイズと VRAM 消費が約4倍になるだけです。

### デザインのポイント

- **くっきりはっきりした色使い** を心がける
- ゲーム画面は暗い場面（夜・洞窟）が多く、暗い色や淡い色のアイコンは背景に溶けて見えなくなる
- コントラストの高い明るめの色を使い、小さいサイズでも一目で識別できるデザインにする
- 透過（アルファチャンネル）を使う場合は、PNG を **32bit RGBA** で保存する

---

## Git Bash について

本ガイドの変換コマンド例は主に **Git Bash** の書式で記載しています。コマンドプロンプト（cmd.exe）でも実行できますが、Git Bash を使うとシェル変数やループが使えて便利です。

### Git Bash とは？

**Git Bash** は、Windows 上で Linux/Mac 風のコマンド（bash）を使えるようにするターミナルです。バージョン管理ツール「Git for Windows」をインストールすると一緒に付いてきます。

> **初心者向けTip:** 「Git って何？」という方も心配いりません。Git Bash は単に「便利なコマンド入力画面」として使えます。Git そのものの知識は TEX 変換には不要です。

### インストール方法

1. [Git for Windows 公式サイト（https://gitforwindows.org/）](https://gitforwindows.org/) にアクセス
2. **Download** ボタンをクリックしてインストーラーをダウンロード
3. ダウンロードした `.exe` を実行
4. 基本的に **すべてデフォルト設定のまま「Next」を押し続けて** インストール完了

> **初心者向けTip:** インストール中にたくさんの設定画面が出てきますが、変換作業だけが目的ならすべてデフォルトのままで問題ありません。

### 起動方法

インストール後、以下のいずれかの方法で起動できます:

- **方法1（おすすめ）:** エクスプローラーで作業したいフォルダを開き、**フォルダ内の空白部分を右クリック** →「**Git Bash Here**」を選択
- **方法2:** スタートメニューから「Git Bash」を検索して起動
- **方法3:** デスクトップにある「Git Bash」ショートカットをダブルクリック

**方法1 が一番おすすめです。** 方法2・3 で起動すると、ホームディレクトリ（`C:\Users\ユーザー名`）で開くため、PNG があるフォルダまで `cd` コマンドで移動する必要があり面倒です。方法1 なら、開いたフォルダがそのまま作業ディレクトリになります。

具体的な使い方は [PNG → TEX 変換](#png--tex-変換) セクションでステップバイステップで説明しています。

### Git Bash を使わない場合

Git Bash をインストールしたくない場合は、Windows 標準の **コマンドプロンプト（cmd.exe）** や **PowerShell** でも変換できます。ただし本ガイドのシェル変数 (`$TEXCONV`) やループ (`for ... done`) の書式はそのままでは使えないため、各セクションの「コマンドプロンプトの場合」の例を参考にしてください。

---

## PNG → TEX 変換

### 作業フォルダと PNG の配置

変換作業は **PNG ファイルがあるフォルダ** で行うのが一番簡単です。出力先（`-o`）も同じフォルダにすると、TEX がそのまま横に生成されます。

おすすめの流れ:

1. MOD フォルダ内に画像の配置先フォルダを **先に作っておく**
2. そのフォルダに PNG を置く
3. **そのフォルダで** ターミナルを開く ← ここが重要！
   - **Git Bash の場合:** フォルダの中の空白部分を **右クリック** → **「Git Bash Here」** を選択
   - **コマンドプロンプトの場合:** エクスプローラーの **アドレスバーをクリック** → `cmd` と入力して **Enter**
4. 変換コマンドを実行
5. 同じフォルダに TEX が生成される → そのまま MOD から参照できる

```
mods/my_awesome_mod/
└── images/
    └── inventoryimages/       ← ★ ここで Git Bash / cmd を開く
        ├── my_item_icon.png   ← ここに PNG を置いて…
        ├── my_item_icon.tex   ← ここに TEX が生成される
        └── my_item_icon.xml   ← XML も同じ場所に手動作成
```

> **初心者向けTip:** PNG は変換後も残しておいて構いません。ゲームは `.tex` と `.xml` しか参照しないので、PNG があっても影響はありません。後でアイコンを修正したくなったとき、元の PNG があるとすぐに再変換できて便利です。

> **別の場所で作業する場合:** もちろん、デスクトップなど好きな場所で変換して、生成された `.tex` を後から MOD フォルダにコピーしても OK です。その場合は `-o` にフルパスを指定するか、変換後に手動でファイルを移動してください。

### 変換の手順（Git Bash の場合）

ここでは **「自作アイテムのインベントリアイコン（持ち物欄に表示される小さい画像）を MOD に追加する」** という、最もよくあるケースを例に説明します。

DST の MOD では、インベントリアイコンの画像を `images/inventoryimages/` というフォルダに置くのがお決まりのルールです。このフォルダ名は DST の公式 MOD や既存の人気 MOD でも共通で使われている慣習的な配置場所です。

最初のフォルダ作成から変換完了までの全手順を説明します。

**ステップ 1: MOD フォルダ内に `images/inventoryimages/` フォルダを作る**

まず、エクスプローラーで自分の MOD フォルダを開きます:

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\mods\my_awesome_mod\
```

> **初心者向けTip:** MOD フォルダがまだない場合は、`mods` フォルダの中に自分の MOD 名のフォルダ（例: `my_awesome_mod`）を新規作成してください。

この中に、インベントリアイコンを格納するためのフォルダを作ります。`images` フォルダを作り、さらにその中に `inventoryimages` フォルダを作ってください:

```
my_awesome_mod\          ← エクスプローラーでここを開いて…
│
│  ここで 右クリック → 新規作成 → フォルダー で「images」を作る
│
└── images\              ← 作った images フォルダを開いて…
    │
    │  同じく「inventoryimages」フォルダを作る
    │
    └── inventoryimages\ ← 完成！ アイコン画像はここに入れる
```

> **なぜこのフォルダ構成なの？** DST の MOD では画像の種類ごとに決まった置き場所があります。後の Lua コードで `"images/inventoryimages/ファイル名.tex"` のようにパスを指定するため、この構成に合わせておく必要があります。

#### 画像の種類と配置先フォルダ一覧

| フォルダパス | 用途 | 説明 | 推奨サイズ |
|------------|------|------|-----------|
| `images/inventoryimages/` | インベントリアイコン | 持ち物欄・クラフト画面に表示される小さいアイコン | 64x64 |
| `images/minimap/` | ミニマップアイコン（建造物等） | マップ上に表示されるアイコン。`AddMinimapAtlas` で登録する | 64x64 |
| `images/map_icons/` | マップアイコン（キャラクター） | キャラクター MOD のミニマップアイコン | 64x64 |
| `images/saveslot_portraits/` | セーブスロットのポートレート | セーブ選択画面に表示されるキャラクターの顔 | 128x128 |
| `images/selectscreen_portraits/` | キャラクター選択画面ポートレート | キャラクター選択時に表示される立ち絵 | 256x512 |
| `images/avatars/` | アバター | チャット送信時にメッセージ横に表示されるアイコン | 64x64 |
| `bigportraits/` | 大きいポートレート | キャラクター選択画面の背景に表示される大きな立ち絵 | 512x512 以上 |
| `images/colour_cubes/` | カラーキューブ | 画面全体の色調補正フィルター（上級者向け） | ― |
| `images/` | その他の UI 画像 | HUD・ボタン・背景など汎用的な画像 | 用途による |

> **初心者向けTip:** 最初は `images/inventoryimages/`（アイテムアイコン）だけ覚えておけば大丈夫です。キャラクター MOD を作る段階になったら、ポートレート系のフォルダも使うことになります。

> **武器・道具などのアイテムを作りたい場合:** インベントリアイコン以外に、地面に落ちた時やキャラクターが装備した時の「ゲーム内スプライト」が必要になります。これは TEX 変換ではなく Spriter アニメーションという別の仕組みで作ります。詳しくは [付録: 武器・アイテム MOD に必要な画像の全体像](#付録-武器アイテム-mod-に必要な画像の全体像) を参照してください。

**ステップ 2: PNG をそのフォルダに入れる**

作成した `inventoryimages` フォルダに、変換したい PNG ファイルをコピーまたは移動します:

```
my_awesome_mod\
└── images\
    └── inventoryimages\
        └── my_item_icon.png   ← ここに PNG を入れる
```

**ステップ 3: PNG があるフォルダで Git Bash を開く**

エクスプローラーで **`inventoryimages` フォルダの中を表示した状態** で:

1. `my_item_icon.png` が見えていることを確認
2. ファイルが無い場所（**空白部分**）を **右クリック**
3. メニューから **「Git Bash Here」** を選択

```
┌─────────────────────────────────────────────────────┐
│ ← → ↑  inventoryimages                              │
├─────────────────────────────────────────────────────┤
│                                                     │
│  📄 my_item_icon.png                                │
│                                                     │
│             ┌──────────────────┐                    │
│             │ Open in Terminal  │                    │
│             │ Git Bash Here ◀── ── ── これをクリック！
│             │ ...              │                    │
│             └──────────────────┘                    │
└─────────────────────────────────────────────────────┘
```

> **注意:** `inventoryimages` フォルダそのものを右クリックするのではなく、**フォルダの中に入ってから** 空白部分を右クリックしてください。フォルダを右クリックしてしまうと、一つ上の階層（`images`）で Git Bash が開いてしまいます。

黒い画面（Git Bash）が開きます。タイトルバーや `$` の左側にフォルダパスが表示されるので、`inventoryimages` で開けたか確認してください:

```
admin@PC MINGW64 ~/.../my_awesome_mod/images/inventoryimages
$                        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                          ↑ ここに inventoryimages が表示されていれば OK
```

**ステップ 4: 変換コマンドを実行する**

開いた Git Bash に以下の **2行** をコピー＆ペーストして Enter を押します:

```bash
TEXCONV="C:/Program Files (x86)/Steam/steamapps/common/Don't Starve Mod Tools/mod_tools/tools/bin/TextureConverter.exe"

"$TEXCONV" -i my_item_icon.png -o my_item_icon.tex -f bc3 -p opengl --mipmap --premultiply
```

> **初心者向けTip:** Git Bash へのペーストは `Ctrl+V` ではなく、**右クリック → Paste**、または **Shift+Insert** です。`Ctrl+V` だと動かないので注意してください。

> **初心者向けTip:** 2行を一度にペーストしても、1行目（変数の設定）だけ実行されることがあります。その場合は Enter を押してから、2行目をもう一度ペーストして Enter を押してください。

**ステップ 5: 変換結果を確認する**

エクスプローラーに戻ると（または Git Bash で `ls` と入力して Enter）、同じフォルダに `my_item_icon.tex` が生成されているはずです:

```
inventoryimages/
├── my_item_icon.png   ← 元画像（そのまま残る）
└── my_item_icon.tex   ← 新しく生成された！
```

> **初心者向けTip:** エクスプローラーにファイルが表示されない場合は、F5 キーを押して表示を更新してみてください。

### 変換の手順（コマンドプロンプトの場合）

Git Bash を使わない場合は、Windows 標準のコマンドプロンプトでも変換できます。フォルダ作成と PNG 配置は Git Bash の場合のステップ 1～2 と同じです。

**ステップ 1: PNG があるフォルダをエクスプローラーで開く**

Git Bash のステップ 1～2 と同様に、`inventoryimages` フォルダを作成し、PNG を入れた状態にしておきます。エクスプローラーで **`inventoryimages` フォルダの中** を表示してください:

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\mods\my_awesome_mod\images\inventoryimages\
```

**ステップ 2: そのフォルダでコマンドプロンプトを開く**

エクスプローラーの **アドレスバー**（上部にフォルダパスが表示されている部分）をクリックすると、テキストが編集可能になります。表示されているパスを全部消して `cmd` と入力し、Enter を押します:

```
┌───────────────────────────────────────────────────────────────────┐
│ ← → ↑  │ ...\my_awesome_mod\images\inventoryimages               │
│         │                                                         │
│         │  ↑ ここをクリックすると…                                │
│         │                                                         │
│         ├─────────────────────────────────────────────────────────┤
│         │  cmd                                                    │
│         │  ↑ 入力欄に変わるので cmd と打って Enter！              │
└───────────────────────────────────────────────────────────────────┘
```

`inventoryimages` フォルダをカレントディレクトリとしてコマンドプロンプトが開きます。プロンプトの表示でパスを確認してください:

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\mods\my_awesome_mod\images\inventoryimages>
                                                                                        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                                                                                          ↑ inventoryimages になっていれば OK
```

**ステップ 3: 変換コマンドを実行する**

以下の **1行** をコピー＆ペーストして Enter を押します:

```cmd
"C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\tools\bin\TextureConverter.exe" -i my_item_icon.png -o my_item_icon.tex -f bc3 -p opengl --mipmap --premultiply
```

> **初心者向けTip:** コマンドプロンプトの場合はシェル変数（`$TEXCONV`）が使えないため、TextureConverter.exe のフルパスを毎回書く必要があります。長いですが、一度コピペすれば OK です。

### コマンドの仕組み

```bash
TEXCONV="C:/Program Files (x86)/Steam/steamapps/common/Don't Starve Mod Tools/mod_tools/tools/bin/TextureConverter.exe"

"$TEXCONV" -i input.png -o output.tex -f bc3 -p opengl --mipmap --premultiply
```

1行目で TextureConverter.exe のパスを `TEXCONV` という変数に保存し、2行目で `$TEXCONV` として呼び出しています。毎回長いパスを打たなくて済む Git Bash ならではの書き方です。

### 実行例

```bash
TEXCONV="C:/Program Files (x86)/Steam/steamapps/common/Don't Starve Mod Tools/mod_tools/tools/bin/TextureConverter.exe"

# インベントリアイコンを変換
"$TEXCONV" -i my_item_icon.png -o my_item_icon.tex -f bc3 -p opengl --mipmap --premultiply
```

### オプション解説

| オプション | 値 | 説明 |
|-----------|-----|------|
| `-i` | 入力ファイル | 変換元の PNG 画像パス |
| `-o` | 出力ファイル | 出力先の .tex ファイルパス |
| `-f` | `bc3` | ピクセルフォーマット。DXT5 相当で、**アルファチャンネル（透過）に対応** |
| `-p` | `opengl` | プラットフォーム指定。**DST は OpenGL** なので必ず `opengl` を指定 |
| `--mipmap` | ― | ミップマップを生成する。**省略するとゲーム起動時にクラッシュする** |
| `--premultiply` | ― | アルファ事前乗算。**省略すると半透明部分の描画がおかしくなる** |

> **重要:** `--mipmap` と `--premultiply` は常に付けてください。省略すると不具合が発生します。初心者のうちは「おまじない」として必ず付けるものだと覚えてしまいましょう。

### 複数ファイルをまとめて変換する例

PNG ファイルが複数ある場合は、bash のループを使うと便利です。

```bash
TEXCONV="C:/Program Files (x86)/Steam/steamapps/common/Don't Starve Mod Tools/mod_tools/tools/bin/TextureConverter.exe"

for png in *.png; do
    tex="${png%.png}.tex"
    "$TEXCONV" -i "$png" -o "$tex" -f bc3 -p opengl --mipmap --premultiply
    echo "Converted: $png -> $tex"
done
```

---

## XML アトラスファイルの作成

TEX ファイルだけでは DST は画像を読み込めません。**テクスチャのどの領域を使うか** を指定する XML ファイル（アトラスファイル）が必要です。

### 基本テンプレート（1枚の TEX に1つの画像を入れる場合）

`my_item_icon.tex` に対応する `my_item_icon.xml` を同じフォルダに作成します:

```xml
<Atlas><Texture filename="my_item_icon.tex" /><Elements><Element name="my_item_icon.tex" u1="0" u2="1" v1="0" v2="1" /></Elements></Atlas>
```

> **初心者向けTip:** テキストエディタ（メモ帳・VSCode など）で新規ファイルを作り、上記をコピペして保存するだけでOKです。ファイル名は **TEX ファイルと同じ名前で、拡張子だけ `.xml`** にしてください。

### UV座標（u1/u2/v1/v2）について

| パラメータ | 意味 |
|-----------|------|
| `u1="0"` | テクスチャの左端（横方向の開始位置） |
| `u2="1"` | テクスチャの右端（横方向の終了位置） |
| `v1="0"` | テクスチャの上端（縦方向の開始位置） |
| `v2="1"` | テクスチャの下端（縦方向の終了位置） |

0～1 の範囲でテクスチャのどの部分を切り出すかを指定します。**1枚のテクスチャに1つの画像しか入っていない場合は、常に `0, 1, 0, 1`（テクスチャ全体）で OK です。** この値を変える必要があるのは、1枚のテクスチャに複数画像を詰め込む「テクスチャアトラス」を作る場合のみです。

> **初心者向けTip:** UV座標は「画像全体を使う」という意味の `0, 1, 0, 1` のままで大丈夫です。変更する必要はほぼありません。

### 重要な注意: ファイル名の一致

XML 内の `filename` と `name` は **TEX ファイル名と完全に一致** させてください。大文字・小文字、スペース、拡張子すべて一致しないとゲームが読み込めません。

```xml
<!-- OK -->
<Texture filename="my_icon.tex" />
<Element name="my_icon.tex" ... />

<!-- NG: 名前が一致していない -->
<Texture filename="my_icon.tex" />
<Element name="My_Icon.tex" ... />
```

---

## MOD フォルダへの配置

### DST の MOD フォルダ構造

DST の MOD は以下の場所にあります:

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\mods\<MOD名>\
```

一般的な MOD のフォルダ構成:

```
mods/
└── my_awesome_mod/
    ├── modinfo.lua          ← MOD の情報（名前・説明・バージョン等）
    ├── modmain.lua          ← MOD のメインスクリプト
    ├── scripts/             ← Lua スクリプト
    │   ├── prefabs/         ← プレハブ定義（アイテム・キャラクター等）
    │   │   └── my_item.lua
    │   └── components/      ← コンポーネント定義
    ├── images/              ← 画像ファイル（.tex + .xml）
    │   └── inventoryimages/ ← インベントリアイコン
    │       ├── my_item_icon.tex
    │       └── my_item_icon.xml
    └── exported/            ← アニメーション用ファイル（使わない場合は不要）
```

### TEX と XML の配置場所

用途に応じたフォルダに .tex と .xml をセットで配置します:

| 用途 | 配置先 |
|------|--------|
| インベントリアイコン | `images/inventoryimages/` |
| マップアイコン（キャラクター） | `images/map_icons/` |
| マップアイコン（建造物等） | `images/minimap/` |
| HUD・UI 画像 | `images/` |

> **初心者向けTip:** フォルダが存在しない場合は自分で作成してください。例えば `images/inventoryimages/` フォルダがなければ、`images` フォルダを作り、その中に `inventoryimages` フォルダを作ります。

> **重要:** `.tex` と `.xml` は **必ず同じフォルダに** 置いてください。別々のフォルダに置くと読み込みに失敗します。

---

## Lua ソースでの使い方（サンプルコード）

### 1. Asset 宣言（modmain.lua）

MOD で画像を使うには、まず `modmain.lua` でアセットとして宣言する必要があります。

```lua
-- modmain.lua

-- アセット宣言: ゲーム起動時にこれらのファイルを読み込む
Assets = {
    Asset("IMAGE", "images/inventoryimages/my_item_icon.tex"),
    Asset("ATLAS", "images/inventoryimages/my_item_icon.xml"),
}
```

> **初心者向けTip:**
> - `Asset("IMAGE", ...)` が TEX ファイル、`Asset("ATLAS", ...)` が XML ファイルに対応します
> - **必ず IMAGE と ATLAS の両方を宣言** してください。片方だけだと動きません
> - パスは MOD フォルダからの相対パスです（`mods/my_mod/` 以降の部分）

### 2. プレハブでの Asset 宣言（scripts/prefabs/ 内）

> **プレハブ（Prefab）とは？** DST のゲーム内に存在する「もの」はすべてプレハブです。アイテム（斧・槍・食べ物）、建造物（科学マシン・焚き火）、モンスター（クモ・ハウンド）、キャラクター、さらには木や岩までプレハブとして定義されています。「こういう見た目で、こういう機能を持つオブジェクト」を1つの Lua ファイルにまとめた設計図のようなものです。MOD で新しいアイテムやモンスターを追加する場合は、`scripts/prefabs/` フォルダに自分のプレハブファイルを作ることになります。

プレハブ定義ファイル内でもアセットを宣言できます。modmain.lua ではなくプレハブ側で宣言する方法もあります:

```lua
-- scripts/prefabs/my_item.lua

local assets = {
    Asset("IMAGE", "images/inventoryimages/my_item_icon.tex"),
    Asset("ATLAS", "images/inventoryimages/my_item_icon.xml"),
}

local function fn()
    local inst = CreateEntity()
    -- ... エンティティのセットアップ ...
    return inst
end

return Prefab("my_item", fn, assets)
```

### 3. インベントリアイコンとして使う

アイテムのインベントリアイコンとして表示する場合:

```lua
-- scripts/prefabs/my_item.lua 内の fn() 関数

local function fn()
    local inst = CreateEntity()

    inst.entity:AddTransform()
    inst.entity:AddAnimState()
    inst.entity:AddNetwork()

    MakeInventoryPhysics(inst)

    -- ここでインベントリアイコンを設定
    inst:AddComponent("inventoryitem")
    inst.components.inventoryitem.atlasname = "images/inventoryimages/my_item_icon.xml"
    inst.components.inventoryitem.imagename = "my_item_icon.tex"

    inst:AddComponent("inspectable")

    inst.entity:SetPristine()
    if not TheWorld.ismastersim then
        return inst
    end

    -- サーバー側の設定 ...

    return inst
end
```

> **初心者向けTip:**
> - `atlasname` には **XML ファイルのフルパス**（MOD ルートからの相対パス）を指定します
> - `imagename` には **TEX ファイルのファイル名のみ**（パスなし）を指定します
> - この2つの違いを間違えるとアイコンが表示されません

### 4. レシピ（クラフト画面）でアイコンを表示する

アイテムのクラフトレシピでカスタムアイコンを表示する場合:

```lua
-- modmain.lua

-- アセット宣言
Assets = {
    Asset("IMAGE", "images/inventoryimages/my_item_icon.tex"),
    Asset("ATLAS", "images/inventoryimages/my_item_icon.xml"),
}

-- レシピの追加
AddRecipe2(
    "my_item",                              -- レシピ名（プレハブ名と同じ）
    {                                        -- 材料
        Ingredient("twigs", 2),
        Ingredient("rocks", 3),
    },
    TECH.SCIENCE_ONE,                        -- 必要なテクノロジー
    {
        atlas = "images/inventoryimages/my_item_icon.xml",  -- アトラス（XML）
        image = "my_item_icon.tex",                          -- 画像（TEX）
    },
    { "TOOLS" }                              -- フィルタータブ
)
```

### 5. ミニマップアイコンとして使う

```lua
-- modmain.lua

Assets = {
    Asset("IMAGE", "images/minimap/my_map_icon.tex"),
    Asset("ATLAS", "images/minimap/my_map_icon.xml"),
}

-- ミニマップアイコンを追加
AddMinimapAtlas("images/minimap/my_map_icon.xml")
```

```lua
-- scripts/prefabs/my_structure.lua 内

inst.entity:AddMiniMapEntity()
inst.MiniMapEntity:SetIcon("my_map_icon.tex")
```

---

## よくあるエラーと対処法

### ゲーム起動時にクラッシュする

| 原因 | 対処 |
|------|------|
| PNG が2のべき乗サイズでない | 画像を 64x64, 128x128 等にリサイズする |
| PNG が正方形でない | 正方形にする（64x128 → 128x128 等） |
| `--mipmap` を付けずに変換した | `--mipmap` を付けて再変換する |

### アイコンが表示されない（白い四角・透明になる）

| 原因 | 対処 |
|------|------|
| Asset 宣言が漏れている | `modmain.lua` に IMAGE と ATLAS の両方を宣言する |
| パスが間違っている | パスの大文字/小文字、スラッシュの向きを確認する |
| XML 内の filename/name が TEX と一致していない | XML を開いてファイル名を確認・修正する |
| TEX と XML が別フォルダにある | 同じフォルダに配置する |

### 半透明の描画がおかしい（白い縁・色が変）

| 原因 | 対処 |
|------|------|
| `--premultiply` を付けずに変換した | `--premultiply` を付けて再変換する |

### `"Could not find an atlas for ..."` というエラー

Asset 宣言のパスと、実際のファイル配置が一致していません。以下をチェックしてください:

1. `Asset("ATLAS", "images/inventoryimages/my_icon.xml")` のパスが正しいか
2. そのパスに実際に `.xml` ファイルが存在するか
3. XML 内の `filename` が同じフォルダにある `.tex` ファイル名と一致しているか

### TextureConverter 実行時に何も起きない・エラーが出る

| 原因 | 対処 |
|------|------|
| パスが間違っている | `TextureConverter.exe` のフルパスを確認する |
| 入力 PNG が見つからない | PNG ファイルのパスを確認する。パスにスペースがある場合は `"` で囲む |
| Don't Starve Mod Tools が未インストール | Steam からインストールする |

---

## Tips

### パスのスラッシュについて

- **Lua ソースコード内** では `/`（スラッシュ）を使う: `"images/inventoryimages/icon.tex"`
- **Windows のコマンドプロンプト** では `\`（バックスラッシュ）を使う
- **Git Bash** ではどちらも使えるが `/` が無難

### TEX → PNG に戻す（逆変換）

TextureConverter で TEX から PNG に戻すこともできます:

```bash
"$TEXCONV" -i input.tex -o output.png
```

デバッグや、他の MOD のテクスチャを確認したいときに便利です。

### 変換作業の効率化

毎回長いパスを打つのが面倒な場合は、変換用のバッチファイル（`.bat`）を作っておくと便利です:

```bat
@echo off
set TEXCONV="C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\tools\bin\TextureConverter.exe"

if "%~1"=="" (
    echo 使い方: convert.bat input.png output.tex
    exit /b 1
)

%TEXCONV% -i "%~1" -o "%~2" -f bc3 -p opengl --mipmap --premultiply
echo 変換完了: %~1 → %~2
```

使い方:

```cmd
convert.bat my_icon.png my_icon.tex
```

---

## 参考リンク

- [Don't Starve Together MOD 公式フォーラム](https://forums.kleientertainment.com/forums/forum/79-dont-starve-together-mods-and-tools/)
- [DST Modding Wiki](https://dontstarve.wiki.gg/)

---

## 付録: 武器・アイテム MOD に必要な画像の全体像

武器や道具などのアイテム MOD を作る場合、**インベントリアイコンだけでは不十分** です。ゲーム内でアイテムが表示される場面ごとに、異なる種類の画像が必要になります。

### アイテムに必要な画像一覧

| 表示される場面 | 画像の種類 | 作り方 | 配置先 |
|--------------|-----------|--------|--------|
| 持ち物欄・クラフト画面 | インベントリアイコン | **PNG → TEX 変換**（本ガイドの方法） | `images/inventoryimages/` |
| 地面に落ちている時 | ゲーム内スプライト（ground anim） | **Spriter アニメーション** | `anim/` (.zip) |
| キャラクターが手に持っている時 | スワップスプライト（swap anim） | **Spriter アニメーション** | `anim/` (.zip) |

つまり、**本ガイドでカバーしているのはインベントリアイコン（1行目）の部分だけ** です。

### インベントリアイコン と ゲーム内スプライト の違い

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  【インベントリアイコン】           【ゲーム内スプライト】       │
│   本ガイドの範囲                    Spriter で作る              │
│                                                                 │
│   ┌──────┐                          ┌───────────┐              │
│   │ ICON │ ← 持ち物欄の             │ ＼(＾o＾)／│ ← 地面や   │
│   │64x64 │   小さい画像             │   剣を     │   キャラの  │
│   └──────┘                          │   持つ姿   │   手に表示  │
│                                     └───────────┘              │
│   PNG → TEX 変換                    Spriter (.scml)            │
│   images/inventoryimages/           → ビルド → anim/*.zip      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ゲーム内スプライト（Spriter アニメーション）について

地面に落ちた時の見た目や、キャラクターが装備している時の見た目は、**Spriter** というアニメーションツールで作ります。TextureConverter による TEX 変換とはまったく別の仕組みです。

Spriter アニメーションの作成方法については、別途 **[DST-Spriter ガイド](../DST-Spriter/)** で詳しく解説しています。そちらを参照してください。

---

## ライセンス

このドキュメントは自由に利用・改変・再配布できます。DST MOD 制作にお役立てください。

---

<sub>Author: [pinpikokun](https://steamcommunity.com/profiles/76561198076111536/)</sub><br>
<sub>[![GitHub](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/pinpikokun/DST-teemo) [![Steam Workshop](https://img.shields.io/badge/Steam%20Workshop-Subscribe-blue?logo=steam)](https://steamcommunity.com/sharedfiles/filedetails/?id=390684095)</sub>
