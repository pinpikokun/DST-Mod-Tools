# DST-Lua-Scripting: Don't Starve Together MOD Lua スクリプティングガイド

Don't Starve Together（DST）の MOD では、Lua スクリプトを書くことで **キャラクターの固有能力・カスタムアイテム・特殊レシピ・マルチプレイ同期** など、ゲームプレイに関わるあらゆる機能を実装できます。

本ガイドでは、DST MOD の Lua スクリプティングを体系的に解説します。プログラミング初心者でも読み進められるよう、基本概念から実装パターンまで順を追って説明します。

> **HTML 版**: [README.html](README.html)

---

## 目次

- [このガイドで学べること](#このガイドで学べること)
- [このガイドで扱わないもの（別途ガイドあり）](#このガイドで扱わないもの別途ガイドあり)
- [前提知識・準備](#前提知識準備)
- [MOD の基本構成](#mod-の基本構成)
- [modinfo.lua — MOD の設定ファイル](#modinfolua--mod-の設定ファイル)
- [modmain.lua — MOD の起動スクリプト](#modmainlua--mod-の起動スクリプト)
- [キャラクター prefab の作成](#キャラクター-prefab-の作成)
- [カスタムアイテム・武器の作成](#カスタムアイテム武器の作成)
- [カスタムコンポーネントの作成](#カスタムコンポーネントの作成)
- [レシピ追加・クラフティング](#レシピ追加クラフティング)
- [ネットワーク同期（マルチプレイ対応）](#ネットワーク同期マルチプレイ対応)
- [UI ウィジェットの作成](#ui-ウィジェットの作成)
- [セーブ・ロード（データ永続化）](#セーブロードデータ永続化)
- [デバッグとトラブルシューティング](#デバッグとトラブルシューティング)
- [よくあるエラーと対処法](#よくあるエラーと対処法)
- [Tips・参考情報](#tips参考情報)
- [参考リンク](#参考リンク)
- [ライセンス](#ライセンス)

---

## このガイドで学べること

このガイドでは、DST の MOD 開発に必要な Lua スクリプティングの知識を体系的に学べます。

| トピック | 内容 |
|---|---|
| MOD の基本構成 | フォルダ構成、modinfo.lua、modmain.lua の役割 |
| キャラクター固有能力 | 体力・空腹・正気度の設定、特殊能力、イベント処理 |
| カスタムアイテム・武器 | prefab（プレハブ: ゲーム内オブジェクトの設計図）の定義、武器の作成 |
| カスタムコンポーネント | 独自の機能パーツの設計と実装 |
| レシピ・クラフティング | レシピ登録、カスタムクラフトタブ、キャラクター専用レシピ |
| ネットワーク同期 | マルチプレイでデータをサーバーとクライアント間で共有する仕組み |
| UI ウィジェット | HUD（画面上のボタンやアイコン表示）の追加 |
| セーブ・ロード | ゲームの保存・読み込み時にデータを維持する方法 |
| デバッグ | ログ出力、コンソールコマンド、よくあるエラー |

> **ポイント**: 本ガイドの概念を実際の MOD コードで学べる実践ガイドも用意しています。[README-DST-teemo.md](README-DST-teemo.md) では、Captain Teemo MOD を題材にした解説を行っています。

---

## このガイドで扱わないもの（別途ガイドあり）

| 内容 | 参照先ガイド |
|---|---|
| アニメーションの作成（Spriter） | [DST-Spriter ガイド](../DST-Spriter/README.md) |
| PNG から TEX への画像変換 | [DST-TextureConverter ガイド](../DST-TextureConverter/README.md) |
| ゲームアセットのデコンパイル | [DST-ktools ガイド](../DST-ktools/README.md) |
| サウンド・ボイスの作成 | [DST-FMOD-Designer ガイド](../DST-FMOD-Designer/README.md) |

---

## 前提知識・準備

### Lua 言語の基礎

DST の MOD は **Lua**（ルア）というプログラミング言語で書かれています。Lua は軽量でシンプルな言語で、ゲーム開発で広く使われています。

このガイドを読むために必要な Lua の知識は以下の通りです。

| 概念 | 説明 | 例 |
|---|---|---|
| 変数 | 値を入れる箱 | `local x = 10` |
| テーブル | 複数の値をまとめるデータの入れ物 | `local t = {a = 1, b = 2}` |
| 関数 | 処理をまとめたもの | `local function add(a, b) return a + b end` |
| if 文 | 条件分岐 | `if x > 0 then ... end` |
| for 文 | 繰り返し | `for i = 1, 10 do ... end` |
| require | 別ファイルの読み込み | `local MyModule = require("mymodule")` |

> **初心者向けTip:** Lua を全く知らなくても大丈夫です。このガイドのコード例にはすべてコメントで説明を付けています。読み進めるうちに自然と慣れていきます。

### 開発環境

以下のツールが必要です（[メインガイド](../README.md) で詳しく解説しています）。

- **Don't Starve Mod Tools**（Steam からインストール）
- **テキストエディタ**（VSCode 推奨）
- **DST 本体**（動作テスト用）

### バニラコードの参照方法

DST のバニラ（ゲーム本体）のスクリプトは、MOD 開発の最良のリファレンス（お手本）です。

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\data\databundles\scripts.zip
```

> **なぜバニラコードを参照するのか？**: DST の API（MOD から使える機能一覧）には公式ドキュメントがほとんどありません。既存のキャラクター（Wilson、Wendy など）や武器（blowdart など）の実装を読むことが、最も確実な学習方法です。

この ZIP を展開すると、以下のようなフォルダ構成になっています。

```
scripts/
├── prefabs/           ← キャラクター・アイテム等の設計図（wilson.lua, spear.lua 等）
├── components/        ← コンポーネント（機能パーツ: combat.lua, health.lua 等）
├── stategraphs/       ← ステートグラフ（アニメーション制御）
├── widgets/           ← UI ウィジェット（画面表示パーツ）
└── tuning.lua         ← ゲームバランス定数（TUNING テーブル）
```

---

## MOD の基本構成

キャラクター MOD の典型的なフォルダ構成です。MOD のフォルダは以下の場所にあります。

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\mods\my_character\
```

> **ポイント**: `my_character` の部分はあなたの MOD 名に置き換えてください。ここが MOD のルートフォルダ（一番上のフォルダ）になります。以降のファイルはすべてこのフォルダの中に作成します。

```
mods/my_character/
├── modinfo.lua                          ← MOD の設定ファイル（必須）
├── modmain.lua                          ← MOD の起動スクリプト（必須）
├── modicon.xml / .tex                   ← MOD のアイコン
├── anim/                                ← アニメーションファイル
├── bigportraits/                        ← キャラクター大ポートレート
├── images/
│   ├── avatars/                         ← アバター画像（64 x 64）
│   ├── map_icons/                       ← マップアイコン（64 x 64）
│   ├── saveslot_portraits/              ← セーブスロット画像（128 x 128）
│   ├── selectscreen_portraits/          ← キャラクター選択画面（256 x 512）
│   ├── inventoryimages/                 ← インベントリアイテム画像
│   └── hud/                             ← HUD（画面表示）用画像
├── scripts/
│   ├── prefabs/                         ← prefab 定義ファイル（ゲーム内オブジェクトの設計図）
│   │   ├── my_character.lua             ← キャラクター本体
│   │   └── my_item.lua                  ← カスタムアイテム
│   ├── components/                      ← カスタムコンポーネント（独自の機能パーツ）
│   └── widgets/                         ← カスタム UI ウィジェット（画面表示パーツ）
└── sound/                               ← サウンドファイル
```

### 3 つの重要ファイル

| ファイル | 役割 | 読み込みタイミング |
|---|---|---|
| `modinfo.lua` | MOD の基本情報（名前・作者・バージョン）と設定オプション | MOD 一覧画面で読み込み |
| `modmain.lua` | MOD の起動スクリプト。全ての初期化処理を書くファイル | MOD 有効化時に読み込み |
| `scripts/prefabs/*.lua` | 個々のゲーム内オブジェクト（キャラクター、アイテム等）の設計図 | ゲーム内で必要になった時 |

---

## modinfo.lua — MOD の設定ファイル

modinfo.lua は MOD の基本情報（名前、作者、バージョン）と、ユーザーが MOD の設定画面から変更できるオプションを定義するファイルです。

### ファイルの作成場所

MOD のルートフォルダ（一番上のフォルダ）に `modinfo.lua` というファイルを作成してください。

```
mods/my_character/modinfo.lua   ← このファイルを作成
```

> **ヒント**: VSCode でファイルを作成するには、エクスプローラー（左のファイル一覧）で MOD のフォルダを右クリック → 「新しいファイル」を選び、ファイル名 `modinfo.lua` を入力します。

### 基本テンプレート

以下の内容を `modinfo.lua` にコピー＆ペーストして、`-- ★書き換え:` の行を自分の MOD に合わせて変更してください。

```lua
-- ★書き換え: MOD の基本情報
name = "My Character MOD"
description = "A custom character for DST."
author = "YourName"
version = "1.0.0"

-- API バージョン（現在は 10 で固定）
api_version = 10

-- そのまま: DST 専用設定
dont_starve_compatible = false
reign_of_giants_compatible = false
dst_compatible = true

-- MOD アイコン
icon_atlas = "modicon.xml"
icon = "modicon.tex"

-- マルチプレイ設定
all_clients_require_mod = true   -- 全員がこの MOD を入れる必要がある
clients_only_mod = false          -- サーバー MOD である

-- ★変更任意: サーバーフィルタータグ（サーバー検索でこの MOD を見つけやすくする文字列）
server_filter_tags = {"my_character"}
```

> **なぜ `all_clients_require_mod = true` なのか？**: キャラクター MOD はアニメーションや画像を含むため、全プレイヤーがインストールしていないと表示が崩れます。

### configuration_options（設定オプション）

ユーザーが MOD の設定画面から変更できるオプションを定義します。先ほど作成した `modinfo.lua` の末尾に追加してください。

```lua
configuration_options = {
    -- ========== セクション見出し（設定画面での仕切り線） ==========
    -- name が空文字のエントリは設定画面でカテゴリの区切りとして表示されます
    {
        name = "",
        label = "Character Stats",  -- カテゴリ名
        hover = "",
        options = {{description = "", data = 0}},
        default = 0,
    },

    -- ========== 実際の設定項目 ==========
    {
        name = "health",                -- ★書き換え: GetModConfigData() で指定する名前
        label = "Health",               -- 設定画面に表示されるラベル
        hover = "Max health",           -- マウスオーバー時の説明
        options = {                     -- 選択肢
            {description = "100", data = 100},
            {description = "150", data = 150},
            {description = "200", data = 200},
        },
        default = 150,                  -- デフォルト値
    },
    {
        name = "damage_multiplier",
        label = "Damage Multiplier",
        hover = "Attack damage multiplier",
        options = {
            {description = "0.75x", data = 0.75},
            {description = "1.0x",  data = 1.0},
            {description = "1.5x",  data = 1.5},
        },
        default = 1.0,
    },
    {
        name = "enable_ability",
        label = "Special Ability",
        hover = "Enable or disable special ability",
        options = {
            {description = "On",  data = true},
            {description = "Off", data = false},
        },
        default = true,
    },
}
```

> **ポイント**: `name` を空文字にしたエントリはカテゴリの見出しとして使えます。設定項目が多い MOD では、カテゴリごとに区切ると見やすくなります。

---

## modmain.lua — MOD の起動スクリプト

modmain.lua は MOD が有効化された時に最初に実行されるファイルです。キャラクターの登録、素材ファイルの読み込み、設定値の取得、各種フック処理（ゲームの動作に割り込む処理）など、MOD の初期化処理をすべてここに書きます。

### ファイルの作成場所

MOD のルートフォルダに `modmain.lua` というファイルを作成してください。

```
mods/my_character/modmain.lua   ← このファイルを作成
```

### modmain.lua の実行環境

modmain.lua はサンドボックス環境（MOD 専用の制限された実行空間）で実行されます。DST のグローバル変数（`TheWorld`、`SpawnPrefab` 等のゲーム本体の機能）に直接アクセスできません。

```lua
-- NG: サンドボックス内なので直接アクセスできない
-- local player = ThePlayer

-- OK: GLOBAL テーブル経由でアクセスする
local player = GLOBAL.ThePlayer
```

> **なぜサンドボックス？**: MOD がゲーム本体のグローバル変数（全体で共有される値）を意図せず上書きしてしまうのを防ぐためです。`GLOBAL` を経由することで、意図的にアクセスしていることが明確になります。

### 設定値の読み込み

modinfo.lua で定義した設定オプションの値を取得します。以下の内容を `modmain.lua` の先頭付近に書いてください。

```lua
-- GetModConfigData() で modinfo.lua の name に対応する値を取得
-- or の後ろはデフォルト値（設定が読み込めなかった場合の代わりの値）
GLOBAL.MY_MOD_HEALTH = GetModConfigData("health") or 150
GLOBAL.MY_MOD_DAMAGE_MULT = GetModConfigData("damage_multiplier") or 1.0

-- boolean（真偽値: true/false）型は or で代替できないため、nil チェックが必要
GLOBAL.MY_MOD_ENABLE_ABILITY = GetModConfigData("enable_ability")
if GLOBAL.MY_MOD_ENABLE_ABILITY == nil then
    GLOBAL.MY_MOD_ENABLE_ABILITY = true
end
```

> **なぜ boolean は `or` が使えないのか？**: `false or true` は `true` になってしまいます。ユーザーが明示的に `false`（Off）を選んでも、常に `true` が返ってしまうため、nil（値が存在しない状態）チェックで分岐します。

### アセット（素材ファイル）の登録

MOD で使用する画像・アニメーション・サウンドファイルをゲームに読み込ませる宣言です。modmain.lua に以下を記述します。

```lua
Assets = {
    -- ポートレート画像
    Asset("IMAGE", "images/saveslot_portraits/my_character.tex"),
    Asset("ATLAS", "images/saveslot_portraits/my_character.xml"),

    Asset("IMAGE", "images/selectscreen_portraits/my_character.tex"),
    Asset("ATLAS", "images/selectscreen_portraits/my_character.xml"),

    Asset("IMAGE", "bigportraits/my_character.tex"),
    Asset("ATLAS", "bigportraits/my_character.xml"),

    -- マップアイコン
    Asset("IMAGE", "images/map_icons/my_character.tex"),
    Asset("ATLAS", "images/map_icons/my_character.xml"),

    -- アバター
    Asset("IMAGE", "images/avatars/avatar_my_character.tex"),
    Asset("ATLAS", "images/avatars/avatar_my_character.xml"),

    Asset("IMAGE", "images/avatars/avatar_ghost_my_character.tex"),
    Asset("ATLAS", "images/avatars/avatar_ghost_my_character.xml"),

    Asset("IMAGE", "images/avatars/self_inspect_my_character.tex"),
    Asset("ATLAS", "images/avatars/self_inspect_my_character.xml"),

    -- インベントリアイテム画像
    Asset("IMAGE", "images/inventoryimages/my_item.tex"),
    Asset("ATLAS", "images/inventoryimages/my_item.xml"),

    -- サウンド
    Asset("SOUNDPACKAGE", "sound/my_mod.fev"),
    Asset("SOUND", "sound/my_mod_bank00.fsb"),
}
```

### prefab（プレハブ: ゲーム内オブジェクトの設計図）の登録

MOD で追加するゲーム内オブジェクト（キャラクター、アイテム等）を宣言します。ここに書いた名前に対応する `scripts/prefabs/<名前>.lua` ファイルが自動的に読み込まれます。

```lua
PrefabFiles = {
    "my_character",       -- scripts/prefabs/my_character.lua を読み込む
    "my_character_none",  -- スキン定義ファイル
    "my_item",            -- scripts/prefabs/my_item.lua を読み込む
}
```

> **なぜ `PrefabFiles` に登録するのか？**: ここに宣言しないと、scripts/prefabs/ にファイルを置いてもゲームには読み込まれません。宣言し忘れると、そのオブジェクトはゲーム内に存在しないことになります。

### キャラクターの登録

キャラクターの名前・説明・セリフなどのテキスト情報を登録します。modmain.lua に記述してください。

```lua
-- キャラクターの表示テキスト
local STRINGS = GLOBAL.STRINGS

STRINGS.CHARACTER_TITLES.my_character = "The Custom Character"   -- 称号
STRINGS.CHARACTER_NAMES.my_character = "My Character"            -- 名前
STRINGS.CHARACTER_DESCRIPTIONS.my_character =                    -- 能力説明（選択画面）
    "*Has a special ability\n*Gains sanity near trees\n*Runs fast"
STRINGS.CHARACTER_QUOTES.my_character = "\"Ready for adventure!\""  -- 選択時セリフ
STRINGS.CHARACTER_ABOUTME.my_character = "A brave adventurer."     -- 自己紹介

-- セリフファイルの読み込み
STRINGS.CHARACTERS.MY_CHARACTER = GLOBAL.require "speech_my_character"

-- キャラクター名（他プレイヤーから見た時の名前）
STRINGS.NAMES.MY_CHARACTER = "My Character"

-- スキン名
STRINGS.SKIN_NAMES.my_character_none = "My Character"

-- 他キャラクターから見た時の説明
STRINGS.CHARACTERS.GENERIC.DESCRIBE.MY_CHARACTER = {
    GENERIC = "It's My Character!",
    ATTACKER = "That one looks dangerous...",
    MURDERER = "Murderer!",
    REVIVER = "A helpful soul.",
    GHOST = "My Character could use a heart.",
}

-- キャラクターを登録
local skin_modes = {
    {
        type = "ghost_skin",
        anim_bank = "ghost",
        idle_anim = "idle",
        scale = 0.75,
        offset = { 0, -25 },
    },
}
AddModCharacter("my_character", "MALE", skin_modes)
AddMinimapAtlas("images/map_icons/my_character.xml")

-- スキンの登録
GLOBAL.PREFAB_SKINS["my_character"] = { "my_character_none" }
```

### サウンドのリマップ（音声の関連付け）

カスタムサウンドを DST のキャラクターサウンドシステムに接続します。modmain.lua に追記します。

```lua
RemapSoundEvent("dontstarve/characters/my_character/death_voice",
    "my_mod/dontstarve/characters/my_character/death_voice")
RemapSoundEvent("dontstarve/characters/my_character/hurt",
    "my_mod/dontstarve/characters/my_character/hurt")
RemapSoundEvent("dontstarve/characters/my_character/talk_LP",
    "my_mod/dontstarve/characters/my_character/talk_LP")
```

### インベントリアイテム画像の登録

カスタムアイテムのインベントリ画像を登録します。modmain.lua に追記します。

```lua
RegisterInventoryItemAtlas("images/inventoryimages/my_item.xml", "my_item.tex")
```

### AddPrefabPostInit — 既存のゲーム内オブジェクトの改変

既存のゲーム内オブジェクト（バニラの焚き火やアイテム等）の動作を変更したい時に使います。modmain.lua に記述します。

```lua
-- 例: 焚き火の光の範囲を特定キャラクターの近くで拡大する
AddPrefabPostInit("campfire", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end

    -- 元の関数を保存
    local _onbuilt = inst.components.burnable.onextinguish

    -- 新しい関数で上書き
    inst.components.burnable.onextinguish = function(inst)
        -- 元の処理を実行
        if _onbuilt ~= nil then _onbuilt(inst) end
        -- 追加の処理
    end
end)
```

> **ポイント**: `AddPrefabPostInit` は元の関数を `_original` のように保存してから上書きするのが定番パターンです。元の処理を完全に消してしまうと、他の MOD やバニラの動作が壊れる可能性があります。

---

## キャラクター prefab の作成

キャラクターの能力・ステータス・イベント処理を定義する、MOD の中核ファイルです。

### ファイルの作成場所

MOD フォルダ内の `scripts/prefabs/` フォルダに、キャラクター名と同じ名前の Lua ファイルを作成します。`scripts/prefabs/` フォルダがまだ存在しない場合は、先にフォルダを作成してください。

```
mods/my_character/scripts/prefabs/my_character.lua   ← このファイルを作成
```

> **ヒント**: modmain.lua の `PrefabFiles` に書いた名前と、このファイル名（拡張子を除いた部分）が一致している必要があります。

### 基本テンプレート

以下の内容を `my_character.lua` にコピー＆ペーストして、`-- ★書き換え:` の行を自分のキャラクターに合わせて変更してください。

```lua
local MakePlayerCharacter = require("prefabs/player_common")

local assets = {
    Asset("ANIM", "anim/my_character.zip"),
    Asset("ANIM", "anim/ghost_my_character_build.zip"),
}
local prefabs = {}

-- ★書き換え: 初期所持アイテム
local start_inv = {
    "flint",
    "flint",
    "twigs",
    "twigs",
}

-- ========== common_postinit: クライアント＆サーバー両方で実行される部分 ==========
local function common_postinit(inst)
    -- サウンド名の設定
    inst.soundsname = "my_character"

    -- マップアイコンの設定
    inst.MiniMapEntity:SetIcon("my_character.tex")

    -- キャラクター識別用タグ（他のスクリプトからこのキャラクターかどうか判定できる目印）
    inst:AddTag("my_character")

    -- ★変更任意: ネットワーク変数の定義（後述のセクションで解説）
end

-- ========== master_postinit: サーバーのみで実行される部分 ==========
local function master_postinit(inst)
    -- ★書き換え: キャラクターステータス
    inst.components.health:SetMaxHealth(MY_MOD_HEALTH)
    inst.components.hunger:SetMax(150)
    inst.components.sanity:SetMax(200)

    -- ★変更任意: 戦闘パラメータ
    inst.components.combat.damagemultiplier = MY_MOD_DAMAGE_MULT
    inst.components.health:SetAbsorptionAmount(0)  -- ダメージ吸収率（0 = 軽減なし、1 = 全軽減）

    -- ★変更任意: 移動速度
    inst.components.locomotor.runspeed = TUNING.WILSON_RUN_SPEED * 1.25

    -- ========== イベントリスナー（特定のイベント発生時に自動実行される処理） ==========
    inst:ListenForEvent("onattackother", function(inst, data)
        -- 攻撃した時に実行される処理
        -- data.target: 攻撃した相手、data.weapon: 使った武器
    end)

    inst:ListenForEvent("attacked", function(inst, data)
        -- 攻撃を受けた時に実行される処理
        -- data.attacker: 攻撃してきた相手、data.damage: 受けたダメージ量
    end)

    inst:ListenForEvent("death", function(inst, data)
        -- 死亡した時に実行される処理
    end)

    inst:ListenForEvent("ms_respawnedfromghost", function(inst)
        -- ゴースト状態からリスポーン（復活）した時に実行される処理
    end)
end

return MakePlayerCharacter("my_character", prefabs, assets,
    common_postinit, master_postinit, start_inv)
```

### common_postinit と master_postinit の違い

| 関数 | 実行場所 | 用途 |
|---|---|---|
| `common_postinit` | クライアント＋サーバー | タグ追加、ネットワーク変数定義、見た目の設定 |
| `master_postinit` | サーバーのみ | ステータス設定、イベントリスナー、ゲームロジック |

> **なぜ分かれているのか？**: DST はマルチプレイゲームです。ゲームロジック（HP、ダメージ計算など）はサーバーだけが管理します。クライアント（各プレイヤーの PC）は見た目の表示だけを担当します。この分離により、チート防止とネットワーク効率が両立します。

### 主要なイベント一覧

キャラクターに `ListenForEvent`（イベント監視）で登録できる主なイベントです。

| イベント名 | 発火タイミング | data の内容 |
|---|---|---|
| `onattackother` | 攻撃した時 | `target`, `weapon` |
| `attacked` | 攻撃を受けた時 | `attacker`, `damage`, `weapon` |
| `death` | 死亡した時 | `cause` |
| `ms_respawnedfromghost` | ゴーストからリスポーンした時 | なし |
| `equipped` | アイテムを装備した時 | `item`, `eslot` |
| `unequipped` | アイテムを外した時 | `item`, `eslot` |
| `onpickup` | アイテムを拾った時 | `item` |
| `buildsuccess` | クラフト成功時 | なし |
| `oneatsomething` | 何かを食べた時 | `food` |
| `working` | 採掘・伐採等の作業中 | なし |
| `picksomething` | 草・花等を採取した時 | なし |
| `harvest` | 農作物等を収穫した時 | なし |

### DoPeriodicTask — 一定間隔で繰り返す処理

キャラクターの状態を定期的にチェックする処理は `DoPeriodicTask`（一定時間ごとに繰り返し実行する命令）で実装します。

```lua
-- 例: 夜間に正気度ボーナスを付与する能力
local function checkNightBonus(inst)
    if inst.components.health == nil then return end
    if inst.components.sanity == nil then return end

    -- 騎乗中は無効
    if inst.components.rider ~= nil and inst.components.rider:IsRiding() then
        return
    end

    -- 夜間判定
    if TheWorld.state.isnight then
        inst.components.sanity:DoDelta(0.5)  -- 正気度を少し回復
    end
end

-- master_postinit 内で:
inst._nightBonusTask = inst:DoPeriodicTask(5.0, checkNightBonus)
```

> **なぜ `DoPeriodicTask(5.0, ...)` なのか？**: 毎フレーム（約 1/30 秒）チェックする必要はありません。適切な間隔（秒数）を設定することで、サーバー負荷を大幅に抑えられます。目安として 0.5〜5.0 秒が一般的です。

### DoTaskInTime — 一定時間後に実行する処理

一定時間後に元に戻す処理は `DoTaskInTime`（指定秒数後に 1 回だけ実行する命令）で実装します。

```lua
-- 例: 攻撃成功時にダメージ倍率を 5 秒間アップ
inst:ListenForEvent("onattackother", function(inst, data)
    inst.components.combat.damagemultiplier = 1.5

    -- 前回のタスクをキャンセル（連続攻撃時にタイマーをリセット）
    if inst._resetDamageTask ~= nil then
        inst._resetDamageTask:Cancel()
    end
    inst._resetDamageTask = inst:DoTaskInTime(5.0, function()
        inst.components.combat.damagemultiplier = MY_MOD_DAMAGE_MULT
        inst._resetDamageTask = nil
    end)
end)
```

> **ポイント**: 一時的な効果を付与する際は、前回のタスクを `Cancel()`（取り消し）してから新しいタスクを登録します。これにより、連続で発動した場合でもタイマーが正しくリセットされます。

---

## カスタムアイテム・武器の作成

### ファイルの作成場所

MOD フォルダ内の `scripts/prefabs/` フォルダに、アイテム名の Lua ファイルを作成します。このフォルダはキャラクター prefab と同じ場所です。

```
mods/my_character/scripts/prefabs/my_weapon.lua   ← このファイルを作成
```

> **重要**: このファイル名（`my_weapon`）を modmain.lua の `PrefabFiles` にも追加してください。追加しないとゲームに読み込まれません。

### 基本テンプレート（武器アイテム）

以下の内容を `my_weapon.lua` にコピー＆ペーストしてください。

```lua
local assets = {
    Asset("ANIM", "anim/my_weapon.zip"),
    Asset("ANIM", "anim/swap_my_weapon.zip"),
    Asset("IMAGE", "images/inventoryimages/my_weapon.tex"),
    Asset("ATLAS", "images/inventoryimages/my_weapon.xml"),
}

local function onequip(inst, owner)
    -- 装備時のアニメーション切り替え
    owner.AnimState:OverrideSymbol("swap_object", "swap_my_weapon", "swap_my_weapon")
    owner.AnimState:Show("ARM_carry")
    owner.AnimState:Hide("ARM_normal")

    -- ★変更任意: 装備時に攻撃間隔を変更
    if owner.components.combat then
        owner.components.combat.min_attack_period = 1.0
    end
end

local function onunequip(inst, owner)
    -- 装備解除時のアニメーション復元
    owner.AnimState:Hide("ARM_carry")
    owner.AnimState:Show("ARM_normal")

    -- 攻撃間隔を通常に戻す
    if owner.components.combat then
        owner.components.combat.min_attack_period = TUNING.WILSON_ATTACK_PERIOD
    end
end

local function fn()
    local inst = CreateEntity()

    -- ========== 共通セットアップ（クライアント＆サーバー両方で実行） ==========
    inst.entity:AddTransform()
    inst.entity:AddAnimState()
    inst.entity:AddSoundEmitter()
    inst.entity:AddNetwork()

    MakeInventoryPhysics(inst)

    inst.AnimState:SetBank("my_weapon")
    inst.AnimState:SetBuild("my_weapon")
    inst.AnimState:PlayAnimation("idle")

    -- タグ（他のスクリプトからこのアイテムを識別するための目印）
    inst:AddTag("sharp")
    inst:AddTag("weapon")

    inst.entity:SetPristine()

    -- ========== ここから下はサーバーのみで実行される ==========
    if not TheWorld.ismastersim then
        return inst
    end

    -- 武器コンポーネント（武器としての機能を付与する部品）
    inst:AddComponent("weapon")
    inst.components.weapon:SetDamage(34)  -- ★書き換え: ダメージ値

    -- 調査（マウスオーバー時の説明表示を可能にする）
    inst:AddComponent("inspectable")

    -- インベントリ（アイテムとして持てるようにする）
    inst:AddComponent("inventoryitem")
    inst.components.inventoryitem.atlasname = "images/inventoryimages/my_weapon.xml"

    -- 耐久力（使用回数に制限を付ける。不要なら削除してもOK）
    inst:AddComponent("finiteuses")
    inst.components.finiteuses:SetMaxUses(100)  -- ★書き換え: 最大使用回数
    inst.components.finiteuses:SetUses(100)
    inst.components.finiteuses:SetOnFinished(function(inst)
        inst:DoTaskInTime(0, function() inst:Remove() end)
    end)

    -- 装備可能（手に持てるようにする）
    inst:AddComponent("equippable")
    inst.components.equippable:SetOnEquip(onequip)
    inst.components.equippable:SetOnUnequip(onunequip)

    return inst
end

return Prefab("common/inventory/my_weapon", fn, assets)
```

### prefab の構造のポイント

prefab ファイルは、以下の順番でオブジェクトを組み立てます。

```
CreateEntity()                 ← 空のオブジェクトを作成
    ↓
基本パーツの追加（Transform, AnimState, Network 等）
    ↓
物理設定（MakeInventoryPhysics 等）
    ↓
アニメーション設定（Bank, Build, Animation）
    ↓
タグ追加（識別用の目印を付ける）
    ↓
SetPristine()                  ← クライアントとサーバーの境界線
    ↓
if not TheWorld.ismastersim then return inst end
    ↓
サーバー専用コンポーネント追加（weapon, inventoryitem 等）
    ↓
return inst
```

> **重要**: `SetPristine()` と `ismastersim` チェックの間にコンポーネント（機能パーツ）を追加してはいけません。`SetPristine()` より上はクライアントでも実行される部分、下はサーバーのみです。

---

## カスタムコンポーネントの作成

コンポーネント（機能パーツ）は、ゲーム内オブジェクトに特定の機能を付与するモジュール（部品）です。DST のバニラには `health`（体力）、`combat`（戦闘）、`inventory`（所持品）など多数のコンポーネントがあります。独自のコンポーネントを作ることで、機能を整理して複数のオブジェクトで使い回せるようにできます。

### ファイルの作成場所

MOD フォルダ内の `scripts/components/` フォルダに、コンポーネント名の Lua ファイルを作成します。`scripts/components/` フォルダがまだ存在しない場合は、先にフォルダを作成してください。

```
mods/my_character/scripts/components/my_component.lua   ← このファイルを作成
```

### コンポーネントの基本テンプレート

以下の内容を `my_component.lua` にコピー＆ペーストしてください。

```lua
local MyComponent = Class(function(self, inst)
    self.inst = inst       -- このコンポーネントが付いているゲーム内オブジェクト

    -- ★書き換え: プロパティ（設定値）の初期値
    self.value = 0
    self.enabled = true
end)

-- 値の取得
function MyComponent:GetValue()
    return self.value
end

-- 値の設定
function MyComponent:SetValue(val)
    self.value = val
end

-- 機能の実行
function MyComponent:DoSomething()
    if not self.enabled then return end
    -- ここに処理内容を書く
end

-- セーブ・ロード（ゲームの保存・読み込みに対応させる場合）
function MyComponent:OnSave()
    return {
        value = self.value,
        enabled = self.enabled,
    }
end

function MyComponent:OnLoad(data)
    if data then
        self.value = data.value or self.value
        self.enabled = data.enabled ~= false
    end
end

return MyComponent
```

### コンポーネントの使い方

prefab ファイル内で、以下のようにコンポーネントをオブジェクトに取り付けます。

```lua
-- prefab ファイル内でコンポーネントを追加
inst:AddComponent("my_component")
inst.components.my_component:SetValue(100)
```

> **なぜコンポーネントを使うのか？**: コンポーネントを使うことで、同じ機能を複数のオブジェクトに付与できます。例えば「キャラクター専用」コンポーネントを作れば、複数のアイテムにキャラクター制限を付けられます。

### キャラクター専用アイテム制限の例

```lua
-- scripts/components/characterspecific.lua
local CharacterSpecific = Class(function(self, inst)
    self.inst = inst
    self.character = nil     -- 許可されたキャラクター名
    self.storable = false    -- 他キャラクターが保管できるかどうか
end)

function CharacterSpecific:SetOwner(name)
    self.character = name
end

function CharacterSpecific:IsStorable()
    return self.storable
end

function CharacterSpecific:SetStorable(value)
    self.storable = value
end

return CharacterSpecific
```

---

## レシピ追加・クラフティング

### カスタムクラフトタブの作成

キャラクター専用のクラフトタブ（クラフト画面に表示されるカテゴリ）を追加できます。modmain.lua に記述してください。

```lua
local myTab = AddRecipeTab(
    "My Items",                                              -- タブ名
    998,                                                     -- ソート順（数字が大きいほど右に表示）
    GLOBAL.resolvefilepath("images/hud/my_tab.xml"),         -- タブアイコンの画像ファイル
    "my_tab.tex",                                            -- アイコンのテクスチャ名
    "my_character"                                           -- このタグを持つキャラクターのみ表示
)
```

### レシピの登録

modmain.lua に以下を追加します。

```lua
AddRecipe2("my_item", {
    -- ★書き換え: 材料
    GLOBAL.Ingredient("boards", 1),      -- 板 x1
    GLOBAL.Ingredient("silk", 2),        -- シルク x2
    GLOBAL.Ingredient("rope", 1),        -- ロープ x1
}, GLOBAL.TECH.NONE, {                   -- 必要テクノロジー（NONE = いつでもクラフト可能）
    atlas = GLOBAL.resolvefilepath("images/inventoryimages/my_item.xml"),
    image = "my_item.tex",
    builder_tag = "my_character",        -- このタグを持つキャラクターのみクラフト可能
    tab = myTab,                         -- 表示するタブ
}, {"CHARACTER"})                        -- レシピフィルタ（キャラクター専用レシピ）
```

### アイテム名とレシピ説明の登録

modmain.lua に追記します。

```lua
STRINGS.NAMES.MY_ITEM = "My Item"
STRINGS.RECIPE_DESC.MY_ITEM = "A special item."
```

> **注意**: `STRINGS.NAMES` と `STRINGS.RECIPE_DESC` のキーは **大文字** です。prefab 名が `my_item` なら、キーは `MY_ITEM` になります。

### 主なテクノロジーレベル

| 定数 | 意味 |
|---|---|
| `TECH.NONE` | いつでもクラフト可能 |
| `TECH.SCIENCE_ONE` | サイエンスマシンが必要 |
| `TECH.SCIENCE_TWO` | 錬金エンジンが必要 |
| `TECH.MAGIC_TWO` | プレスティハッティテイターが必要 |

---

## ネットワーク同期（マルチプレイ対応）

DST はクライアント・サーバーモデル（各プレイヤーの PC がクライアント、ゲームのホストがサーバー）のマルチプレイゲームです。ゲームロジックはサーバーが管理し、クライアントは画面表示を担当します。サーバーの値をクライアントに通知するには **Net Variables**（ネットワーク変数）と **RPC** を使います。

### Net Variables（ネットワーク変数）とは

サーバーで値を変更すると、自動的に全クライアントにその値が同期（共有）される特殊な変数です。たとえば「クールダウンの残り秒数」をサーバーで減らすと、全プレイヤーの画面に自動で反映されます。

| 型 | 扱える値の範囲 | 用途の例 |
|---|---|---|
| `net_byte` | 0〜255 の整数 | スタック数、小さなカウンター |
| `net_ushortint` | 0〜65535 の整数 | クールダウン秒数 |
| `net_bool` | true（はい）/ false（いいえ） | 能力の有効/無効 |
| `net_float` | 小数点付きの数値 | パーセンテージ値 |
| `net_event` | イベント通知のみ（値なし） | サウンド再生のきっかけ |

### Net Variables の定義と使い方

キャラクター prefab ファイル（`scripts/prefabs/my_character.lua`）の中に記述します。

```lua
-- common_postinit 内（クライアント＆サーバー両方で実行される部分）
local function common_postinit(inst)
    -- Net Variables の定義
    -- 第1引数: オブジェクトの固有ID
    -- 第2引数: この変数を識別するための名前（重複しないようにする）
    -- 第3引数: 値が変わった時に発生するイベントの名前
    inst._myCounter = net_byte(inst.GUID, "my_character._myCounter", "mycounterdirty")
    inst._myCooldown = net_ushortint(inst.GUID, "my_character._myCooldown", "mycooldowndirty")

    -- サウンド通知用（値のない純粋なイベント通知）
    inst._sound_attack = net_event(inst.GUID, "my_character._sound_attack")

    -- クライアント側でイベントを受信して画面表示を更新する
    inst:ListenForEvent("mycounterdirty", function()
        -- UI を更新するなどの処理
        local value = inst._myCounter:value()
    end)

    -- net_event のリスナー（サウンド再生のきっかけを受け取る）
    inst:ListenForEvent("my_character._sound_attack", function()
        inst.SoundEmitter:PlaySound("dontstarve/characters/my_character/attack")
    end)
end

-- master_postinit 内（サーバーのみで実行される部分）
local function master_postinit(inst)
    -- 値の設定（サーバーが値を変更 → 自動でクライアントに同期される）
    inst._myCounter:set(3)

    -- カウントダウン処理（1秒ごとにクールダウンを1減らす）
    inst:DoPeriodicTask(1, function()
        local cd = inst._myCooldown:value()
        if cd > 0 then
            inst._myCooldown:set(cd - 1)
        end
    end)

    -- サウンドを全クライアントに通知（音を鳴らすきっかけを送る）
    inst._sound_attack:push()
end
```

### RPC（Remote Procedure Call: 遠隔手続き呼び出し）とは

クライアント（プレイヤーの PC）からサーバーに「この処理を実行してほしい」とリクエストを送る仕組みです。たとえば、プレイヤーが画面上のボタンをクリックした時に、サーバーに「能力を発動してほしい」と依頼する場合に使います。

以下の内容を modmain.lua に記述します。

```lua
-- サーバー側 RPC ハンドラ（クライアントからのリクエストを受け取って処理する部分）
AddModRPCHandler("my_mod", "use_ability", function(player)
    -- バリデーション（不正なリクエストを防ぐチェック）
    if not player:HasTag("my_character") then return end
    if player:HasTag("playerghost") then return end
    if player._myCooldown == nil or player._myCooldown:value() > 0 then return end

    -- 能力の処理
    player._myCooldown:set(60)  -- 60 秒クールダウン開始

    -- 例: 周囲の敵にダメージを与える
    local x, y, z = player.Transform:GetWorldPosition()
    local ents = GLOBAL.TheSim:FindEntities(x, y, z, 8, {"_combat"}, {"player"})
    for _, ent in pairs(ents) do
        if ent.components.combat then
            ent.components.combat:GetAttacked(player, 20)
        end
    end
end)

-- クライアント側 RPC ハンドラ（サーバーからの通知を受け取って処理する部分）
AddClientModRPCHandler("my_mod", "show_effect", function(x, y, z, value)
    -- UI 表示やエフェクトの処理
end)
```

### RPC の送信方法

```lua
-- クライアント → サーバー（プレイヤーが操作した時にサーバーに依頼を送る）
SendModRPCToServer(MOD_RPC["my_mod"]["use_ability"], x, z)

-- サーバー → 特定クライアント（特定のプレイヤーだけに通知を送る）
GLOBAL.SendModRPCToClient(
    GLOBAL.CLIENT_MOD_RPC["my_mod"]["show_effect"],
    player.userid, x, y, z, value
)

-- サーバー → 範囲内の全クライアント（近くにいる全プレイヤーに通知を送る）
local players = GLOBAL.FindPlayersInRange(x, y, z, 40)
for _, player in pairs(players) do
    GLOBAL.SendModRPCToClient(
        GLOBAL.CLIENT_MOD_RPC["my_mod"]["show_effect"],
        player.userid, x, y, z, value
    )
end
```

> **なぜ RPC でバリデーション（チェック処理）が必要なのか？**: クライアントからのリクエストは改ざんされる可能性があります。タグチェック、クールダウンチェック、範囲チェックなど、サーバー側で必ず検証してください。

---

## UI ウィジェットの作成

UI ウィジェット（画面に表示するボタンやアイコンなどの部品）を使って、HUD（ゲーム画面上の操作パネル）にカスタム要素（ボタン、カウンター、クールダウン表示など）を追加できます。

### AddClassPostConstruct — 既存 UI の拡張

既存の画面表示に自分のウィジェットを追加する方法です。modmain.lua に記述します。

```lua
-- modmain.lua 内
AddClassPostConstruct("widgets/inventorybar", function(self)
    -- 特定キャラクターのみ
    if not self.owner:HasTag("my_character") then return end

    local MyWidget = require("widgets/my_widget")

    -- 既存の Rebuild メソッド（再構築処理）をラップ（元の処理を包み込む）
    local _Rebuild = self.Rebuild
    self.Rebuild = function(self, ...)
        -- 既存のカスタムウィジェットを破棄
        if self.mywidget ~= nil then
            self.mywidget:Kill()
            self.mywidget = nil
        end

        -- 元の Rebuild を実行
        _Rebuild(self, ...)

        -- カスタムウィジェットを追加
        self.mywidget = self.toprow:AddChild(MyWidget(self.owner))
        self.mywidget:SetPosition(0, 0, 0)
    end

    self.rebuild_pending = true
end)
```

### ファイルの作成場所

MOD フォルダ内の `scripts/widgets/` フォルダに、ウィジェット名の Lua ファイルを作成します。`scripts/widgets/` フォルダがまだ存在しない場合は、先にフォルダを作成してください。

```
mods/my_character/scripts/widgets/my_widget.lua   ← このファイルを作成
```

### カスタムウィジェットの基本テンプレート

以下の内容を `my_widget.lua` にコピー＆ペーストしてください。

```lua
local Widget = require("widgets/widget")
local Image = require("widgets/image")
local ImageButton = require("widgets/imagebutton")
local Text = require("widgets/text")

local MyWidget = Class(Widget, function(self, owner)
    Widget._ctor(self, "MyWidget")
    self.owner = owner

    -- 背景画像
    self.bg = self:AddChild(Image("images/hud/my_bg.xml", "my_bg.tex"))

    -- ボタン（クリックできるアイコン画像）
    self.button = self:AddChild(ImageButton(
        "images/hud/my_icon.xml", "my_icon.tex"
    ))
    self.button:SetOnClick(function()
        self:OnClick()
    end)

    -- テキスト表示（数字やメッセージを画面に表示する）
    self.text = self:AddChild(Text(NUMBERFONT, 28, "0"))
    self.text:SetPosition(10, -15)
    self.text:SetColour(1, 1, 1, 1)  -- 色（赤, 緑, 青, 透明度）各 0〜1

    -- ネットワーク変数の変更を監視（サーバーから値が届いた時に表示を更新）
    self.inst:ListenForEvent("mycounterdirty", function()
        self:UpdateDisplay()
    end, owner)

    self:UpdateDisplay()
end)

function MyWidget:OnClick()
    -- RPC でサーバーに送信（「能力を発動してほしい」というリクエストを送る）
    SendModRPCToServer(MOD_RPC["my_mod"]["use_ability"])
end

function MyWidget:UpdateDisplay()
    local value = self.owner._myCounter:value()
    self.text:SetString(tostring(value))

    -- 値が 0 の時はグレーアウト（暗くして使用不可を表現）
    if value <= 0 then
        self.button:SetTint(0.5, 0.5, 0.5, 1)
    else
        self.button:SetTint(1, 1, 1, 1)
    end
end

return MyWidget
```

---

## セーブ・ロード（データ永続化）

ゲームを保存・読み込みした時に、MOD のデータを維持するための仕組みです。この処理を入れないと、ゲームを閉じて再開した時にクールダウンやカウンターがリセットされてしまいます。

### キャラクター prefab での OnSave/OnLoad

キャラクター prefab ファイル（`scripts/prefabs/my_character.lua`）の `master_postinit` 内に記述します。

```lua
-- master_postinit 内

-- セーブ（ゲーム保存時に呼ばれる）
local _OnSave = inst.OnSave
inst.OnSave = function(inst, data)
    if _OnSave then _OnSave(inst, data) end
    data.my_counter = inst._myCounter:value()
    data.my_cooldown = inst._myCooldown:value()
    data.my_timer = inst._myTimer
end

-- ロード（ゲーム読み込み時に呼ばれる）
local _OnLoad = inst.OnLoad
inst.OnLoad = function(inst, data)
    if _OnLoad then _OnLoad(inst, data) end
    if data then
        if data.my_counter then
            inst._myCounter:set(data.my_counter)
        end
        if data.my_cooldown then
            inst._myCooldown:set(data.my_cooldown)
        end
        if data.my_timer then
            inst._myTimer = data.my_timer
        end
    end
end
```

> **なぜ `_OnSave` を保存するのか？**: `MakePlayerCharacter` がゲームの内部で `OnSave` / `OnLoad` を定義しています。それを上書きしてしまうと、バニラの保存機能が壊れます。元の関数を保存して呼び出すことで、安全に機能を拡張できます。

### アイテム prefab での OnSave/OnLoad

アイテムの prefab ファイル内に記述します。

```lua
local function onSave(inst, data)
    data.is_activated = inst.isActivated
    data.elapsed_time = inst.elapsedTime
end

local function onLoad(inst, data)
    if data then
        if data.is_activated then
            inst.isActivated = data.is_activated
            -- 状態の復元処理
        end
        if data.elapsed_time then
            inst.elapsedTime = data.elapsed_time
        end
    end
end

-- fn()（prefab の生成関数）内で:
inst.OnSave = onSave
inst.OnLoad = onLoad
```

### 保存できるデータ型

| 型 | 例 |
|---|---|
| 数値 | `data.count = 5` |
| 文字列 | `data.name = "hello"` |
| 真偽値 | `data.enabled = true` |
| テーブル（データの入れ物） | `data.items = {1, 2, 3}` |

> **注意**: 関数、ゲーム内オブジェクトの参照（inst）、メタテーブルは保存できません。保存が必要な場合は、固有 ID や prefab 名などの文字列・数値に変換してから保存してください。

---

## デバッグとトラブルシューティング

### ログ出力

問題が起きた時、まずはログ（動作記録）を確認するのが基本です。

```lua
-- modmain.lua やprefab 内で print() を使うとログファイルに出力される
print("My MOD: value =", someValue)

-- ログファイルの場所
-- C:\Users\<ユーザー名>\Documents\Klei\DoNotStarveTogether\client_log.txt
-- C:\Users\<ユーザー名>\Documents\Klei\DoNotStarveTogether\Cluster_X\Master\server_log.txt
```

### コンソールコマンド

ゲーム内でコンソール（`~` キーで開く入力欄）を使って実行できるデバッグ用コマンドです。

```lua
-- 選択中のオブジェクトの情報を表示
c_select()

-- 選択中オブジェクトに付いているコンポーネント（機能パーツ）の一覧を表示
for k, v in pairs(c_select().components) do print(k) end

-- アイテムをプレイヤーの所持品に追加
c_give("my_item")

-- 指定した prefab をプレイヤーの近くにスポーン（出現させる）
c_spawn("my_item")

-- 無敵モード（テスト用）
c_godmode()

-- ワールド再生成
c_regenerateworld()

-- prefab のリロード（コードの変更をゲームに即時反映）
c_reset()
```

### デバッグの進め方

ゲーム内コンソールの入力は1行に限られます。複雑なデバッグには、以下の手順で対応できます。

1. MOD 内の怪しい箇所に `print()` を仕込む
2. ゲームを起動して動作させる
3. `client_log.txt` でログを確認する

> **初心者向けTip:** `print()` は最も基本的で確実なデバッグ手段です。問題が起きたら、まず怪しい箇所に `print()` を入れてログを確認しましょう。

---

## よくあるエラーと対処法

| 症状 | 原因 | 解決法 |
|---|---|---|
| `attempt to index a nil value` | 存在しないコンポーネントやプロパティにアクセスした | `if inst.components.health then` のように nil（値が存在しない状態）チェックを追加 |
| `variable is not declared` | modmain.lua でグローバル変数に直接アクセスした | `GLOBAL.TheWorld` のように `GLOBAL` を経由する |
| キャラクターが表示されない | PrefabFiles に登録していない | modmain.lua の `PrefabFiles` にファイル名を追加 |
| アイテム画像が白い四角 | Asset の登録漏れまたはパスの間違い | `Assets` テーブルと実際のファイルパスを確認 |
| レシピが表示されない | `builder_tag` とキャラクターのタグが不一致 | `AddTag()` と `builder_tag` が同じ文字列か確認 |
| クラフトが「Character only」と表示 | `AddRecipe2` の第5引数が不正 | `{"CHARACTER"}` を指定しているか確認 |
| マルチプレイで他プレイヤーに反映されない | サーバー専用処理をクライアントで実行している | `ismastersim` チェックの位置を確認、Net Variables を使用 |
| セーブ後にデータが消える | OnSave/OnLoad の実装漏れ | Net Variables の値を OnSave で保存しているか確認 |
| `attempt to call a nil value` | 関数名のタイプミスまたは require のパス間違い | 関数名とファイルパスのスペルを確認 |

---

## Tips・参考情報

### バニラコードの読み方のコツ

- **似た機能のキャラクター** を探して実装を読む（例: Wortox のテレポート → `scripts/prefabs/wortox.lua`）
- **バニラのコンポーネント** をまず調べる。独自に作る前に、既存のコンポーネントで代替できないか確認する
- `TUNING.lua` にはゲームバランス定数がすべて定義されている（HP、ダメージ、速度など）

### パフォーマンス（動作速度）のコツ

- `DoPeriodicTask` の間隔は最低 0.3〜0.5 秒にする（短すぎるとサーバー負荷が増大）
- `FindEntities`（周囲のオブジェクト検索）の検索範囲は必要最小限にする
- クライアント専用の描画処理は `if not TheNet:IsDedicated()` で囲む

### マルチプレイ対応のチェックリスト

- ゲームロジックは `ismastersim` チェックの後に書いているか
- プレイヤーに見せたい値は Net Variables（ネットワーク変数）で同期しているか
- クライアントからサーバーへの指示は RPC 経由にしているか
- RPC ハンドラにバリデーション（タグ・クールダウン・範囲チェック）を入れているか

---

## 参考リンク

- [Don't Starve Together Modding Guide（Klei Forums）](https://forums.kleientertainment.com/forums/topic/72-dont-starve-together-api/)
- [DST API Documentation（非公式 Wiki）](https://dontstarve.fandom.com/wiki/Guides/Don%27t_Starve_Together_Lua_Modding)
- バニラソースコード: `C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\data\databundles\scripts.zip`

---

## ライセンス

このドキュメントは自由に利用・改変・再配布できます。DST MOD 制作にお役立てください。

---

<sub>Author: [pinpikokun](https://steamcommunity.com/profiles/76561198076111536/)</sub><br>
<sub>[![GitHub](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/pinpikokun/DST-teemo) [![Steam Workshop](https://img.shields.io/badge/Steam%20Workshop-Subscribe-blue?logo=steam)](https://steamcommunity.com/sharedfiles/filedetails/?id=390684095)</sub>
