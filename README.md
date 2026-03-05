# DST MOD 開発ガイド — はじめてのキャラクター MOD を作ろう

Don't Starve Together (DST) の **キャラクター MOD** を作りたい人のための、初心者向け総合ガイドです。

このガイドでは、ウィルソン（ゲーム内の基本キャラクター）をベースに **見た目だけを変更したオリジナルキャラクター** を、コピペ＋キャラクター名の書き換えだけで作れるようになるまでを解説します。

> **HTML 版**: [README.html](https://pinpikokun.github.io/DST-Mod-Guide/README.html)

---

## 目次

1. [DST MOD とは](#dst-mod-とは)
2. [必要なツールのインストール](#必要なツールのインストール)
3. [MOD フォルダ構成の全体像](#mod-フォルダ構成の全体像)
4. [MOD 開発の全体フロー](#mod-開発の全体フロー)
5. [はじめての MOD — ウィルソンの見た目だけ変更する](#はじめての-mod--ウィルソンの見た目だけ変更する)
6. [各ツール詳細ガイド](#各ツール詳細ガイド)
7. [よくあるトラブルと解決法](#よくあるトラブルと解決法)
8. [Tips・参考情報](#tips参考情報)

---

## DST MOD とは

DST の MOD（モッド）とは、ゲームに **自分だけのキャラクター・アイテム・エフェクト** などを追加できる拡張機能です。MOD は **Lua** というプログラミング言語で書かれており、Steam Workshop を通じて他のプレイヤーと共有できます。

### このガイドで作れるもの

- ウィルソンと同じ動きをする、**見た目だけが違うオリジナルキャラクター**
- キャラクター選択画面に表示されるポートレート画像
- ゲーム内で動くアニメーション

### このガイドで扱わないもの（別途ガイドあり）

> **おすすめ**: プログラミング不要で MOD を作りたい方は **[DST-AI-Agent ガイド](DST-AI-Agent/README.md)** をご覧ください。AI エージェントに指示するだけでキャラクター MOD を作成できます。

| 内容 | 参照先ガイド |
|---|---|
| **AI エージェントを使った MOD 開発（おすすめ）** | **[DST-AI-Agent ガイド](DST-AI-Agent/README.md)** |
| PNG から TEX ファイルへの変換 | [DST-TextureConverter ガイド](DST-TextureConverter/README.md) |
| ゲームアセットのデコンパイル・TEX ↔ PNG 変換 | [DST-ktools ガイド](DST-ktools/README.md) |
| アニメーションの詳細 | [DST-Spriter ガイド](DST-Spriter/README.md) |
| 武器・アイテムの追加 | [武器・アイテムガイド](DST-Spriter/README-item-equipment.md) |
| エフェクトの作成 | [エフェクトガイド](DST-Spriter/README-effect.md) |
| サウンドの追加 | [DST-FMOD-Designer ガイド](DST-FMOD-Designer/README.md) |
| AI 音声合成によるボイス生成 | [DST-Chatterbox ガイド](DST-Chatterbox/README.md) |
| Lua スクリプティング（キャラクター固有能力・カスタムコンポーネント・レシピ追加など） | [DST-Lua-Scripting ガイド](DST-Lua-Scripting/README.md) |
| キャラクターのセリフ（Speech）カスタマイズ | [Speech ファイル作成ガイド](DST-Lua-Scripting/README-speech.md) |
| MOD のコンフィグオプション（設定画面） | [modinfo コンフィグガイド](DST-Lua-Scripting/README-modinfo-config.md) |

### 対象読者

- Windows 11 ユーザー
- DST をプレイしたことがある人
- MOD 作成は初めての人
- プログラミング経験は不要（コピペ＋キャラクター名の書き換えで OK）

---

## 必要なツールのインストール

### 1. Don't Starve Together（ゲーム本体）

Steam で購入済みであることを前提とします。

### 2. Don't Starve Mod Tools

MOD 開発に必要なツールがすべて入ったパッケージです。**無料** でインストールできます。

#### インストール手順

1. **Steam を開く**
2. 上部メニューの **「ライブラリ」** をクリック
3. 検索バーに `Don't Starve Mod Tools` と入力
4. **見つからない場合**: 検索バーの横にあるフィルターアイコン（漏斗マーク）をクリックして **「ツール」** にチェックを入れてください

   ```
   ライブラリ → 検索バー横のフィルター → ✅ ツール にチェック
   ```

5. **Don't Starve Mod Tools** をクリックし、**「インストール」** ボタンを押す
6. インストールが完了するまで待つ

#### インストール先の確認

デフォルトでは以下のパスにインストールされます。

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\
```

> **注意**: Steam のインストール先を変更している場合は、パスが異なります。Steam の「設定」→「ストレージ」で確認してください。

#### 含まれる主なツール

| ツール | 場所 | 用途 |
|---|---|---|
| **Spriter** | `mod_tools\Spriter\` | アニメーション作成・編集 |
| **FMOD Designer** | `mod_tools\FMOD_Designer\` | サウンド/ボイス作成 |
| **TextureConverter** | `mod_tools\tools\bin\TextureConverter.exe` | PNG → TEX 画像変換 |
| **autocompiler** | `mod_tools\autocompiler.exe` | アニメーション自動ビルド |
| **sample_build** | `mod_tools\sample_build\` | ウィルソンのパーツ画像テンプレート |

### 3. Git for Windows（Git Bash）

TextureConverter を使うために **Git Bash** が必要です。

> **なぜ Git Bash が必要？**: TextureConverter はコマンドラインツールです。Windows 標準のコマンドプロンプトでも動きますが、パスにスペースが含まれる場合の扱いが面倒なため、Git Bash を使うのが最も安全です。

#### インストール手順

1. 以下の URL にアクセスします。

   ```
   https://gitforwindows.org/
   ```

2. **「Download」** ボタンをクリックしてインストーラーをダウンロード
3. ダウンロードした `Git-*.*.*-64-bit.exe` を実行
4. インストール設定はすべて **デフォルトのまま「Next」** を押し続けて OK です

   > **補足**: 途中で「Adjusting your PATH environment」という画面が出たら、デフォルトの **「Git from the command line and also from 3rd-party software」** が選択されていることを確認してください。

5. **「Install」** をクリックしてインストール完了

#### インストールの確認

1. デスクトップまたはフォルダの空いている場所で **右クリック**
2. **「Open Git Bash here」**（または「Git Bash Here」）が表示されれば成功です

   > **Windows 11 の場合**: 右クリックメニューに直接表示されないことがあります。その場合は **「その他のオプションを確認」**（Show more options）をクリックすると表示されます。

### 4. テキストエディタ

Lua コードを編集するためのテキストエディタが必要です。**Visual Studio Code（VSCode）** を推奨します。

#### VSCode のインストール

1. 以下の URL にアクセスします。

   ```
   https://code.visualstudio.com/
   ```

2. **「Download for Windows」** をクリック
3. ダウンロードしたインストーラーを実行
4. デフォルト設定のままインストール

> **メモ帳でも OK？**: Windows 標準のメモ帳でも編集はできますが、Lua の色分け表示やエラー検出ができないため、VSCode を強く推奨します。

---

## MOD フォルダ構成の全体像

### 標準的なキャラクター MOD のフォルダ構成

```
my_character/                            ← MOD フォルダ（この名前は自由に決められる）
├── modinfo.lua                          ← MOD の情報（名前・説明・バージョンなど）
├── modmain.lua                          ← MOD のメインスクリプト
├── modicon.tex                          ← MOD 一覧に表示されるアイコン（TEX 形式）
├── modicon.xml                          ← アイコンのアトラスファイル
│
├── anim/                                ← アニメーション ZIP ファイル
│   ├── my_character.zip                 ← キャラクターのアニメーション
│   └── ghost_my_character_build.zip     ← ゴースト（死亡時）のアニメーション
│
├── bigportraits/                        ← 大きなポートレート画像
│   └── my_character.tex / .xml          ← キャラクター選択画面の大きな立ち絵
│
├── images/                              ← 各種画像ファイル
│   ├── avatars/                         ← チャット横のアバター画像
│   │   ├── avatar_my_character.tex / .xml
│   │   └── avatar_ghost_my_character.tex / .xml
│   ├── map_icons/                       ← ミニマップ上のアイコン
│   │   └── my_character.tex / .xml
│   ├── saveslot_portraits/              ← セーブスロットのポートレート
│   │   └── my_character.tex / .xml
│   └── selectscreen_portraits/          ← キャラクター選択画面のポートレート
│       └── my_character.tex / .xml
│
├── scripts/                             ← Lua スクリプト
│   ├── prefabs/                         ← エンティティ定義
│   │   └── my_character.lua             ← キャラクターの定義ファイル
│   └── speech_my_character.lua          ← キャラクターのセリフ
│
└── sound/                               ← サウンドファイル（オプション）
    ├── my_character.fev                 ← FMOD イベントファイル
    └── my_character_bank00.fsb          ← FMOD サウンドバンク
```

> **ポイント**: `my_character` の部分はすべて自分のキャラクター名に統一してください。大文字・小文字が混在するとエラーの原因になります。**すべて小文字** がおすすめです。

### DST が MOD を読み込む場所

DST はゲーム本体の `mods` フォルダから MOD を読み込みます。

**mods フォルダの開き方:**

1. **Steam** を開く
2. ライブラリから **Don't Starve Together** を右クリック
3. **管理 → ローカルファイルを閲覧** をクリック
4. 開いたフォルダ内の `mods` フォルダが MOD の配置先です

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\mods\
```

> **パスが違う場合**: Steam のインストール先を変更している場合はパスが異なります。上記の手順で Steam から開けば、正しい場所が必ず開きます。
>
> **日本語ユーザー名の注意**: Windows のユーザー名に日本語が含まれている場合、一部のツールが正しく動作しないことがあります。その場合は Steam のインストール先を日本語を含まないパス（例: `D:\Steam\`）に変更することを検討してください。

---

## MOD 開発の全体フロー

```
┌─────────────────────────────────────────────────────────┐
│                  MOD 開発の全体フロー                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. MOD フォルダを作成する                                  │
│       ↓                                                 │
│  2. modinfo.lua を作成する（MOD の情報を定義）               │
│       ↓                                                 │
│  3. キャラクターの画像を用意する（PNG）                       │
│       ↓                                                 │
│  4. Spriter でアニメーションを作る → ZIP にビルド             │
│       ↓                                                 │
│  5. ポートレート等の画像を TEX に変換する                     │
│       ↓                                                 │
│  6. modmain.lua・prefab ファイルを作成する（Lua）            │
│       ↓                                                 │
│  7. ゲーム内でテストする                                    │
│       ↓                                                 │
│  8. Steam Workshop に公開する（任意）                       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

> **ポートレートとは？**: キャラクター選択画面の立ち絵、セーブスロットのアイコン、チャットアイコンなど、ゲーム内の各所に表示されるキャラクターの画像をまとめて「ポートレート」と呼びます。

### 最低限必要なファイル

MOD として認識されるために最低限必要なファイルは以下の 2 つです。

| ファイル | 役割 |
|---|---|
| `modinfo.lua` | MOD の名前・説明・バージョン情報 |
| `modmain.lua` | MOD のメインスクリプト（何をするかを記述） |

これに加えて、キャラクター MOD では以下も必要です。

| ファイル | 役割 |
|---|---|
| `modicon.tex` + `modicon.xml` | MOD 一覧に表示されるアイコン |
| `anim/<キャラ名>.zip` | キャラクターのアニメーション |
| `scripts/prefabs/<キャラ名>.lua` | キャラクターの定義 |
| `scripts/speech_<キャラ名>.lua` | キャラクターのセリフ |
| 各種画像ファイル | ポートレート・アバター・マップアイコン |

---

## はじめての MOD — ウィルソンの見た目だけ変更する

ここからは、実際に手を動かして MOD を作ります。コードをコピペして、**★マークのコメントがある箇所を自分のキャラクター名に書き換える** だけで作成できます。

この章では、キャラクター名を `my_character` として進めます。自分のキャラクター名に置き換えてください。

> **命名ルール**:
> - **半角英数字とアンダースコア `_` のみ** 使用可能
> - **すべて小文字** を推奨（大文字小文字の不一致はクラッシュの原因）
> - スペース・日本語・ハイフン `-` は **使用不可**
> - 例: `my_character`, `fire_knight`, `sakura_witch`

---

### Step 1: MOD フォルダを作成する

MOD の開発用フォルダを作成します。

1. 以下のフォルダを開きます。

   ```
   C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\
   ```

   > **なぜここに作るの？**: この場所に MOD フォルダを置くと、`autocompiler.exe` がアニメーションを自動的にビルドしてくれるためです。開発が完了したら、ゲームの mods フォルダにコピーします。

2. このフォルダ内に `my_character` という名前のフォルダを作成します。

3. `my_character` フォルダの中に以下のフォルダを作成します。

   ```
   my_character/
   ├── anim/
   ├── bigportraits/
   ├── images/
   │   ├── avatars/
   │   ├── map_icons/
   │   ├── saveslot_portraits/
   │   └── selectscreen_portraits/
   ├── scripts/
   │   └── prefabs/
   └── sound/
   ```

   > **フォルダ作成のコツ**: エクスプローラーで `my_character` フォルダを開き、空いている場所で右クリック → 「新規作成」→「フォルダー」で作成できます。`images` の中にさらにサブフォルダを作ることを忘れないでください。

---

### Step 2: modinfo.lua を作成する

MOD の基本情報を定義するファイルです。以下のコードをコピペして、`my_character` の部分を **自分のキャラクター名に書き換えて** から `modinfo.lua` として保存してください。

> **ファイルの作り方（VSCode の場合）**:
> 1. VSCode を開き、メニューの **「ファイル」→「フォルダーを開く」** で `my_character` フォルダを開く
> 2. 左側のエクスプローラーパネルで **`my_character` フォルダ名を右クリック →「新しいファイル」**
> 3. ファイル名に `modinfo.lua` と入力して Enter
> 4. 開いたエディタに下記のコードをコピペして **Ctrl+S** で保存
>
> 以降の Step 3〜5 の `.lua` ファイルもすべてこの方法で作成できます。

```lua
-- MOD の基本情報
name = "My Character"                    -- ★書き換え: MOD の名前（ゲーム内で表示される）
description = "My first DST character!"  -- ★書き換え: MOD の説明文
author = "Your Name"                     -- ★書き換え: 作者名
version = "1.0.0"                        -- バージョン番号（そのままでOK）

-- この行はそのまま残してください（DST の API バージョン）
api_version = 10

-- DST 専用 MOD（そのまま）
dont_starve_compatible = false
reign_of_giants_compatible = false
dst_compatible = true

-- MOD アイコン（そのまま）
icon_atlas = "modicon.xml"
icon = "modicon.tex"

-- マルチプレイ設定（そのまま）
all_clients_require_mod = true           -- 全員がこの MOD を持っている必要がある
clients_only_mod = false                 -- サーバー MOD（false のままにする）

-- MOD の設定オプション（そのまま）
configuration_options = {}
```

> **各行の意味**:
> - `name`: Steam Workshop やゲーム内 MOD 一覧に表示される名前です
> - `api_version = 10`: DST の現在の API バージョンです。**変更しないでください**
> - `all_clients_require_mod = true`: マルチプレイで遊ぶとき、全員がこの MOD を入れている必要があるという設定です
> - `configuration_options`: ゲーム内で設定を変更できるオプションです。今回は使わないので空にします。詳しくは [modinfo コンフィグガイド](DST-Lua-Scripting/README-modinfo-config.md) を参照

---

### Step 3: modmain.lua を作成する

MOD のメインスクリプトです。以下のコードをコピペして、`my_character` の部分を **自分のキャラクター名に書き換えて** から `modmain.lua` として保存してください。

```lua
-- ========================================
-- アセット（画像・アニメーション）の登録
-- ========================================
-- ここに記載したファイルがゲーム起動時に読み込まれます。
-- ファイルが存在しないとクラッシュするので注意してください。
-- ★以下すべての "my_character" を自分のキャラクター名に書き換えてください

Assets = {
    -- キャラクターアニメーション
    Asset("ANIM", "anim/my_character.zip"),                                  -- ★my_character を書き換え
    Asset("ANIM", "anim/ghost_my_character_build.zip"),                      -- ★my_character を書き換え

    -- キャラクター選択画面の画像
    Asset("IMAGE", "images/saveslot_portraits/my_character.tex"),            -- ★my_character を書き換え
    Asset("ATLAS", "images/saveslot_portraits/my_character.xml"),            -- ★my_character を書き換え
    Asset("IMAGE", "images/selectscreen_portraits/my_character.tex"),        -- ★my_character を書き換え
    Asset("ATLAS", "images/selectscreen_portraits/my_character.xml"),        -- ★my_character を書き換え
    Asset("IMAGE", "bigportraits/my_character.tex"),                         -- ★my_character を書き換え
    Asset("ATLAS", "bigportraits/my_character.xml"),                         -- ★my_character を書き換え

    -- アバター（チャット横のアイコン）
    Asset("IMAGE", "images/avatars/avatar_my_character.tex"),               -- ★my_character を書き換え
    Asset("ATLAS", "images/avatars/avatar_my_character.xml"),               -- ★my_character を書き換え
    Asset("IMAGE", "images/avatars/avatar_ghost_my_character.tex"),         -- ★my_character を書き換え
    Asset("ATLAS", "images/avatars/avatar_ghost_my_character.xml"),         -- ★my_character を書き換え

    -- マップアイコン
    Asset("IMAGE", "images/map_icons/my_character.tex"),                    -- ★my_character を書き換え
    Asset("ATLAS", "images/map_icons/my_character.xml"),                    -- ★my_character を書き換え
}

-- ========================================
-- キャラクター情報の設定
-- ========================================
local STRINGS = GLOBAL.STRINGS

-- キャラクター選択画面に表示される情報
STRINGS.CHARACTER_TITLES.my_character = "The Custom Character"                             -- ★my_character を書き換え、★"The Custom Character" を書き換え（称号）
STRINGS.CHARACTER_NAMES.my_character = "My Character"                                      -- ★my_character を書き換え、★"My Character" を書き換え（表示名）
STRINGS.CHARACTER_DESCRIPTIONS.my_character = "*Has custom appearance\n*Based on Wilson"   -- ★my_character を書き換え、★説明文を書き換え（各行 * で始める）
STRINGS.CHARACTER_QUOTES.my_character = "\"Hello, world!\""                                -- ★my_character を書き換え、★セリフを書き換え
STRINGS.CHARACTER_ABOUTME.my_character = "A custom character created for learning."        -- ★my_character を書き換え、★自己紹介を書き換え

-- セリフデータの読み込み
STRINGS.CHARACTERS.MY_CHARACTER = require "speech_my_character"  -- ★MY_CHARACTER を大文字の自分のキャラクター名に書き換え、★speech_my_character を書き換え

-- キャラクター名（ゲーム内で他プレイヤーから見た名前）
STRINGS.NAMES.MY_CHARACTER = "My Character"                      -- ★MY_CHARACTER を大文字の自分のキャラクター名に書き換え、★"My Character" を書き換え

-- スキン名（スキン未対応でも必要）
STRINGS.SKIN_NAMES.my_character_none = "My Character"            -- ★my_character を書き換え、★"My Character" を書き換え

-- ========================================
-- キャラクターを登録する
-- ========================================
AddModCharacter("my_character", "MALE")  -- ★"my_character" を書き換え、★"MALE" を性別に合わせて "MALE" または "FEMALE" に変更
```

> **重要**: `MY_CHARACTER`（大文字）と `my_character`（小文字）の使い分けに注意してください。
> - **小文字** (`my_character`): ファイル名、prefab 名、フォルダ名
> - **大文字** (`MY_CHARACTER`): STRINGS テーブルのキー、キャラクター内部定数
>
> この対応を間違えるとゲームがクラッシュします。

---

### Step 4: キャラクター prefab ファイルを作成する

> **prefab（プレハブ）とは？**: DST の世界に存在するすべてのもの（キャラクター、アイテム、モンスター、木、石など）は「prefab」として定義されています。prefab は「設計図」のようなもので、見た目（アニメーション）、能力（コンポーネント）、ステータス（体力・攻撃力など）をひとまとめにした定義です。ゲームはこの設計図をもとに、実際のオブジェクトを世界に配置します。

キャラクターの定義ファイルです。以下のコードをコピペして、`my_character` の部分を **自分のキャラクター名に書き換えて** から `scripts/prefabs/my_character.lua` として保存してください（ファイル名も書き換え）。

```lua
local MakePlayerCharacter = require("prefabs/player_common")  -- そのまま

-- ========================================
-- アセットの定義
-- ========================================
local assets = {
    Asset("ANIM", "anim/my_character.zip"),                -- ★my_character を書き換え
    Asset("ANIM", "anim/ghost_my_character_build.zip"),    -- ★my_character を書き換え
}

-- 初期所持アイテム（空 = デフォルトのアイテム）
local start_inv = {}  -- ★変更任意: アイテムを追加できる（例: {"flint", "twigs"}）

-- ========================================
-- common_postinit: サーバー・クライアント両方で実行される処理
-- ========================================
local common_postinit = function(inst)
    -- ミニマップアイコンの設定
    inst.MiniMapEntity:SetIcon("my_character.tex")  -- ★my_character を書き換え
end

-- ========================================
-- master_postinit: サーバーのみで実行される処理
-- ========================================
local master_postinit = function(inst)
    -- キャラクターの基本ステータス
    inst.components.health:SetMaxHealth(150)     -- ★変更任意: 体力（ウィルソン = 150）
    inst.components.hunger:SetMax(150)           -- ★変更任意: 空腹（ウィルソン = 150）
    inst.components.sanity:SetMax(200)           -- ★変更任意: 正気度（ウィルソン = 200）

    -- 移動速度
    inst.components.locomotor.walkspeed = 4      -- ★変更任意: 歩行速度（デフォルト: 4）
    inst.components.locomotor.runspeed = 6       -- ★変更任意: 走行速度（デフォルト: 6）

    -- 攻撃力倍率
    inst.components.combat.damagemultiplier = 1  -- ★変更任意: 攻撃力倍率（1.0 = デフォルト）

    -- ボイスの設定
    inst.soundsname = "wilson"                   -- ★変更任意: "willow", "wendy" など既存キャラ名に変更可
end

-- ========================================
-- キャラクターを生成して返す
-- ========================================
return MakePlayerCharacter("my_character", prefabs, assets, common_postinit, master_postinit, start_inv)  -- ★"my_character" を書き換え
```

> **カスタマイズのヒント**:
> - `SetMaxHealth(150)` の数値を変えると体力が変わります（ウィルソン = 150、ウィグフリッド = 200）
> - `inst.soundsname = "wilson"` を `"willow"` や `"wendy"` に変えると、既存キャラクターのボイスを使えます
> - `start_inv` にアイテム名を追加すると、ゲーム開始時にそのアイテムを持った状態で始まります
>   - 例: `local start_inv = {"flint", "flint", "twigs"}` → フリント2個と小枝を持ってスタート

---

### Step 5: セリフファイルを作成する

キャラクターがアイテムを調べたときなどに表示されるセリフです。`my_character/scripts/speech_my_character.lua` として保存してください。

```lua
-- ウィルソンのセリフをベースにした最小セリフファイル
-- 詳細なセリフを追加したい場合は、ウィルソンの speech ファイルをコピーして
-- 各セリフを自分のキャラクターに合った内容に書き換えてください。
--
-- ウィルソンの speech ファイルの場所:
-- C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\data\databundles\scripts.zip
-- ↑この ZIP を展開すると scripts\speech_wilson.lua が見つかります

return require("speech_wilson")
```

> **解説**: `require("speech_wilson")` はウィルソンのセリフデータをそのまま使うという意味です。これにより、すべてのセリフがウィルソンと同じ内容で表示されます。オリジナルのセリフに変更したい場合は、ウィルソンの speech ファイルの内容をコピーして、各行を書き換えてください。
>
> **ウィルソンの speech ファイルの場所**:
> DST のスクリプトは ZIP にバンドルされています。中身を確認するには、以下の ZIP を展開してください。
> ```
> C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\data\databundles\scripts.zip
> ```
> 展開すると `scripts\speech_wilson.lua` が見つかります。
> このファイルは非常に長い（数千行）ですが、構造はシンプルで、各アイテムやイベントに対する一言セリフが並んでいるだけです。
>
> セリフのカスタマイズ方法について詳しくは [Speech ファイル作成ガイド](DST-Lua-Scripting/README-speech.md) を参照してください。

---

### Step 6: キャラクター画像を用意する（sample_build）

ここがキャラクター MOD の **一番楽しい部分** です。ウィルソンのパーツ画像を自分のキャラクターに差し替えます。

#### sample_build とは

DST Mod Tools には、ウィルソンの身体パーツが分かれた PNG 画像のテンプレートが含まれています。

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\sample_build\
```

このフォルダの中身は以下の通りです。

| フォルダ | 中身 | 説明 |
|---|---|---|
| `arm_lower/` | arm_lower-0.png 〜 2.png | 前腕（3 パターン） |
| `arm_upper/` | arm_upper-0.png 〜 4.png | 上腕（5 パターン） |
| `cheeks/` | cheeks-0.png 〜 1.png | 頬（食べるアニメーション用） |
| `face/` | face-0.png 〜 13.png | 顔の表情（14 パターン） |
| `foot/` | foot-0.png 〜 6.png | 足（7 パターン） |
| `hair/` | hair-0.png 〜 2.png | 髪（3 パターン） |
| `hair_hat/` | hair_hat-0.png 〜 2.png | 帽子を被ったときの髪 |
| `hand/` | hand-0.png 〜 10.png | 手（11 パターン） |
| `headbase/` | headbase-0.png 〜 2.png | 頭の輪郭（3 パターン） |
| `headbase_hat/` | headbase_hat-0.png 〜 2.png | 帽子を被ったときの頭 |
| `leg/` | leg-0.png 〜 8.png | 脚（9 パターン） |
| `torso/` | torso-0.png 〜 9.png | 胴体（10 パターン） |
| `build.scml` | ― | Spriter プロジェクトファイル |

> **各パーツに番号が振られている理由**: 同じパーツでも、正面・横向き・後ろ向きなどでパターンが違います。たとえば `face-0.png` は正面の普通の表情、`face-4.png` は横向きの表情です。

#### 画像を差し替える手順

1. `sample_build` フォルダを **丸ごとコピー** して、直接 `exported` フォルダ内に配置します（Step 7 で autocompiler がここを参照するため）。

   コピー元:
   ```
   C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\sample_build\
   ```

   コピー先（`exported` フォルダが無い場合は作成してください）:
   ```
   C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\exported\my_character_build\
   ```

   > **フォルダ名が重要**: このフォルダの名前が **Build 名**（見た目の名前）になります。`my_character_build` としてください。

2. コピーしたフォルダ内の **`build.scml`** を Spriter で開いて、パーツ画像の配置を確認します。

   Spriter の場所:
   ```
   C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\Spriter\Spriter.exe
   ```

   #### Spriter の基本操作

   1. **Spriter.exe をダブルクリック** して起動します
   2. メニューバーの **「File」→「Open Project」** をクリックします
   3. ファイル選択ダイアログが開くので、以下のファイルを選択して **「開く」** をクリックします
      ```
      C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\exported\my_character_build\build.scml
      ```
   4. プロジェクトが読み込まれると、画面中央にウィルソンのキャラクターが表示されます

   ```
   ┌──────────────────────────────────────────────────────────┐
   │  File  Edit  View  Help                                  │
   ├────────────┬─────────────────────────┬───────────────────┤
   │            │                         │                   │
   │  ファイル   │    キャラクター表示      │   パレット        │
   │  ツリー     │    （プレビュー）        │  （パーツ一覧）    │
   │            │                         │                   │
   ├────────────┴─────────────────────────┴───────────────────┤
   │  ▶ ■ ◀◀ ▶▶   タイムライン（アニメーション再生バー）       │
   └──────────────────────────────────────────────────────────┘
   ```

   - **左側（ファイルツリー）**: フォルダとパーツ画像の一覧が表示されます
   - **中央（プレビュー）**: キャラクターの見た目が表示されます。画像を差し替えた後にここで確認できます
   - **下部（タイムライン）**: **▶ ボタン** を押すとアニメーションが再生されます。歩く・走る・攻撃するなどの動きを確認できます
   - **アニメーションの切り替え**: タイムライン上部のドロップダウンリストからアニメーション名（`idle`、`run`、`attack` など）を選ぶと、別のアニメーションを確認できます

   > **Spriter でやること**: このガイドでは Spriter は **プレビュー確認** のためだけに使います。画像の差し替えはエクスプローラーで PNG を上書きするだけで OK です。

3. 以下のフォルダ内の PNG を、**同じファイル名・同じサイズ** で自分の画像に差し替えます。

   ```
   C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\exported\my_character_build\
   ├── arm_lower\    ← 前腕の画像
   ├── arm_upper\    ← 上腕の画像
   ├── cheeks\       ← 頬の画像
   ├── face\         ← 顔の表情（★まずはここだけ差し替えるのがおすすめ）
   ├── foot\         ← 足の画像
   ├── hair\         ← 髪の画像
   ├── hair_hat\     ← 帽子を被ったときの髪
   ├── hand\         ← 手の画像
   ├── headbase\     ← 頭の輪郭
   ├── headbase_hat\ ← 帽子を被ったときの頭
   ├── leg\          ← 脚の画像
   └── torso\        ← 胴体の画像
   ```

   これらのフォルダ内にある PNG ファイルを、自分のキャラクターの画像で上書きしてください。

   > **差し替えのルール**:
   > - ファイル名は **絶対に変えない**（`face-0.png` は `face-0.png` のまま）
   > - 画像サイズは **元の画像と同じサイズにする**（ピクセル数を揃える）
   > - **透過 PNG** で保存する（背景は透明に）
   > - 色モードは **RGBA 32bit**

4. 差し替えが終わったら、`build.scml` を Spriter で開いて、見た目が正しいか確認します。

> **初心者向けアドバイス**: 最初は全パーツを差し替える必要はありません。まずは `face/` フォルダの画像だけ差し替えて、顔だけ変えたキャラクターから始めるのがおすすめです。

---

### Step 7: アニメーションをビルドする

差し替えた画像を、DST が読み込める ZIP 形式に変換します。

#### 方法 A: autocompiler を使う（推奨）

autocompiler は、フォルダの変更を監視して自動的にアニメーションをビルドしてくれるツールです。

1. Step 6 で `exported/my_character_build/` に画像を配置済みであることを確認します。

2. `autocompiler.exe` を実行します。

   ```
   C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\autocompiler.exe
   ```

   > コマンドプロンプトの黒い画面が表示されます。ビルド処理のログが流れた後、**ウィンドウが自動的に閉じるか、「Press any key to continue...」** のようなメッセージが表示されたら完了です。エラーが出ていないか、閉じる前にログの最後を確認してください。

3. ビルドが成功すると、以下の場所に ZIP ファイルが生成されます。

   ```
   C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\anim\my_character_build.zip
   ```

4. この ZIP ファイルを MOD フォルダの `anim/` にコピーし、**ファイル名を変更** します。

   ```
   コピー先: my_character\anim\my_character.zip
   ```

   > **ファイル名の注意**: ビルドされた ZIP のファイル名（拡張子を除いた部分）が **Build 名** になります。prefab ファイルで `AnimState:SetBuild("my_character")` と指定する場合、ZIP のファイル名は `my_character.zip` である必要があります。

#### ゴーストアニメーション

キャラクターが死亡するとゴースト（幽霊）になります。ゴースト用のアニメーションも必要です。

一番簡単な方法は、DST の既存ゴーストアニメーションを使うことです。

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\data\anim\ghost_wilson_build.zip
```

このファイルを以下の名前でコピーしてください。

```
コピー先: my_character\anim\ghost_my_character_build.zip
```

> **注意**: ゴースト用アニメーションの Build 名は `ghost_<キャラ名>_build` という命名規則に従ってください。

---

### Step 8: ポートレート画像を TEX に変換する

DST では PNG 画像をそのまま使えません。**TEX 形式** に変換する必要があります。

#### 必要な画像一覧

| 画像 | サイズ | 配置先フォルダ | 説明 |
|---|---|---|---|
| modicon | 128 × 128 | MOD フォルダ直下 | MOD 一覧に表示されるアイコン |
| selectscreen portrait | 256 × 512 | `images/selectscreen_portraits/` | キャラクター選択画面の一覧に表示される立ち絵 |
| saveslot portrait | 128 × 128 | `images/saveslot_portraits/` | セーブデータのスロットに表示されるアイコン |
| bigportrait | 512 × 512 以上 | `bigportraits/` | キャラクター選択画面でキャラを選んだときに右側に大きく表示される立ち絵 |
| avatar | 64 × 64 | `images/avatars/` | チャット送信時にメッセージ横に表示されるアイコン |
| avatar (ghost) | 64 × 64 | `images/avatars/` | 死亡してゴースト状態のときのチャットアイコン |
| map icon | 64 × 64 | `images/map_icons/` | ミニマップ上でキャラの位置を示すアイコン |

> **画像サイズの重要ルール**: すべての画像サイズは **各辺が 2 の累乗**（64, 128, 256, 512...）でなければなりません。それ以外のサイズ（例: 100 × 100）を使うと **ゲームがクラッシュ** します。正方形でなくても構いませんが、各辺はそれぞれ 2 の累乗にしてください（例: 256 × 512 は OK）。

#### PNG の準備

1. 上の表に合わせて PNG 画像を作成します。
2. すべて **透過 PNG（RGBA 32bit）** で保存します。
3. 画像サイズは **必ず各辺が 2 の累乗** にしてください。

> **画像作成ソフト**: 以下のいずれかを使って PNG を作成できます。
> - **ペイント（Windows 標準）**: スタートメニューで「ペイント」と検索。透過 PNG は保存時に「PNG」形式を選択すれば OK です。サイズ変更は「サイズ変更」→ ピクセルを指定。
> - **GIMP（無料）**: 高機能な画像編集ソフト。https://www.gimp.org/ からダウンロード。透過やレイヤーに対応。
> - **Photoshop / Clip Studio Paint**: 有料ですが使い慣れていれば最適です。
>
> **画像サイズの確認方法**: エクスプローラーで PNG ファイルを右クリック →「プロパティ」→「詳細」タブ → 幅・高さが表示されます。

> **まずは仮画像で OK**: 最初は適当な色の四角形でも構いません。まずは MOD が動くことを確認してから、本番の画像に差し替えましょう。

#### TEX への変換手順

Git Bash を開いて、以下のコマンドを実行します。

1. デスクトップなどに作業用フォルダを作り、そこに PNG を置きます。

2. Git Bash を開きます（フォルダ内で右クリック →「Open Git Bash here」）。

3. 以下のコマンドで PNG を TEX に変換します。**すべての画像で同じコマンド形式** です（ファイル名だけ変えてください）。

```bash
"/c/Program Files (x86)/Steam/steamapps/common/Don't Starve Mod Tools/mod_tools/tools/bin/TextureConverter.exe" -i ファイル名.png -o ファイル名.tex -f bc3 -p opengl --mipmap --premultiply
```

変換する画像ごとにファイル名を変えて実行します（例: `modicon.png → modicon.tex`、`my_character.png → my_character.tex`）。

> **コマンドの解説**:
> - `-i`: 入力ファイル（PNG）
> - `-o`: 出力ファイル（TEX）
> - `-f bc3 -p opengl --mipmap --premultiply`: **おまじないとして常にこの通り入力してください**。意味が分からなくても大丈夫です。省略するとゲームがクラッシュしたり表示がおかしくなります。
>
> 詳細が気になる方は [DST-TextureConverter ガイド](DST-TextureConverter/README.md) を参照してください。

すべての PNG を TEX に変換してください。同じ名前の画像はすべて同じコマンドで変換できます。

#### XML アトラスファイルの作成

> **XML アトラスファイルとは？**: TEX ファイル（画像データ）だけでは、ゲームはその画像をどう読み込めばいいかわかりません。XML アトラスファイルは「この TEX ファイルの中に、どんな画像が、どの位置にあるか」を記述した設定ファイルです。TEX と XML は常にセットで必要になります。

TEX ファイルには対応する XML ファイルが必要です。合計 **7 個** の XML ファイルを作成します。

必要な XML ファイルの一覧（`my_character` の部分は自分のキャラクター名に置き換えてください）:

| # | ファイル名 | 中に書く TEX ファイル名 | 配置先フォルダ |
|---|---|---|---|
| 1 | `modicon.xml` | `modicon.tex` | MOD フォルダ直下 |
| 2 | `my_character.xml` | `my_character.tex` | `bigportraits/` |
| 3 | `avatar_my_character.xml` | `avatar_my_character.tex` | `images/avatars/` |
| 4 | `avatar_ghost_my_character.xml` | `avatar_ghost_my_character.tex` | `images/avatars/` |
| 5 | `my_character.xml` | `my_character.tex` | `images/map_icons/` |
| 6 | `my_character.xml` | `my_character.tex` | `images/saveslot_portraits/` |
| 7 | `my_character.xml` | `my_character.tex` | `images/selectscreen_portraits/` |

#### XML ファイルの作り方

1. Windows の **メモ帳** を開きます（スタートメニューで「メモ帳」と検索）
2. 以下のテンプレートをコピペします:

```xml
<Atlas><Texture filename="★ここにTEXファイル名★" /><Elements><Element name="★ここにTEXファイル名★" u1="0" u2="1" v1="0" v2="1" /></Elements></Atlas>
```

3. `★ここにTEXファイル名★` の 2 箇所を、上の表の「中に書く TEX ファイル名」に書き換えます
4. **ファイル → 名前を付けて保存** で、上の表の「ファイル名」で保存します

> **保存時の注意**: メモ帳の「名前を付けて保存」画面で、ファイルの種類を **「すべてのファイル (\*.\*)」** に変更してから保存してください。そうしないと `modicon.xml.txt` のように余計な `.txt` が付いてしまいます。

**例: modicon.xml を作る場合:**

1. メモ帳を開く
2. テンプレートをコピペ
3. `★ここにTEXファイル名★` を `modicon.tex` に書き換える（2 箇所とも）
4. 「名前を付けて保存」→ ファイルの種類を「すべてのファイル」→ ファイル名に `modicon.xml` と入力して保存

完成した modicon.xml の中身:
```xml
<Atlas><Texture filename="modicon.tex" /><Elements><Element name="modicon.tex" u1="0" u2="1" v1="0" v2="1" /></Elements></Atlas>
```

この手順を 7 回繰り返して、上の表のすべての XML ファイルを作成してください。

#### 各画像の配置チェックリスト

変換した TEX と作成した XML を、MOD フォルダの正しい場所に配置してください。

```
my_character/
├── modicon.tex                                          ✅
├── modicon.xml                                          ✅
├── bigportraits/
│   └── my_character.tex + .xml                          ✅
├── images/
│   ├── avatars/
│   │   ├── avatar_my_character.tex + .xml               ✅
│   │   └── avatar_ghost_my_character.tex + .xml         ✅
│   ├── map_icons/
│   │   └── my_character.tex + .xml                      ✅
│   ├── saveslot_portraits/
│   │   └── my_character.tex + .xml                      ✅
│   └── selectscreen_portraits/
│       └── my_character.tex + .xml                      ✅
```

---

### Step 9: ファイルの最終確認

すべてのファイルが揃っているか確認しましょう。

```
my_character/
├── modinfo.lua                          ← Step 2 で作成
├── modmain.lua                          ← Step 3 で作成
├── modicon.tex                          ← Step 8 で変換
├── modicon.xml                          ← Step 8 で作成
│
├── anim/
│   ├── my_character.zip                 ← Step 7 でビルド
│   └── ghost_my_character_build.zip     ← Step 7 でコピー
│
├── bigportraits/
│   ├── my_character.tex                 ← Step 8 で変換
│   └── my_character.xml                 ← Step 8 で作成
│
├── images/
│   ├── avatars/
│   │   ├── avatar_my_character.tex      ← Step 8 で変換
│   │   ├── avatar_my_character.xml
│   │   ├── avatar_ghost_my_character.tex
│   │   └── avatar_ghost_my_character.xml
│   ├── map_icons/
│   │   ├── my_character.tex
│   │   └── my_character.xml
│   ├── saveslot_portraits/
│   │   ├── my_character.tex
│   │   └── my_character.xml
│   └── selectscreen_portraits/
│       ├── my_character.tex
│       └── my_character.xml
│
├── scripts/
│   ├── prefabs/
│   │   └── my_character.lua             ← Step 4 で作成
│   └── speech_my_character.lua          ← Step 5 で作成
│
└── sound/                               ← 今回は空でOK
```

> **ファイルが 1 つでも足りないとクラッシュします**。modmain.lua の `Assets` テーブルに記載したファイルがすべて存在するか、もう一度確認してください。

---

### Step 10: ゲーム内でテストする

#### MOD フォルダをゲームに配置する

1. 完成した `my_character` フォルダを DST の `mods` フォルダにコピーします。

   **mods フォルダの開き方:** Steam → Don't Starve Together を右クリック → **管理 → ローカルファイルを閲覧** → `mods` フォルダ

   ```
   C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\mods\
   ```

   > **mods フォルダが無い場合**: 一度ゲームを起動して MOD 設定画面を開くと、自動的に作成されます。

2. `mods` フォルダの中にある `dedicated_server_mods_setup.lua` は **触らないでください**。

#### MOD を有効にしてテストする

1. **DST を起動** する
2. メインメニューから **「ゲームをホスト」** を選択
3. **「新しい世界を作成する」** をクリック
4. 上部の **「MOD」** タブを開く
5. 自分の作った MOD（「My Character」）が表示されるので、**チェックボックスをチェック** して有効にする
6. **「ワールドを作成」** をクリック
7. キャラクター選択画面で **自分の MOD キャラクター** を選択する
8. ゲームが開始され、**キャラクターが正常に動けることを確認** できれば成功！

#### もしエラーが出たら

ログファイルを確認しましょう。

**ログファイルの場所:**

```
C:\Users\<ユーザー名>\Documents\Klei\DoNotStarveTogether\client_log.txt
```

> **見つけ方**: エクスプローラーのアドレスバーに `%USERPROFILE%\Documents\Klei\DoNotStarveTogether` と入力すると直接開けます。フォルダ内の `client_log.txt` が最新のログです。

このファイルをメモ帳で開き、**末尾付近** にエラーメッセージが出ていないか確認してください。

> **よくあるエラーの例**:
> ```
> PANIC: Error loading module 'speech_my_character'
> ```
> → `scripts/speech_my_character.lua` が見つからないか、ファイル名が間違っている
>
> ```
> Could not find anim [anim/my_character.zip]
> ```
> → `anim/my_character.zip` が存在しない、またはビルドに失敗している

---

## 各ツール詳細ガイド

このリポジトリには、MOD 開発の各工程に特化した詳細ガイドが含まれています。

> **おすすめ**: AI エージェントを使えば、プログラミング不要で MOD を作成できます。まずは **[DST-AI-Agent ガイド](DST-AI-Agent/README.md)** から始めるのがおすすめです。Claude Code / GitHub Copilot / Google Antigravity に対応しています。

| ガイド | 内容 | リンク |
|---|---|---|
| **DST-AI-Agent** | **AI エージェントを使った MOD 開発（おすすめ）** | [DST-AI-Agent/README.md](DST-AI-Agent/README.md) |
| **DST-Spriter** | アニメーション作成（キャラ・武器・エフェクト） | [DST-Spriter/README.md](DST-Spriter/README.md) |
| **DST-TextureConverter** | PNG → TEX 画像変換の詳細手順 | [DST-TextureConverter/README.md](DST-TextureConverter/README.md) |
| **DST-ktools** | ゲームアセットのデコンパイル・TEX ↔ PNG 変換 | [DST-ktools/README.md](DST-ktools/README.md) |
| **DST-FMOD-Designer** | キャラクターボイス・サウンド作成 | [DST-FMOD-Designer/README.md](DST-FMOD-Designer/README.md) |
| **DST-Chatterbox** | AI 音声合成によるボイス生成 | [DST-Chatterbox/README.md](DST-Chatterbox/README.md) |
| **Speech ファイル** | キャラクターのセリフカスタマイズ | [DST-Lua-Scripting/README-speech.md](DST-Lua-Scripting/README-speech.md) |
| **modinfo コンフィグ** | MOD 設定画面のオプション作成 | [DST-Lua-Scripting/README-modinfo-config.md](DST-Lua-Scripting/README-modinfo-config.md) |

### おすすめの学習順序

1. [DST-AI-Agent ガイド](DST-AI-Agent/README.md) で **AI エージェントを使って MOD を作成する**（おすすめ）
2. まずこのガイドで **見た目だけ変更したキャラクター** を手動で作る
3. [DST-Spriter ガイド](DST-Spriter/README.md) で **オリジナルアニメーション** の作り方を学ぶ
4. [DST-TextureConverter ガイド](DST-TextureConverter/README.md) で **画像変換の詳細** を理解する
5. [DST-ktools ガイド](DST-ktools/README.md) で **既存アセットのデコンパイル** を学ぶ
6. [Speech ファイル作成ガイド](DST-Lua-Scripting/README-speech.md) で **キャラクターのセリフ** をカスタマイズする
7. [modinfo コンフィグガイド](DST-Lua-Scripting/README-modinfo-config.md) で **設定オプション** を追加する
8. [DST-FMOD-Designer ガイド](DST-FMOD-Designer/README.md) で **キャラクターボイス** を追加する
9. [DST-Chatterbox ガイド](DST-Chatterbox/README.md) で **AI 音声** を作成する

---

## よくあるトラブルと解決法

### ゲームが起動しない / クラッシュする

| 症状 | 原因 | 解決法 |
|---|---|---|
| MOD を有効にするとクラッシュ | `Assets` に登録したファイルが存在しない | 全ファイルの存在を確認する |
| キャラ選択でクラッシュ | 画像サイズが 2 の累乗でない | 画像を 64, 128, 256, 512 のいずれかにリサイズ |
| ゲーム開始時にクラッシュ | prefab ファイルの文法エラー | `client_log.txt` を確認して該当行を修正 |

### MOD が一覧に表示されない

- `modinfo.lua` が **MOD フォルダの直下** にあるか確認
- `modinfo.lua` に文法エラーがないか確認（カンマの抜け、引用符の閉じ忘れなど）
- `dst_compatible = true` が設定されているか確認

### キャラクターが透明 / 真っ黒になる

- `anim/my_character.zip` が正しくビルドされているか確認
- prefab ファイルの `AnimState:SetBuild()` と ZIP のファイル名（拡張子なし）が一致しているか確認
- `AnimState:SetBank("wilson")` が正しく設定されているか確認（Bank は wilson のまま）

### autocompiler が動かない

- `exported` フォルダに Spriter プロジェクト（`.scml` + PNG フォルダ）を配置しているか確認
- `autocompiler.exe` を **管理者権限** で実行してみる
- パスに日本語が含まれていないか確認

### パスに日本語が含まれている

Windows のユーザー名が日本語の場合、一部のツールが正常に動作しないことがあります。

**対処法**:
1. Steam のインストール先を `D:\Steam\` など、日本語を含まないパスに変更する
2. または、ゲームのインストール先だけを移動する（Steam → ゲームを右クリック → プロパティ → インストールフォルダの移動）

---

## Tips・参考情報

### ログファイルの場所

DST のログファイルはエラーの原因を特定するのに非常に役立ちます。

| ログ | パス |
|---|---|
| クライアントログ | `Documents\Klei\DoNotStarveTogether\<クラスタID>\client_log.txt` |
| サーバーログ | `Documents\Klei\DoNotStarveTogether\<クラスタID>\Master\server_log.txt` |

> **ヒント**: ゲームを起動するたびにログは上書きされます。エラーが出たらすぐに確認しましょう。

### Steam Workshop への公開

MOD が完成したら、Steam Workshop に公開して他のプレイヤーに遊んでもらえます。公開前の準備からアップロード手順、更新方法まで、詳しくは専用ガイドを参照してください。

→ [DST-Workshop-Upload ガイド — Steam ワークショップへの公開マニュアル](DST-Workshop-Upload/README.md)

### DST のゲームファイルを参照する

既存キャラクターのコードを参考にしたい場合、ゲームのファイルを確認できます。

DST のスクリプトは以下の ZIP にバンドルされています。

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\data\databundles\scripts.zip
```

この ZIP を展開すると以下のような構成が得られます。

```
scripts\
├── prefabs\           ← キャラクター・アイテムの定義
├── components\        ← コンポーネント（機能の部品）
├── speech_wilson.lua  ← ウィルソンのセリフ
└── speech_willow.lua  ← ウィロウのセリフ（他キャラも同様）
```

アニメーションファイルは以下にあります。

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\data\anim\
```

### 参考リンク

- [Don't Starve Wiki — Modding](https://dontstarve.wiki.gg/wiki/Modding) — 公式 Wiki の MOD セクション
- [Klei Forums — Mods and Tools](https://forums.kleientertainment.com/forums/forum/79-modding-tools-tutorials/) — 公式フォーラム
- [DST API ドキュメント](https://forums.kleientertainment.com/forums/topic/27583-documentation-for-modders/) — MOD 開発者向けドキュメント

## 付録: デバッグガイド

MOD 開発中にエラーが発生した場合の調査方法をまとめます。

### ログファイルの確認

DST のログファイルはエラーの原因を特定する最も重要な手段です。

| ログ | パス |
|---|---|
| クライアントログ | `Documents\Klei\DoNotStarveTogether\<クラスタID>\client_log.txt` |
| サーバーログ | `Documents\Klei\DoNotStarveTogether\<クラスタID>\Master\server_log.txt` |

> **ヒント**: ゲームを起動するたびにログは上書きされます。エラーが出たらすぐに確認しましょう。ログファイルの **末尾付近** にエラーメッセージが出ていることが多いです。

### ゲーム内コンソール

ゲーム中に **`~`**（チルダ）キーを押すとコンソールが開きます。以下のコマンドが役立ちます。

| コマンド | 説明 |
|---|---|
| `c_spawn("prefab名")` | 指定した prefab をプレイヤーの足元にスポーンさせる |
| `c_give("prefab名", 数量)` | 指定したアイテムをインベントリに追加する |
| `c_godmode()` | 無敵モードの切り替え |
| `c_supergodmode()` | 無敵＋空腹・正気度が減らなくなる |
| `c_speed(倍率)` | 移動速度を変更（デフォルト: 1） |
| `c_reset()` | ワールドをリセットする |
| `c_save()` | ワールドを保存する |
| `c_rollback(日数)` | 指定した日数分ロールバックする |

### よくあるエラーメッセージと対処法

#### `PANIC: Error loading module '...'`

```
PANIC: Error loading module 'speech_my_character'
```

**原因**: 指定された Lua ファイルが見つからない、または文法エラーがある。

**対処法**:
1. ファイルが `scripts/` フォルダ内の正しい場所に存在するか確認
2. ファイル名の大文字・小文字が一致しているか確認
3. Lua の文法エラーがないか確認（カンマ抜け、`end` の不足など）

#### `Could not find anim [...]`

```
Could not find anim [anim/my_character.zip]
```

**原因**: アニメーション ZIP ファイルが見つからない。

**対処法**:
1. `anim/` フォルダに ZIP ファイルが存在するか確認
2. `modmain.lua` の `Assets` テーブルに記載したパスと実際のファイルパスが一致しているか確認
3. autocompiler でビルドが成功しているか確認

#### `attempt to index a nil value`

```
my_character.lua:15: attempt to index a nil value
```

**原因**: 存在しない変数やテーブルにアクセスしようとしている。

**対処法**:
1. エラーメッセージに記載された **ファイル名と行番号** を確認
2. その行で使っている変数が正しく定義・初期化されているか確認
3. コンポーネントの追加順序が正しいか確認（コンポーネントを使う前に `AddComponent` が必要）

### デバッグ用 print 文

Lua コード内に `print()` を挿入すると、ログファイルに出力されます。

```lua
print("=== my_character: master_postinit called ===")
print("Health:", inst.components.health.maxhealth)
```

ログファイルで `===` を検索すれば、自分の出力を素早く見つけられます。

---

## ライセンス

このガイドおよびリポジトリ内のテンプレートは、学習目的で自由に使用できます。
このドキュメントは自由に利用・改変・再配布できます。DST MOD 制作にお役立てください。

---

<sub>Author: [pinpikokun](https://steamcommunity.com/profiles/76561198076111536/)</sub><br>
<sub>[![GitHub](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/pinpikokun/DST-teemo) [![Steam Workshop](https://img.shields.io/badge/Steam%20Workshop-Subscribe-blue?logo=steam)](https://steamcommunity.com/sharedfiles/filedetails/?id=390684095)</sub>
