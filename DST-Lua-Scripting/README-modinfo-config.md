# DST modinfo.lua コンフィグオプション詳細ガイド — プレイヤーが設定を変更できる MOD を作ろう

Don't Starve Together (DST) の MOD で、プレイヤーがゲーム内の設定画面から値を変更できる **コンフィグオプション（configuration_options）** の作り方を解説します。体力や攻撃力をユーザーが自由に調整できる MOD を作りましょう。

> **HTML 版**: [README-modinfo-config.html](https://pinpikokun.github.io/DST-Mod-Guide/DST-Lua-Scripting/README-modinfo-config.html)

---

## 目次

1. [コンフィグオプションとは](#コンフィグオプションとは)
2. [configuration_options の基本構造](#configuration_options-の基本構造)
3. [データ型別の書き方](#データ型別の書き方)
4. [セクション見出しでグループ分けする](#セクション見出しでグループ分けする)
5. [modmain.lua での設定値の取得](#modmainlua-での設定値の取得)
6. [prefab ファイルでの設定値の使い方](#prefab-ファイルでの設定値の使い方)
7. [実践例: キャラクター MOD の設定テンプレート](#実践例-キャラクター-mod-の設定テンプレート)
8. [よくあるトラブルと解決法](#よくあるトラブルと解決法)
9. [Tips・参考情報](#tips参考情報)
10. [参考リンク](#参考リンク)

---

## コンフィグオプションとは

コンフィグオプションとは、MOD の設定画面でプレイヤーが値を変更できる仕組みです。

例えば、キャラクター MOD で以下のような設定を追加できます。

- キャラクターの体力を 100 / 150 / 200 から選べるようにする
- 攻撃力の倍率を 0.75x / 1.0x / 1.5x から選べるようにする
- 特殊能力の ON / OFF を切り替えられるようにする
- 移動速度の倍率を調整できるようにする

### 設定画面の開き方

プレイヤーは以下の手順で MOD の設定画面を開きます。

1. DST のメインメニューで **「ゲームをホスト」** を選択
2. **「MOD」** タブを開く
3. 設定したい MOD の **「設定」** ボタンをクリック
4. 各項目を選択して **「適用」** をクリック

> **なぜコンフィグオプションを使うのか？**: ステータス値をコードに直接書き込む（ハードコーディングする）と、値を変更するたびにコードの編集が必要です。コンフィグオプションを使えば、プレイヤーがゲーム内から簡単に値を変更でき、MOD の自由度が大幅に上がります。

---

## configuration_options の基本構造

コンフィグオプションは `modinfo.lua` の中に `configuration_options` テーブルとして定義します。

### 1 つの設定項目の構造

```lua
{
    name = "health",                -- 設定項目の内部名（GetModConfigData で指定する名前）
    label = "Health",               -- 設定画面に表示されるラベル
    hover = "Max health",           -- マウスオーバー時の説明文
    options = {                     -- 選択肢の一覧
        {description = "100", data = 100},   -- description: 表示テキスト、data: 実際の値
        {description = "150", data = 150},
        {description = "200", data = 200},
    },
    default = 150,                  -- デフォルト値（options の data のいずれかと一致させる）
},
```

### 各フィールドの意味

| フィールド | 型 | 説明 |
|---|---|---|
| `name` | 文字列 | 設定項目の識別名。`GetModConfigData("name")` で値を取得する時に使う |
| `label` | 文字列 | 設定画面でプレイヤーに表示されるラベル |
| `hover` | 文字列 | 設定画面でマウスを重ねた時に表示される説明文 |
| `options` | テーブル配列 | 選択肢のリスト。各要素に `description`（表示名）と `data`（値）を持つ |
| `default` | 任意 | デフォルト値。`options` 内のいずれかの `data` と一致させること |

> **重要**: `name` は MOD 内で一意（重複しない）にしてください。同じ `name` を持つ設定項目が複数あると、正しく動作しません。

> **ポイント**: `default` の値は `options` 内の `data` のいずれかと完全に一致する必要があります。一致しない場合、設定画面で初期選択が正しく表示されません。

---

## データ型別の書き方

### 数値型（整数）

体力、空腹度、正気度など、整数値を設定する場合のパターンです。

```lua
{
    name = "health",
    label = "Health",
    hover = "キャラクターの最大体力",
    options = {
        {description = "50",  data = 50},
        {description = "100", data = 100},
        {description = "150", data = 150},   -- ★ ウィルソンと同じ
        {description = "200", data = 200},
        {description = "250", data = 250},
        {description = "300", data = 300},
    },
    default = 150,
},
```

### 数値型（小数）

速度倍率やダメージ倍率など、小数値を設定する場合のパターンです。

```lua
{
    name = "speed_multiplier",
    label = "Move Speed",
    hover = "移動速度の倍率",
    options = {
        {description = "0.75x (遅い)",  data = 0.75},
        {description = "1.0x (通常)",   data = 1.0},
        {description = "1.25x (やや速い)", data = 1.25},
        {description = "1.5x (速い)",   data = 1.5},
        {description = "2.0x (とても速い)", data = 2.0},
    },
    default = 1.0,
},
```

> **ヒント**: `description` に倍率の意味（「通常」「速い」等）を添えると、プレイヤーが選びやすくなります。

### 真偽値型（ON / OFF）

機能の有効・無効を切り替える場合のパターンです。

```lua
{
    name = "enable_ability",
    label = "Special Ability",
    hover = "特殊能力の有効/無効",
    options = {
        {description = "On",  data = true},
        {description = "Off", data = false},
    },
    default = true,
},
```

> **注意**: 真偽値型（boolean）は modmain.lua での取得方法が他の型と異なります。詳しくは [modmain.lua での設定値の取得](#modmainlua-での設定値の取得) を参照してください。

### 文字列型（モード選択）

複数のモードから 1 つを選択する場合のパターンです。

```lua
{
    name = "difficulty",
    label = "Difficulty",
    hover = "ゲームの難易度設定",
    options = {
        {description = "Easy",   data = "easy"},
        {description = "Normal", data = "normal"},
        {description = "Hard",   data = "hard"},
    },
    default = "normal",
},
```

---

## セクション見出しでグループ分けする

設定項目が多い MOD では、カテゴリごとにグループ分けすると見やすくなります。`name` を空文字にしたエントリを挿入すると、設定画面でセクション見出し（区切り線）として表示されます。

```lua
configuration_options = {
    -- ========== キャラクターステータス ==========
    {
        name = "",                           -- ★ 空文字 = セクション見出し
        label = "Character Stats",           -- 見出しに表示されるテキスト
        hover = "",
        options = {{description = "", data = 0}},
        default = 0,
    },

    -- 体力の設定
    {
        name = "health",
        label = "Health",
        hover = "キャラクターの最大体力",
        options = {
            {description = "100", data = 100},
            {description = "150", data = 150},
            {description = "200", data = 200},
        },
        default = 150,
    },

    -- 空腹度の設定
    {
        name = "hunger",
        label = "Hunger",
        hover = "キャラクターの最大空腹度",
        options = {
            {description = "100", data = 100},
            {description = "150", data = 150},
            {description = "200", data = 200},
        },
        default = 150,
    },

    -- ========== 戦闘設定 ==========
    {
        name = "",
        label = "Combat Settings",
        hover = "",
        options = {{description = "", data = 0}},
        default = 0,
    },

    -- 攻撃力倍率
    {
        name = "damage_multiplier",
        label = "Damage Multiplier",
        hover = "攻撃力の倍率",
        options = {
            {description = "0.75x", data = 0.75},
            {description = "1.0x",  data = 1.0},
            {description = "1.5x",  data = 1.5},
            {description = "2.0x",  data = 2.0},
        },
        default = 1.0,
    },
}
```

> **ポイント**: セクション見出し用のエントリは `name = ""` にする以外に、`options` と `default` にもダミー値を設定します。この形式はお決まりのパターンなので、そのままコピペして `label` だけ変更してください。

---

## modmain.lua での設定値の取得

modinfo.lua でコンフィグオプションを定義したら、modmain.lua で設定値を取得して使います。

### 基本的な取得方法

```lua
-- ========================================
-- 設定値の取得
-- ========================================

-- GetModConfigData() で modinfo.lua の name に対応する値を取得
-- or の後ろはフォールバック値（設定が読み込めなかった場合の代替値）
local health = GetModConfigData("health") or 150
local hunger = GetModConfigData("hunger") or 150
local damage_mult = GetModConfigData("damage_multiplier") or 1.0
local speed_mult = GetModConfigData("speed_multiplier") or 1.0
```

> **なぜ `or` でフォールバック値を設定するのか？**: `GetModConfigData()` は設定が見つからない場合に `nil`（値が存在しない状態）を返します。`nil or 150` は `150` になるので、設定が読み込めない場合でもエラーにならずにデフォルト値が使われます。

### boolean 型の注意点

boolean（真偽値: true/false）型の設定値は、`or` パターンが使えません。

```lua
-- NG: boolean 型で or を使うと正しく動作しない
local enable_ability = GetModConfigData("enable_ability") or true
-- ↑ ユーザーが Off (false) を選んでも、false or true = true になってしまう！

-- OK: nil チェックで分岐する
local enable_ability = GetModConfigData("enable_ability")
if enable_ability == nil then
    enable_ability = true   -- デフォルト値
end
```

> **なぜ `or` が使えないのか？**: Lua では `false or true` は `true` になります。つまり、ユーザーが明示的に `false`（Off）を選んでも、常に `true` が返ってしまいます。`nil`（未設定）と `false`（Off を選択）を区別するために、`== nil` で判定する必要があります。

### GLOBAL テーブルに格納する

取得した設定値を prefab ファイルから参照できるように、GLOBAL テーブルに格納します。

```lua
-- GLOBAL に格納して、prefab ファイルから参照できるようにする
GLOBAL.MY_MOD_HEALTH = GetModConfigData("health") or 150
GLOBAL.MY_MOD_HUNGER = GetModConfigData("hunger") or 150
GLOBAL.MY_MOD_DAMAGE_MULT = GetModConfigData("damage_multiplier") or 1.0
GLOBAL.MY_MOD_SPEED_MULT = GetModConfigData("speed_multiplier") or 1.0

-- boolean 型は nil チェック
GLOBAL.MY_MOD_ENABLE_ABILITY = GetModConfigData("enable_ability")
if GLOBAL.MY_MOD_ENABLE_ABILITY == nil then
    GLOBAL.MY_MOD_ENABLE_ABILITY = true
end
```

> **なぜ GLOBAL に格納するのか？**: modmain.lua はサンドボックス環境で実行されるため、modmain.lua 内のローカル変数は prefab ファイルからアクセスできません。GLOBAL テーブルに格納することで、prefab ファイルから `GLOBAL.MY_MOD_HEALTH` のように参照できるようになります。

> **重要**: GLOBAL に格納する変数名は、他の MOD と衝突しないように **MOD 固有のプレフィックス**（`MY_MOD_` など）を付けてください。

---

## prefab ファイルでの設定値の使い方

modmain.lua で GLOBAL に格納した設定値を、キャラクターの prefab ファイル（`scripts/prefabs/my_character.lua`）で使う方法です。

### 基本ステータスへの反映

```lua
-- scripts/prefabs/my_character.lua

local master_postinit = function(inst)
    -- GLOBAL に格納した設定値を使う
    inst.components.health:SetMaxHealth(GLOBAL.MY_MOD_HEALTH)        -- 体力
    inst.components.hunger:SetMax(GLOBAL.MY_MOD_HUNGER)              -- 空腹度
    inst.components.sanity:SetMax(200)                                -- 正気度（固定値の例）

    -- 移動速度の反映
    inst.components.locomotor.walkspeed = 4 * GLOBAL.MY_MOD_SPEED_MULT   -- 歩行速度
    inst.components.locomotor.runspeed = 6 * GLOBAL.MY_MOD_SPEED_MULT    -- 走行速度

    -- 攻撃力倍率の反映
    inst.components.combat.damagemultiplier = GLOBAL.MY_MOD_DAMAGE_MULT
end
```

### 条件分岐（ON / OFF 系）

boolean 型の設定値で機能の有効・無効を切り替える例です。

```lua
local master_postinit = function(inst)
    -- 基本ステータスの設定（省略）

    -- 特殊能力の ON/OFF
    if GLOBAL.MY_MOD_ENABLE_ABILITY then
        -- 特殊能力を追加する処理
        inst:AddComponent("myability")
        inst.components.myability:SetPower(10)
    end
end
```

### 文字列型の条件分岐

```lua
local master_postinit = function(inst)
    -- 難易度によって初期アイテムを変える
    local difficulty = GLOBAL.MY_MOD_DIFFICULTY

    if difficulty == "easy" then
        -- Easy: 多めの初期アイテム
        inst.components.inventory:GiveItem(SpawnPrefab("axe"))
        inst.components.inventory:GiveItem(SpawnPrefab("torch"))
    elseif difficulty == "hard" then
        -- Hard: 初期アイテムなし
        -- 何もしない
    else
        -- Normal: 標準の初期アイテム
        inst.components.inventory:GiveItem(SpawnPrefab("flint"))
    end
end
```

---

## 実践例: キャラクター MOD の設定テンプレート

キャラクター MOD でよく使われる設定項目を一式まとめたテンプレートです。そのままコピペして使えます。

### modinfo.lua に追加するコード

```lua
-- ★ modinfo.lua の末尾に追加してください
-- ★ "my_character" の部分は自分のキャラクター名に合わせて変更してください

configuration_options = {
    -- ========== キャラクターステータス ==========
    {
        name = "",
        label = "Character Stats",
        hover = "",
        options = {{description = "", data = 0}},
        default = 0,
    },
    {
        name = "health",
        label = "Health",
        hover = "キャラクターの最大体力（デフォルト: 150）",
        options = {
            {description = "75",  data = 75},
            {description = "100", data = 100},
            {description = "150", data = 150},
            {description = "200", data = 200},
            {description = "250", data = 250},
            {description = "300", data = 300},
        },
        default = 150,
    },
    {
        name = "hunger",
        label = "Hunger",
        hover = "キャラクターの最大空腹度（デフォルト: 150）",
        options = {
            {description = "75",  data = 75},
            {description = "100", data = 100},
            {description = "150", data = 150},
            {description = "200", data = 200},
            {description = "250", data = 250},
        },
        default = 150,
    },
    {
        name = "sanity",
        label = "Sanity",
        hover = "キャラクターの最大正気度（デフォルト: 200）",
        options = {
            {description = "100", data = 100},
            {description = "150", data = 150},
            {description = "200", data = 200},
            {description = "250", data = 250},
            {description = "300", data = 300},
        },
        default = 200,
    },

    -- ========== 移動・戦闘 ==========
    {
        name = "",
        label = "Movement & Combat",
        hover = "",
        options = {{description = "", data = 0}},
        default = 0,
    },
    {
        name = "speed_multiplier",
        label = "Move Speed",
        hover = "移動速度の倍率（デフォルト: 1.0x）",
        options = {
            {description = "0.75x", data = 0.75},
            {description = "1.0x",  data = 1.0},
            {description = "1.25x", data = 1.25},
            {description = "1.5x",  data = 1.5},
        },
        default = 1.0,
    },
    {
        name = "damage_multiplier",
        label = "Damage",
        hover = "攻撃力の倍率（デフォルト: 1.0x）",
        options = {
            {description = "0.75x", data = 0.75},
            {description = "1.0x",  data = 1.0},
            {description = "1.5x",  data = 1.5},
            {description = "2.0x",  data = 2.0},
        },
        default = 1.0,
    },

    -- ========== その他 ==========
    {
        name = "",
        label = "Other",
        hover = "",
        options = {{description = "", data = 0}},
        default = 0,
    },
    {
        name = "enable_ability",
        label = "Special Ability",
        hover = "特殊能力の有効/無効",
        options = {
            {description = "On",  data = true},
            {description = "Off", data = false},
        },
        default = true,
    },
}
```

### modmain.lua に追加するコード

```lua
-- ========================================
-- 設定値の取得（modmain.lua の先頭付近に追加）
-- ========================================
GLOBAL.MY_MOD_HEALTH = GetModConfigData("health") or 150
GLOBAL.MY_MOD_HUNGER = GetModConfigData("hunger") or 150
GLOBAL.MY_MOD_SANITY = GetModConfigData("sanity") or 200
GLOBAL.MY_MOD_SPEED_MULT = GetModConfigData("speed_multiplier") or 1.0
GLOBAL.MY_MOD_DAMAGE_MULT = GetModConfigData("damage_multiplier") or 1.0

GLOBAL.MY_MOD_ENABLE_ABILITY = GetModConfigData("enable_ability")
if GLOBAL.MY_MOD_ENABLE_ABILITY == nil then
    GLOBAL.MY_MOD_ENABLE_ABILITY = true
end
```

### prefab ファイルでの使用

```lua
-- scripts/prefabs/my_character.lua 内の master_postinit

local master_postinit = function(inst)
    -- コンフィグオプションの値を反映
    inst.components.health:SetMaxHealth(GLOBAL.MY_MOD_HEALTH)
    inst.components.hunger:SetMax(GLOBAL.MY_MOD_HUNGER)
    inst.components.sanity:SetMax(GLOBAL.MY_MOD_SANITY)

    inst.components.locomotor.walkspeed = 4 * GLOBAL.MY_MOD_SPEED_MULT
    inst.components.locomotor.runspeed = 6 * GLOBAL.MY_MOD_SPEED_MULT

    inst.components.combat.damagemultiplier = GLOBAL.MY_MOD_DAMAGE_MULT

    if GLOBAL.MY_MOD_ENABLE_ABILITY then
        -- 特殊能力を有効にする処理をここに書く
    end
end
```

---

## よくあるトラブルと解決法

### 設定画面に関するトラブル

| 症状 | 原因 | 解決法 |
|---|---|---|
| 設定画面に項目が表示されない | `configuration_options` の文法エラー | カンマ抜け、括弧の対応を確認する |
| 設定画面に項目が表示されない | `modinfo.lua` 自体が読み込めていない | `modinfo.lua` の他の項目（name 等）に文法エラーがないか確認する |
| デフォルト値が選択されていない | `default` の値が `options` の `data` と不一致 | `default` の値を `options` 内のいずれかの `data` と完全一致させる |

### 設定値が反映されないトラブル

| 症状 | 原因 | 解決法 |
|---|---|---|
| 設定を変えても値が変わらない | `GetModConfigData` の引数と `name` が不一致 | modinfo.lua の `name` と `GetModConfigData("name")` の文字列を確認 |
| boolean の Off が効かない | `or` パターンで取得している | `nil` チェック方式に修正する（[boolean 型の注意点](#boolean-型の注意点) 参照） |
| prefab から値を参照できない | GLOBAL に格納していない | modmain.lua で `GLOBAL.MY_MOD_XXX = ...` の形で格納する |

### よくある文法エラー

#### カンマの抜け

```lua
-- NG: 設定項目間のカンマが抜けている
configuration_options = {
    {
        name = "health",
        label = "Health",
        -- ...
    }                    -- ← ここにカンマがない！
    {
        name = "hunger",
        -- ...
    },
}

-- OK
configuration_options = {
    {
        name = "health",
        label = "Health",
        -- ...
    },                   -- ← カンマを忘れずに
    {
        name = "hunger",
        -- ...
    },
}
```

#### options テーブルの構造ミス

```lua
-- NG: options 内の要素に {} が足りない
options = {
    description = "100", data = 100,    -- ← {} で囲む必要がある
},

-- OK
options = {
    {description = "100", data = 100},  -- ← 各要素を {} で囲む
    {description = "150", data = 150},
},
```

> **ヒント**: 文法エラーが見つからない場合は、`modinfo.lua` のコード全体を [Lua syntax checker](https://www.lua.org/cgi-bin/demo) 等のオンラインツールに貼り付けてチェックしてみてください。

---

## Tips・参考情報

### 設定変更の反映タイミング

コンフィグオプションの値は **ワールド作成時** に読み込まれます。既に作成済みのワールドで設定を変更した場合、新しいワールドを作成するか、ゲームを再起動する必要がある場合があります。

### label と description の言語について

`label`（ラベル）と `description`（選択肢の表示名）は、ゲーム内のフォントで表示されます。

- **英語**: 確実に表示されます
- **日本語**: フォントによっては表示されない場合があります

Steam Workshop に公開する場合は、**英語** で記述することをおすすめします。

### hover を活用する

`hover` フィールドはマウスオーバー時に表示される説明文です。プレイヤーが設定の意味を理解しやすいように、具体的な説明を書きましょう。

```lua
-- 良い例
hover = "キャラクターの最大体力（ウィルソンのデフォルト値: 150）",

-- 悪い例
hover = "Health",  -- label と同じでは意味がない
```

### 設定項目数の目安

- **3〜5 項目**: ちょうど良い。プレイヤーが迷わずに設定できる
- **6〜10 項目**: セクション見出しで整理すれば問題ない
- **10 項目以上**: 多すぎる可能性あり。本当に必要な項目か検討する

> **初心者向けTip:** 最初は少ない項目から始めて、プレイヤーの要望に応じて追加していくのがおすすめです。

### このガイドで扱わないもの

| 内容 | 参照先ガイド |
|---|---|
| modinfo.lua の基本項目（name, version 等） | [DST-Lua-Scripting ガイド](README.md) |
| キャラクターのセリフ設定 | [Speech ファイル作成ガイド](README-speech.md) |
| キャラクター prefab の書き方全般 | [DST-Lua-Scripting ガイド](README.md) |
| Steam Workshop への公開 | [DST-Workshop-Upload ガイド](../DST-Workshop-Upload/README.md) |

---

## 参考リンク

- [Don't Starve Wiki — Modding](https://dontstarve.wiki.gg/wiki/Modding) — 公式 Wiki の MOD セクション
- [Klei Forums — Mods and Tools](https://forums.kleientertainment.com/forums/forum/79-modding-tools-tutorials/) — 公式フォーラム
- [← DST-Lua-Scripting ガイドに戻る](README.md)
- [← メインガイドに戻る](../README.md)

---

## ライセンス

このドキュメントは自由に利用・改変・再配布できます。DST MOD 制作にお役立てください。

---

<sub>Author: [pinpikokun](https://steamcommunity.com/profiles/76561198076111536/)</sub><br>
<sub>[![GitHub](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/pinpikokun/DST-teemo) [![Steam Workshop](https://img.shields.io/badge/Steam%20Workshop-Subscribe-blue?logo=steam)](https://steamcommunity.com/sharedfiles/filedetails/?id=390684095)</sub>
