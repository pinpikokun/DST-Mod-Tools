# DST-Lua-Scripting 実践編: Captain Teemo MOD を読み解く

Captain Teemo MOD は、League of Legends のキャラクター「テーモ」を DST に再現した MOD です。ステルス、毒吹き矢、キノコ罠、サモナースペルなど、多彩な機能を実装しています。

本ガイドでは、この MOD の実際のソースコードを読み解きながら、[Lua スクリプティングガイド](README.md) で学んだ概念がどのように実装されるかを実践的に解説します。

> **HTML 版**: [README-DST-teemo.html](README-DST-teemo.html)

---

## 目次

- [DST-teemo モッドについて](#dst-teemo-モッドについて)
- [Captain Teemo MOD の概要](#captain-teemo-mod-の概要)
- [ファイル構成](#ファイル構成)
- [modinfo.lua の実装](#modinfolua-の実装)
- [modmain.lua の実装](#modmainlua-の実装)
- [キャラクター prefab — teemo.lua](#キャラクター-prefab--teemolua)
- [カスタムアイテムの実装](#カスタムアイテムの実装)
- [カスタムコンポーネントの実装](#カスタムコンポーネントの実装)
- [ユーティリティ — teemo_poison_util.lua](#ユーティリティ--teemo_poison_utillua)
- [ネットワーク同期の実装](#ネットワーク同期の実装)
- [UI ウィジェットの実装](#ui-ウィジェットの実装)
- [セーブ・ロードの実装](#セーブロードの実装)
- [能力一覧まとめ](#能力一覧まとめ)
- [参考リンク](#参考リンク)
- [ライセンス](#ライセンス)

---

## DST-teemo モッドについて

[![Steam Workshop](https://img.shields.io/badge/Steam%20Workshop-Subscribe-blue?logo=steam)](https://steamcommunity.com/sharedfiles/filedetails/?id=390684095) [![GitHub](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/pinpikokun/DST-teemo)

ゲーム内画像は CLIP STUDIO PAINT で作成しており、キャラクターボイスはすべて生成 AI で作成しています。

---

## Captain Teemo MOD の概要

Captain Teemo は以下の能力を持つキャラクター MOD です。

| 能力 | 種類 | 概要 |
|---|---|---|
| カモフラージュ | パッシブ | 3 秒間静止すると透明化。敵の攻撃を無効化 |
| Blind Dart | 武器 | 右クリックで毒吹き矢を発射。命中時にブラインド + 毒 DOT |
| Noxious Trap | 設置型 | 毒キノコ罠。敵が近づくと爆発し範囲ダメージ + 移動速度低下 |
| Flash | サモナースペル | 瞬間移動（壁抜け対応） |
| Ignite | サモナースペル | 対象にトゥルーダメージ DOT |
| キノコの達人 | パッシブ | キノコのマイナス効果を無効化 |
| 毒による食料腐敗 | パッシブ | 毒で倒した敵のドロップ食料が腐敗 |

> **ポイント**: この MOD はカスタムアクション、Net Variables、RPC、UI ウィジェット、コンポーネント、ステートグラフ等を活用しており、実践的な学習素材として利用可能です。

---

## ファイル構成

```
DST-teemo/
├── modinfo.lua                          ← MOD 設定（20 個の設定オプション）
├── modmain.lua                          ← エントリポイント（設定読込・キャラクター登録・アクション定義・RPC）
├── modicon.xml / .tex                   ← MOD アイコン
├── anim/                                ← アニメーションファイル
├── bigportraits/                        ← 大ポートレート（512 x 512 以上）
│   ├── teemo.tex / .xml
│   └── teemo_none.tex / .xml
├── images/
│   ├── avatars/                         ← アバター画像（64 x 64）
│   ├── hud/                             ← HUD 用画像（クラフトタブアイコン）
│   ├── inventoryimages/                 ← インベントリアイテム画像
│   ├── map_icons/                       ← マップアイコン（64 x 64）
│   ├── saveslot_portraits/              ← セーブスロット画像（128 x 128）
│   └── selectscreen_portraits/          ← キャラクター選択画面（256 x 512）
├── scripts/
│   ├── prefabs/
│   │   ├── teemo.lua                    ← キャラクター本体（カモフラージュ、ステータス、イベント）
│   │   ├── teemo_none.lua               ← スキン定義
│   │   ├── blind_dart.lua               ← Blind Dart（武器アイテム）
│   │   ├── blind_dart_projectile.lua    ← 吹き矢の飛翔体（命中処理・ブラインド・DOT）
│   │   ├── noxious_trap.lua             ← Noxious Trap（設置型罠）
│   │   ├── explode_noxious_trap.lua     ← 罠爆発エフェクト
│   │   ├── toxic_effect_by_teemo.lua    ← 毒エフェクト
│   │   └── blind_effect.lua             ← ブラインドエフェクト
│   ├── components/
│   │   ├── characterspecific.lua        ← キャラクター専用アイテム制限
│   │   └── explosive_noxious_trap.lua   ← 罠爆発ダメージロジック
│   ├── widgets/
│   │   ├── noxioustrap_slot.lua         ← トラップスタック UI
│   │   ├── summoner_spell_slot.lua      ← Flash/Ignite UI（ターゲティング機能付き）
│   │   └── teemo_popupnumber.lua        ← フローティングダメージ数字
│   ├── speech_teemo.lua                 ← キャラクターセリフ
│   └── teemo_poison_util.lua            ← 毒ユーティリティ（食料腐敗システム）
└── sound/                               ← サウンドファイル
```

---

## modinfo.lua の実装

teemo の modinfo.lua は **20 個の設定オプション** を持つ、高度にカスタマイズ可能な設計です。

### セクション区切りの活用

設定項目が多い MOD では、`name` を空文字にしたエントリでカテゴリを区切っています。

```lua
-- セクション見出し（設定画面にラベルとして表示される）
{name = "", label = "Character Stats", hover = "", options = {{description = "", data = 0}}, default = 0},
{name = "", label = "Blind Dart", hover = "", options = {{description = "", data = 0}}, default = 0},
{name = "", label = "Noxious Trap", hover = "", options = {{description = "", data = 0}}, default = 0},
{name = "", label = "Summoner Spells", hover = "", options = {{description = "", data = 0}}, default = 0},
{name = "", label = "Other", hover = "", options = {{description = "", data = 0}}, default = 0},
```

### 設定カテゴリ一覧

| カテゴリ | 設定項目 |
|---|---|
| Character Stats | Health, Hunger, Sanity, Move Speed, Damage Multiplier, Defense |
| Blind Dart | Hit Damage, DOT Damage, Durability, Range Indicator |
| Noxious Trap | Hit Damage, DOT Damage |
| Summoner Spells | Flash Range, Flash Cooldown, Ignite Damage, Ignite Cooldown |
| Other | Poison Food Spoilage, Mushroom Immunity, Damage Numbers |

### 数値型とブーリアン型の混在

```lua
-- 数値型: options の data に数値を設定
{
    name = "health",
    label = "Health",
    options = {
        {description = "50",  data = 50},
        {description = "100", data = 100},   -- そのまま
        {description = "200", data = 200},
    },
    default = 100,
},

-- ブーリアン型: On/Off の切り替え
{
    name = "mushroom_immunity",
    label = "Mushroom Immunity",
    options = {
        {description = "On",  data = true},
        {description = "Off", data = false},
    },
    default = true,
},
```

---

## modmain.lua の実装

teemo の modmain.lua は約 800 行あり、以下の機能を担当しています。

### 1. 設定値の読み込み

```lua
-- 数値型は or でデフォルト値を設定
GLOBAL.TEEMO_HEALTH = GetModConfigData("health") or 100
GLOBAL.TEEMO_DAMAGE_MULT = GetModConfigData("damage_multiplier") or 1.0

-- boolean 型は nil チェックが必要（false or true は true になってしまう）
GLOBAL.TEEMO_MUSHROOM_IMMUNITY = GetModConfigData("mushroom_immunity")
if GLOBAL.TEEMO_MUSHROOM_IMMUNITY == nil then GLOBAL.TEEMO_MUSHROOM_IMMUNITY = true end
```

> **なぜ GLOBAL に格納するのか？**: modmain.lua のローカル変数は prefab ファイル（teemo.lua 等）からアクセスできません。`GLOBAL` に格納することで、全スクリプトから参照可能になります。

### 2. ダメージ数字表示のヘルパー関数

サーバーから範囲内の全クライアントにダメージ数字を表示させるヘルパーです。

```lua
local function TeemoShowDamageNumber(target, damage, colour_type)
    if not GLOBAL.TEEMO_SHOW_DAMAGE_NUMBERS then return end
    if not GLOBAL.TheWorld.ismastersim then return end
    if target == nil or not target:IsValid() then return end

    local x, y, z = target.Transform:GetWorldPosition()
    local players = GLOBAL.FindPlayersInRange(x, y, z, 40)
    for _, player in pairs(players) do
        GLOBAL.SendModRPCToClient(
            GLOBAL.CLIENT_MOD_RPC["teemo"]["show_damage_number"],
            player.userid, x, y, z, math.abs(damage), colour_type
        )
    end
end
GLOBAL.TeemoShowDamageNumber = TeemoShowDamageNumber
```

> **ポイント**: `FindPlayersInRange` で 40 タイルの範囲内のプレイヤーにのみ送信しています。全プレイヤーに送ると帯域を無駄に消費するため、見える範囲のプレイヤーに限定しています。

### 3. キャラクター登録

```lua
-- TUNING テーブルに登録（キャラクター選択画面で表示される）
GLOBAL.TUNING.TEEMO_HEALTH = GLOBAL.TEEMO_HEALTH
GLOBAL.TUNING.TEEMO_HUNGER = GLOBAL.TEEMO_HUNGER
GLOBAL.TUNING.TEEMO_SANITY = GLOBAL.TEEMO_SANITY

-- 初期所持アイテム
GLOBAL.TUNING.GAMEMODE_STARTING_ITEMS.DEFAULT.TEEMO = {"blind_dart", "noxious_trap"}

-- キャラクター表示テキスト
STRINGS.CHARACTER_TITLES.teemo = "Captain Teemo"
STRINGS.CHARACTER_NAMES.teemo = "Captain Teemo"
STRINGS.CHARACTER_DESCRIPTIONS.teemo =
    "*Goes invisible when standing still\n*Has a poison blowdart\n..."
STRINGS.CHARACTER_QUOTES.teemo = "\"on duty !! \""

-- セリフファイル
STRINGS.CHARACTERS.TEEMO = GLOBAL.require "speech_teemo"

-- 登録
AddModCharacter("teemo", "MALE", skin_modes)
AddMinimapAtlas("images/map_icons/teemo.xml")
GLOBAL.PREFAB_SKINS["teemo"] = { "teemo_none" }
```

### 4. カスタムアクション — 右クリック発射

Blind Dart の右クリック発射システムは、DST の MOD でカスタムアクションを作る典型的な実装例です。

```lua
-- アクション定義: 右クリックで Blind Dart 発射
AddAction("TEEMO_SHOOT_DART", "Auto Attack", function(act)
    local weapon = act.invobject   -- 使用武器
    local doer = act.doer          -- プレイヤー
    local target = act.target      -- 右クリック対象（いれば）
    local target_pos = act:GetActionPoint()  -- クリック座標

    -- 飛翔体をスポーン
    local proj = GLOBAL.SpawnPrefab("blind_dart_projectile")
    proj._teemo_weapon = weapon    -- onhit で参照するための情報を格納
    proj._teemo_attacker = doer

    if target ~= nil and target:IsValid() and target.components.combat then
        -- エンティティを右クリック → 追尾発射
        proj.components.projectile:Throw(weapon, target, doer)
    else
        -- 地面を右クリック → クリック地点近くの敵を自動検索
        local px, py, pz = target_pos:Get()
        local ents = GLOBAL.TheSim:FindEntities(px, py, pz, 3, {"_combat"}, {"player", "INLIMBO"})
        -- 最も近い敵を探して発射...
    end
    return true
end)

-- アクションのプロパティ
GLOBAL.ACTIONS.TEEMO_SHOOT_DART.rmb = true       -- 右クリック
GLOBAL.ACTIONS.TEEMO_SHOOT_DART.priority = -1    -- 低優先度
GLOBAL.ACTIONS.TEEMO_SHOOT_DART.distance = 5     -- 射程
```

### 5. ComponentAction — 発動条件

```lua
-- 地面を右クリック（POINT）
AddComponentAction("POINT", "weapon", function(inst, doer, pos, actions, right, target)
    if right and inst:HasTag("blowdart") and doer:HasTag("teemo")
        and not (doer.replica.rider and doer.replica.rider:IsRiding()) then
        table.insert(actions, GLOBAL.ACTIONS.TEEMO_SHOOT_DART)
    end
end)

-- エンティティを右クリック（EQUIPPED）
AddComponentAction("EQUIPPED", "weapon", function(inst, doer, target, actions, right)
    if right and inst:HasTag("blowdart") and doer:HasTag("teemo")
        and not (doer.replica.rider and doer.replica.rider:IsRiding())
        and target and target ~= doer and target:HasTag("_combat") then
        table.insert(actions, GLOBAL.ACTIONS.TEEMO_SHOOT_DART)
    end
end)
```

> **ポイント**: `doer.replica.rider` で騎乗チェックしています。`replica` はクライアント側でアクセス可能なプロキシオブジェクトです。ComponentAction はクライアント側で実行されるため、`components.rider` ではなく `replica.rider` を使います。

### 6. ステートグラフ — 攻撃アニメーション

```lua
-- サーバー側ステート
AddStategraphState("wilson", GLOBAL.State {
    name = "teemo_shoot_dart",
    tags = { "attack", "abouttoattack", "notalking", "autopredict" },

    onenter = function(inst)
        if inst.components.combat:InCooldown() then
            inst:ClearBufferedAction()
            inst.sg:GoToState("idle", true)
            return
        end
        inst.components.combat:StartAttack()
        inst.components.locomotor:Stop()
        inst.AnimState:PlayAnimation("dart")
        inst.sg:SetTimeout(inst.components.combat.min_attack_period)
    end,

    timeline = {
        -- 6 フレーム目でアクション実行（矢を発射）
        GLOBAL.TimeEvent(6 * GLOBAL.FRAMES, function(inst)
            inst.sg:RemoveStateTag("abouttoattack")
            inst.bufferedaction = inst.sg.statemem.action
            inst:PerformBufferedAction()
            inst.SoundEmitter:PlaySound("dontstarve/wilson/blowdart_shoot")
        end),
    },

    events = {
        GLOBAL.EventHandler("animqueueover", function(inst)
            if inst.AnimState:AnimDone() then inst.sg:GoToState("idle") end
        end),
    },
})
```

> **なぜ `abouttoattack` タグがあるのか？**: このタグは「まさに攻撃しようとしている」状態を示します。6 フレーム目で実際にアクションが実行された時に `RemoveStateTag` で外し、`onexit` で残っていれば `CancelAttack()` します。これにより、攻撃中断時の整合性が保たれます。

### 7. RPC — サモナースペル

#### Flash（テレポート）

```lua
AddModRPCHandler("teemo", "use_flash", function(player, x, z)
    -- バリデーション
    if not player:HasTag("teemo") then return end
    if player:HasTag("playerghost") then return end
    if player._flashCooldown == nil or player._flashCooldown:value() > 0 then return end

    -- 範囲チェック（最大距離にクランプ）
    local px, py, pz = player.Transform:GetWorldPosition()
    local dx, dz = x - px, z - pz
    local dist = math.sqrt(dx * dx + dz * dz)
    local maxRange = GLOBAL.TEEMO_FLASH_RANGE
    if dist > maxRange then
        local ratio = maxRange / dist
        x = px + dx * ratio
        z = pz + dz * ratio
    end

    -- 壁抜け判定（LoL 準拠）
    if not GLOBAL.TheWorld.Map:IsPassableAtPoint(x, 0, z) then
        -- 1. 目標地点の先に歩行可能地点があるか探す
        -- 2. なければ手前にフォールバック
        -- ...
    end

    -- クールダウン開始（連打防止のため処理の最初に設定）
    player._flashCooldown:set(GLOBAL.TEEMO_FLASH_COOLDOWN)

    -- テレポート実行
    player.Physics:Teleport(x, 0, z)
end)
```

> **ポイント**: 壁抜け判定が実装されています。テレポート先が壁の中の場合、同じ方向にさらに 3 ユニット先まで歩行可能地点を探し、見つからなければ手前にフォールバックします。LoL の Flash と同じ挙動を再現しています。

#### Ignite（炎上 DOT）

```lua
AddModRPCHandler("teemo", "use_ignite", function(player)
    -- バリデーション（省略）

    -- 最も近い敵対エンティティを単体選択
    local target = nil
    local closestDist = math.huge
    local ents = GLOBAL.TheSim:FindEntities(x, y, z, range, {"_combat"})
    for _, v in pairs(ents) do
        if v ~= player and v.components.combat
            and not v:HasTag(nonTarget) and not v:HasTag("companion")
            and (v:HasTag("hostile") or v.components.combat.target == player) then
            -- 距離比較で最も近い敵を選択
        end
    end

    -- 敵がいなければ発動しない（CD を消費しない）
    if target == nil then return end

    -- クールダウン開始
    player._igniteCooldown:set(GLOBAL.TEEMO_IGNITE_COOLDOWN)

    -- トゥルーダメージ DOT（DoDelta で防御無視）
    local dmg = GLOBAL.TEEMO_IGNITE_DAMAGE
    if v:HasTag("player") then dmg = dmg * 0.3 end  -- 対人ダメージ 30%

    v._igniteDotTask = v:DoPeriodicTask(1.0, function()
        v.components.health:DoDelta(-dmg, nil, "ignite")
        TeemoShowDamageNumber(v, dmg, TEEMO_DMG_COLOUR_TRUE)
    end)

    -- 5 秒後に DOT 停止
    v._igniteDotEndTask = v:DoTaskInTime(5.05, function()
        if v._igniteDotTask ~= nil then
            v._igniteDotTask:Cancel()
            v._igniteDotTask = nil
        end
    end)
end)
```

> **なぜ `5.05` 秒なのか？**: `DoPeriodicTask(1.0, ...)` は 1 秒ごとにダメージを与えます。5 秒だと 5 回目のティックと停止が同時になり、実行順序が不確定になります。0.05 秒の余裕を持たせることで、5 回目のダメージが確実に適用されてから停止します。

### 8. AddPrefabPostInit — 月キノコの免疫

```lua
if GLOBAL.TEEMO_MUSHROOM_IMMUNITY then
    AddPrefabPostInit("moon_cap", function(inst)
        if not GLOBAL.TheWorld.ismastersim then return end
        local _oneaten = inst.components.edible.oneaten  -- 元の関数を保存
        inst.components.edible:SetOnEatenFn(function(inst, eater)
            if eater:HasTag("teemo") then return end     -- teemo なら効果をスキップ
            if _oneaten ~= nil then _oneaten(inst, eater) end  -- それ以外は元の処理
        end)
    end)
end
```

### 9. クラフトタブとレシピ

```lua
-- テーモ専用クラフトタブ（"teemo" タグを持つキャラクターのみ表示）
local teemoTab = AddRecipeTab(
    "Teemo Items", 998,
    GLOBAL.resolvefilepath("images/hud/teemotab.xml"), "teemotab.tex",
    "teemo"
)

-- Blind Dart のレシピ
AddRecipe2("blind_dart", {
    GLOBAL.Ingredient("boards", 1),
    GLOBAL.Ingredient("green_cap", 1),
    GLOBAL.Ingredient("silk", 1),
    GLOBAL.Ingredient("stinger", 1),
    GLOBAL.Ingredient("rope", 1),
}, GLOBAL.TECH.NONE, {
    atlas = GLOBAL.resolvefilepath("images/inventoryimages/blind_dart.xml"),
    image = "blind_dart.tex",
    builder_tag = "teemo",   -- teemo のみクラフト可能
    tab = teemoTab,
}, {"CHARACTER"})
```

### 10. インベントリバーのカスタマイズ

最後の 3 スロットを専用 UI（トラップ・フラッシュ・イグナイト）に置き換えています。

```lua
AddClassPostConstruct("widgets/inventorybar", function(self)
    if not self.owner:HasTag("teemo") then return end

    local NoxiousTrapSlot = require("widgets/noxioustrap_slot")
    local SummonerSpellSlot = require("widgets/summoner_spell_slot")

    local _Rebuild = self.Rebuild
    self.Rebuild = function(self, ...)
        -- 既存のカスタムスロットを破棄
        if self.noxioustrapslot ~= nil then self.noxioustrapslot:Kill() end
        if self.flashslot ~= nil then self.flashslot:Kill() end
        if self.igniteslot ~= nil then self.igniteslot:Kill() end

        _Rebuild(self, ...)  -- 元の Rebuild を実行

        -- 最後の 3 スロットを非表示にする
        local numSlots = #self.inv
        for i = numSlots - GLOBAL.TEEMO_RESERVED_SLOTS + 1, numSlots do
            local slot = self.inv[i]
            if slot then
                slot:Hide()
                slot.OnControl = function() return false end
            end
        end

        -- カスタムウィジェットを配置
        local trapSlot = self.inv[numSlots - 2]
        if trapSlot then
            local pos = trapSlot:GetPosition()
            self.noxioustrapslot = self.toprow:AddChild(NoxiousTrapSlot(self.owner))
            self.noxioustrapslot:SetPosition(pos.x, pos.y, pos.z)
        end
        -- Flash, Ignite も同様に配置...
    end

    self.rebuild_pending = true
end)
```

> **なぜ `rebuild_pending` を設定するのか？**: `Rebuild` メソッドをオーバーライドしただけでは、既に構築済みのインベントリバーには反映されません。`rebuild_pending = true` を設定することで、次フレームで再構築が実行されます。

---

## キャラクター prefab — teemo.lua

teemo.lua はキャラクターの中核ファイルで、パッシブ能力「カモフラージュ」やステータス設定を実装しています。

### パッシブ能力: カモフラージュ（透明化）

3 秒間静止するとステルス状態になる能力です。

#### ステルス発動

```lua
local function doCamouflage(inst)
    if not inst.isCamouflage then
        inst.isCamouflage = true
        -- 半透明化
        inst.AnimState:SetMultColour(.5, .5, .5, .8)
        -- 影を非表示
        inst.DynamicShadow:Enable(false)

        -- 砂煙エフェクト
        local puff = SpawnPrefab("shadow_puff")
        if puff ~= nil then
            puff.Transform:SetPosition(inst.Transform:GetWorldPosition())
        end

        -- セリフ（30% 確率）
        if math.random() < 0.3 and inst.components.talker then
            inst.components.talker:Say(CAMOUFLAGE_QUOTES[math.random(#CAMOUFLAGE_QUOTES)])
        end

        -- 当たり判定を変更（敵をすり抜ける）
        inst.Physics:ClearCollisionMask()
        inst.Physics:CollidesWith(COLLISION.WORLD)
        inst.Physics:CollidesWith(COLLISION.OBSTACLES)
        inst.Physics:CollidesWith(COLLISION.SMALLOBSTACLES)
        -- COLLISION.CHARACTERS と COLLISION.GIANTS を除外 → すり抜け

        -- 敵の攻撃を無効化（0.5 秒ごと）
        inst._blankOutTask = inst:DoPeriodicTask(.5, function()
            local x, y, z = inst.Transform:GetWorldPosition()
            local ents = TheSim:FindEntities(x, y, z, 20, {"_combat"})
            for k, v in pairs(ents) do
                if v.components.combat and v.components.combat.target == inst then
                    v.components.combat:BlankOutAttacks(1)
                end
            end
        end)
    end
end
```

> **なぜ `BlankOutAttacks` を使うのか？**: 単にターゲットを外すと敵が別のプレイヤーに向かってしまいます。`BlankOutAttacks` はターゲットを維持したまま攻撃を空振りにするため、ステルス解除後に即座に戦闘が再開されます。

#### ステルス定期チェック

```lua
local function checkCamouflage(inst)
    if inst.components.health == nil then return end
    if inst.components.locomotor == nil then return end

    -- 騎乗中は無効
    if inst.components.rider ~= nil and inst.components.rider:IsRiding() then
        disableCamouflage(inst)
        return
    end

    -- 被ダメージチェック
    if inst.components.health.currenthealth < inst.camouflage_h then
        disableCamouflage(inst)
        return
    end

    -- 位置チェック: 3 秒間動いていなければ発動
    local x, _, z = inst.Transform:GetWorldPosition()
    if x == inst.camouflage_x and z == inst.camouflage_z then
        if GetTime() - inst.camouflage_t > 3 then doCamouflage(inst) end
    else
        disableCamouflage(inst)
    end
end

-- master_postinit 内で 0.5 秒ごとにチェック
inst.camouflageTask = inst:DoPeriodicTask(.5, checkCamouflage)
```

#### ステルス解除ボーナス

```lua
local function disableCamouflage(inst)
    -- ...（透明化解除処理）...

    -- ステルス解除ボーナス: Blind Dart 装備時は攻撃速度 UP（5 秒間）
    if inst:HasTag("blind_dart_equipped") then
        inst.components.combat.min_attack_period = 0.5   -- 通常 1.0 → 0.5 秒
        if inst.resetAttackSpeedTask ~= nil then
            inst.resetAttackSpeedTask:Cancel()
        end
        inst.resetAttackSpeedTask = inst:DoTaskInTime(5.0, function(inst)
            if inst:HasTag("blind_dart_equipped") then
                inst.components.combat.min_attack_period = 1.0
            end
        end, inst)
    end
end
```

### ステータス設定

```lua
local function master_postinit(inst)
    inst.components.health:SetMaxHealth(TEEMO_HEALTH)
    inst.components.hunger:SetMax(TEEMO_HUNGER)
    inst.components.sanity:SetMax(TEEMO_SANITY)
    inst.components.combat.damagemultiplier = TEEMO_DAMAGE_MULT
    inst.components.health:SetAbsorptionAmount(TEEMO_ABSORPTION)

    -- キノコのマイナス効果無効化
    if TEEMO_MUSHROOM_IMMUNITY then
        inst.components.eater.custom_stats_mod_fn = mushroomStatsMod
    end
end
```

`custom_stats_mod_fn` はキャラクターの食事効果を加工するコールバックです。

```lua
local function mushroomStatsMod(inst, health_delta, hunger_delta, sanity_delta, food, feeder)
    if food ~= nil and MUSHROOM_PREFABS[food.prefab] then
        if health_delta < 0 then health_delta = 0 end   -- マイナス HP を 0 に
        if hunger_delta < 0 then hunger_delta = 0 end
        if sanity_delta < 0 then sanity_delta = 0 end
    end
    return health_delta, hunger_delta, sanity_delta
end
```

### 豊富なイベントリスナー

teemo.lua では多数のイベントリスナーでカモフラージュを管理しています。

```lua
-- これらのイベントすべてでカモフラージュ解除
inst:ListenForEvent("buildsuccess", function() disableCamouflage(inst) end)
inst:ListenForEvent("equipped", function() disableCamouflage(inst) end)
inst:ListenForEvent("onpickup", function() disableCamouflage(inst) end)
inst:ListenForEvent("ondropped", function() disableCamouflage(inst) end)
inst:ListenForEvent("oneatsomething", function() disableCamouflage(inst) end)
inst:ListenForEvent("oneaten", function() disableCamouflage(inst) end)

-- 作業時はカモフラージュ解除 + ボイス再生
inst:ListenForEvent("working", function()
    disableCamouflage(inst)
    if math.random() < 0.25 then inst._sound_emote:push() end
end)

-- 攻撃時はカモフラージュ解除 + 毒マーク設定
inst:ListenForEvent("onattackother", function(inst, data)
    disableCamouflage(inst)
    if data and data.weapon and data.weapon:HasTag("blowdart") and data.target then
        TeemoPoison.markTeemoPoisoned(data.target)
    end
end)
```

### インベントリスロット制限

最後の 3 スロットを専用スロットとして確保します。

```lua
local _CanTakeItemInSlot = inst.components.inventory.CanTakeItemInSlot
rawset(inst.components.inventory, "CanTakeItemInSlot", function(self, item, slot)
    if slot > self.maxslots - TEEMO_RESERVED_SLOTS then return false end
    return _CanTakeItemInSlot(self, item, slot)
end)
```

> **なぜ `rawset` を使うのか？**: DST の `inventory` コンポーネントはメタテーブルでメソッドを定義しています。単純に `self.CanTakeItemInSlot = ...` と書くとメタテーブルではなくインスタンスに書き込まれ、意図した動作にならない場合があります。`rawset` で確実にインスタンスレベルで上書きします。

---

## カスタムアイテムの実装

### Blind Dart（吹き矢武器）

`blind_dart.lua` は装備可能な武器アイテムです。

```lua
-- 左クリックのダメージは 0（右クリックの projectile で発射する設計）
inst:AddComponent("weapon")
inst.components.weapon:SetDamage(0)

-- 耐久力: 敵に攻撃されると減少（設定値で ON/OFF 可能）
if TEEMO_BLIND_DART_DURABILITY > 0 then
    inst:AddComponent("finiteuses")
    inst.components.finiteuses:SetMaxUses(TEEMO_BLIND_DART_DURABILITY)
    inst.components.finiteuses:SetUses(TEEMO_BLIND_DART_DURABILITY)
    -- 攻撃時の自動消費を無効化（被ダメージ時のみ手動で減少）
    inst.components.finiteuses:SetIgnoreCombatDurabilityLoss(true)
end

-- teemo のみ装備可能
inst:AddComponent("characterspecific")
inst.components.characterspecific:SetOwner("teemo")
```

装備コールバックでは、被攻撃時に耐久力を減らすリスナーを登録しています。

```lua
local function onequip(inst, owner)
    owner.AnimState:OverrideSymbol("swap_object", "swap_blind_dart", "swap_blind_dart")
    owner.AnimState:Show("ARM_carry")
    owner.AnimState:Hide("ARM_normal")
    owner:AddTag("blind_dart_equipped")

    -- 装備中に攻撃されたら耐久力を減らす
    if inst.components.finiteuses then
        inst._onowner_attacked = function(owner, data)
            if data and data.attacker and inst.components.finiteuses then
                local cur = inst.components.finiteuses:GetUses()
                if cur > 0 then inst.components.finiteuses:SetUses(cur - 1) end
            end
        end
        owner:ListenForEvent("attacked", inst._onowner_attacked)
    end
end

local function onunequip(inst, owner)
    -- リスナー解除（メモリリーク防止）
    if inst._onowner_attacked then
        owner:RemoveEventCallback("attacked", inst._onowner_attacked)
        inst._onowner_attacked = nil
    end
end
```

> **重要**: `onunequip` で必ずリスナーを解除しています。これを忘れると、装備を外した後もリスナーが残り、メモリリークや意図しない動作の原因になります。

### Blind Dart Projectile（飛翔体）

`blind_dart_projectile.lua` は命中時に複数の効果を適用する飛翔体です。

```lua
local function onhit(inst, owner, target)
    local attacker = inst._teemo_attacker  -- modmain.lua で格納したプレイヤー参照

    -- 1. ダメージ計算（攻撃者の倍率を適用）
    local damage = TEEMO_BLIND_DART_DAMAGE
    if attacker and attacker.components.combat then
        damage = damage * (attacker.components.combat.damagemultiplier or 1)
        if attacker.components.combat.externaldamagemultipliers then
            damage = damage * attacker.components.combat.externaldamagemultipliers:Get()
        end
    end

    -- 2. 初撃は GetAttacked（怯みあり）、2 撃目以降は DoDelta（怯みなし）
    if not target._teemo_dart_flinched then
        target._teemo_dart_flinched = true
        target.components.combat:GetAttacked(attacker, damage, weapon)
    else
        target.components.health:DoDelta(-damage, nil, "blind_dart")
        target.components.combat:SetTarget(attacker)  -- アグロだけ付与
    end

    -- 3. ブラインド効果（10 秒 CD）
    if weapon._lastBlindTime == nil or GetTime() - weapon._lastBlindTime >= 10 then
        doBlind(target)         -- externaldamagemultipliers を 0 に
        weapon._lastBlindTime = GetTime()
    end

    -- 4. 毒 DOT
    doToxicShot(target)

    -- 5. onattackother イベント発火
    attacker:PushEvent("onattackother", { target = target, weapon = weapon })

    inst:Remove()
end
```

> **なぜ `GetAttacked` と `DoDelta` を使い分けるのか？**: `GetAttacked` はダメージ適用 + 怯みアニメーション + 防御計算を行います。連続ヒットで毎回怯むとゲーム体験が悪いため、初撃のみ `GetAttacked`、2 撃目以降は `DoDelta`（直接 HP 減算）にしています。

#### ブラインド効果の仕組み

```lua
local function doBlind(target)
    if target.components.combat and target.components.locomotor then
        -- 攻撃力を 0 に設定（空振り状態）
        target.components.combat.externaldamagemultipliers:SetModifier(target, 0, "teemo_blind")

        -- 3 秒後に攻撃力を復元
        target._teemoBlindTask = target:DoTaskInTime(3.0, function(target)
            target.components.combat.externaldamagemultipliers:RemoveModifier(target, "teemo_blind")
        end)
    end
end
```

> **ポイント**: `externaldamagemultipliers` は `SourceModifierList` というデータ構造で、複数のソースから独立した乗算効果を管理できます。`SetModifier` で 0 を設定すると、他の MOD の倍率に影響を与えずにダメージを 0 にできます。

### Noxious Trap（毒キノコ罠）

`noxious_trap.lua` は設置型の罠アイテムです。

```lua
local function startTrap(inst)
    inst.isSetTrap = true
    inst.elapsed = 0

    -- 青い光（トラップの目印）
    inst.Light:SetColour(155/255, 225/255, 250/255)
    inst.Light:SetRadius(1.5)

    -- 1 秒後に半透明化（ステルス化）
    inst:DoTaskInTime(1.0, function()
        inst.AnimState:SetMultColour(.8, .8, .8, .8)
    end)

    inst:RemoveComponent("inventoryitem")  -- 拾えなくする

    -- 0.3 秒ごとに敵を検索
    inst.searchTask = inst:DoPeriodicTask(.3, findTarget, 2.0)
end
```

設置上限の管理:

```lua
local function onDeploy(inst, pt, deployer)
    -- 設置上限（10 個）の管理
    if deployer._noxiousTraps == nil then
        deployer._noxiousTraps = {}
    end

    -- 無効なトラップを除去
    local validTraps = {}
    for _, trap in ipairs(deployer._noxiousTraps) do
        if trap:IsValid() then table.insert(validTraps, trap) end
    end

    -- 上限超過時は最も古いトラップを削除
    while #validTraps >= MAX_TRAPS do
        local oldest = table.remove(validTraps, 1)
        if oldest:IsValid() then oldest:Remove() end
    end

    table.insert(validTraps, inst)
    deployer._noxiousTraps = validTraps
end
```

---

## カスタムコンポーネントの実装

### characterspecific.lua — キャラクター専用制限

シンプルなコンポーネントで、特定のキャラクターのみがアイテムを装備できるようにします。

```lua
local CharacterSpecific = Class(function(self, inst)
    self.inst = inst
    self.character = nil     -- 許可されたキャラクター名
    self.storable = false    -- 他キャラクターが保管できるか
    self.comment = "That does not belong to me."  -- 装備拒否時のメッセージ
end)

function CharacterSpecific:SetOwner(name) self.character = name end
function CharacterSpecific:IsStorable() return self.storable end
function CharacterSpecific:SetStorable(value) self.storable = value end

return CharacterSpecific
```

### explosive_noxious_trap.lua — 爆発ダメージロジック

罠の爆発処理をカプセル化したコンポーネントです。範囲ダメージ、毒 DOT、移動速度低下を適用します。

```lua
local Explosive_Noxious_Trap = Class(function(self, inst)
    self.inst = inst
    self.explosiveRange = 4
    self.explosiveDamage = TEEMO_NOXIOUS_TRAP_DAMAGE
    self.explosiveDotDamage = TEEMO_NOXIOUS_TRAP_DOT
    self.deployer = nil
end)

function Explosive_Noxious_Trap:OnBurnt()
    local x, y, z = self.inst.Transform:GetWorldPosition()

    local ents = TheSim:FindEntities(x, y, z, self.explosiveRange)
    for k, v in pairs(ents) do
        if v.components.combat and v.components.locomotor
            and not v:HasTag(nonTarget) and not v:HasTag("companion") then

            -- 初撃ダメージ
            v.components.combat:GetAttacked(counterPlayer, self.explosiveDamage, nil)

            -- 毒 DOT（毎秒、4 秒間）
            v.noxiousTrapDamageTask = v:DoPeriodicTask(1.0, function()
                local dmg = dotDamage
                if v:HasTag("player") then dmg = dotDamage * 0.3 end
                v.components.health:DoDelta(-dmg, nil, "noxiousTrap")
            end)

            -- 移動速度 50% 低下
            doSlow(v)

            -- 4 秒後に全効果解除
            doEndTask(v)
        end
    end
    self.inst:Remove()
end
```

> **ポイント**: プレイヤーへの DOT ダメージは 30%（`* 0.3`）に減衰しています。PvP バランスの調整としてよく使われるパターンです。

---

## ユーティリティ — teemo_poison_util.lua

毒で倒した敵の食料ドロップを腐敗させるシステムです。`require` で複数のファイルから共有しています。

```lua
local function onLootSpawned(inst, data)
    if TEEMO_POISON_SPOIL_PERCENT == nil or TEEMO_POISON_SPOIL_PERCENT <= 0 then return end
    if data.loot == nil then return end
    if data.loot.components.edible == nil then return end
    if data.loot.components.perishable == nil then return end

    -- 中心値 ±15% のランダムで鮮度を決定
    local variance = (math.random() * 2 - 1) * 0.15
    local spoilPercent = math.max(0, math.min(1, TEEMO_POISON_SPOIL_PERCENT + variance))

    local currentPercent = data.loot.components.perishable:GetPercent()
    if currentPercent > spoilPercent then
        data.loot.components.perishable:SetPercent(spoilPercent)
    end
end

local function markTeemoPoisoned(target)
    if target._teemoPoisoned then return end  -- 重複防止
    target._teemoPoisoned = true
    target:ListenForEvent("loot_prefab_spawned", onLootSpawned)
end

local function unmarkTeemoPoisoned(target)
    -- 他の毒 DOT が残っている場合はマークを維持
    if target.toxicShotDamageTask ~= nil then return end
    if target.noxiousTrapDamageTask ~= nil then return end
    target._teemoPoisoned = nil
    target:RemoveEventCallback("loot_prefab_spawned", onLootSpawned)
end

return {
    markTeemoPoisoned = markTeemoPoisoned,
    unmarkTeemoPoisoned = unmarkTeemoPoisoned,
}
```

> **なぜ `unmark` で他の DOT を確認するのか？**: Blind Dart と Noxious Trap の両方が同じターゲットに毒を付与する可能性があります。一方の DOT が終了しても、もう一方がまだ続いているなら毒マークを維持する必要があります。

---

## ネットワーク同期の実装

teemo の Net Variables は `common_postinit`（クライアント＆サーバー）で定義しています。

### 定義

```lua
local function common_postinit(inst)
    -- スタック数（0〜5）: net_byte で十分
    inst._noxiousTrapStacks = net_byte(inst.GUID, "teemo._noxiousTrapStacks", "noxioustrapstacksdirty")

    -- クールダウン秒数（0〜300）: net_ushortint（0〜65535）
    inst._flashCooldown = net_ushortint(inst.GUID, "teemo._flashCooldown", "flashcooldowndirty")
    inst._igniteCooldown = net_ushortint(inst.GUID, "teemo._igniteCooldown", "ignitecooldowndirty")

    -- サウンド通知用（値なし、イベントのみ）
    inst._sound_spwn   = net_event(inst.GUID, "teemo._sound_spwn")
    inst._sound_attack = net_event(inst.GUID, "teemo._sound_attack")
    inst._sound_emote  = net_event(inst.GUID, "teemo._sound_emote")
    inst._sound_move   = net_event(inst.GUID, "teemo._sound_move")

    -- クライアント側でサウンドイベントを受信
    inst:ListenForEvent("teemo._sound_spwn", function()
        inst.SoundEmitter:PlaySound("dontstarve/characters/teemo/spwn")
    end)
end
```

### サーバー側でのスタック管理

```lua
local function startNoxiousTrapRecovery(inst)
    inst._noxiousTrapTimer = inst._noxiousTrapTimer or 0
    inst._noxiousTrapRecoveryTask = inst:DoPeriodicTask(1, function()
        local stacks = inst._noxiousTrapStacks:value()
        if stacks >= NOXIOUS_TRAP_MAX_STACKS then
            inst._noxiousTrapTimer = 0
            return
        end
        inst._noxiousTrapTimer = inst._noxiousTrapTimer + 1
        if inst._noxiousTrapTimer >= NOXIOUS_TRAP_RECOVERY_INTERVAL then
            inst._noxiousTrapTimer = 0
            inst._noxiousTrapStacks:set(stacks + 1)  -- → クライアントに自動同期
        end
    end)
end
```

### データの流れ

```
サーバー: inst._noxiousTrapStacks:set(3)
    ↓ （自動同期）
クライアント: "noxioustrapstacksdirty" イベント発火
    ↓
UI ウィジェット: inst._noxiousTrapStacks:value() で値を取得して表示更新
```

---

## UI ウィジェットの実装

### NoxiousTrapSlot — トラップスタック UI

```lua
local NoxiousTrapSlot = Class(Widget, function(self, owner)
    Widget._ctor(self, "NoxiousTrapSlot")
    self.owner = owner

    -- 背景: DST 標準のインベントリスロット
    self.bgimage = self:AddChild(Image("images/hud.xml", "inv_slot.tex"))

    -- アイコン: クリック可能なボタン
    self.icon = self:AddChild(ImageButton(
        "images/inventoryimages/noxious_trap.xml", "noxious_trap.tex"
    ))
    self.icon:SetOnClick(function() self:OnClick() end)

    -- スタック数テキスト
    self.stackcount = self:AddChild(Text(NUMBERFONT, 22))
    self.stackcount:SetPosition(16, -16, 0)

    -- Net Variable の変更を監視
    self.inst:ListenForEvent("noxioustrapstacksdirty", function()
        self:UpdateDisplay()
    end, owner)
end)

function NoxiousTrapSlot:UpdateDisplay()
    local stacks = self.owner._noxiousTrapStacks:value()
    self.stackcount:SetString(tostring(stacks))
    -- 0 でグレーアウト
    if stacks > 0 then
        self.icon.image:SetTint(1, 1, 1, 1)
    else
        self.icon.image:SetTint(0.3, 0.3, 0.3, 1)
    end
end

function NoxiousTrapSlot:OnClick()
    if self.owner._noxiousTrapStacks:value() > 0 then
        SendModRPCToServer(MOD_RPC["teemo"]["use_noxious_trap_stack"])
    end
end
```

### SummonerSpellSlot — Flash/Ignite UI

サモナースペル用のウィジェットは、設定をテーブルで受け取る汎用設計です。

```lua
local SummonerSpellSlot = Class(Widget, function(self, owner, config)
    Widget._ctor(self, "SummonerSpellSlot")
    self.spell_name = config.spell_name       -- "flash" or "ignite"
    self.icon_atlas = config.icon_atlas
    self.cooldown_event = config.cooldown_event
    self.on_activate = config.on_activate     -- 発動時コールバック
    -- ...
end)
```

#### Flash のターゲティングモード

Flash アイコンをクリックすると、地面にレティクル（照準円）が表示され、左クリックでテレポート先を指定します。

```lua
function SummonerSpellSlot:StartFlashTargeting()
    -- レティクル生成（DST 標準の AoE ターゲティング円を流用）
    local reticule = SpawnPrefab("reticuleaoe")
    reticule.AnimState:SetScale(0.5, 0.5, 0.5)
    self._reticule = reticule

    -- マウス移動でレティクル位置を更新
    self._moveHandler = TheInput:AddMoveHandler(function(x, y)
        local pos = TheInput:GetWorldPosition()
        self._reticule.Transform:SetPosition(pos.x, 0, pos.z)
        -- 範囲内: 青、範囲外: 赤
        local dist = -- プレイヤーとの距離を計算
        if dist <= maxRange then
            self._reticule.AnimState:SetMultColour(0.3, 0.7, 1.0, 1)
        else
            self._reticule.AnimState:SetMultColour(1.0, 0.2, 0.2, 1)
        end
    end)

    -- ESC でキャンセル
    self._keyHandler = TheInput:AddKeyUpHandler(KEY_ESCAPE, function()
        self:StopFlashTargeting()
    end)

    -- 左クリックでテレポート、右クリックでキャンセル
    self._clickHandler = TheInput:AddMouseButtonHandler(function(button, down, x, y)
        if button == MOUSEBUTTON_LEFT then
            local pos = TheInput:GetWorldPosition()
            SendModRPCToServer(MOD_RPC["teemo"]["use_flash"], pos.x, pos.z)
            self:StopFlashTargeting()
            return true  -- イベント消費（他の処理に伝播させない）
        elseif button == MOUSEBUTTON_RIGHT then
            self:StopFlashTargeting()
            return true
        end
    end)
end
```

> **重要**: `StopFlashTargeting` で全ハンドラを `Remove()` しています。入力ハンドラを解除し忘れると、ターゲティングモードが終了した後もマウス移動やクリックが処理され続けます。また、`Kill()` メソッドをオーバーライドして、ウィジェット破棄時にも確実にクリーンアップしています。

### TeemoPopupNumber — フローティングダメージ数字

2 フェーズのアニメーション（上昇 → 下降 + フェードアウト）で数字を表示します。

```lua
local TeemoPopupNumber = Class(Widget,
    function(self, owner, val, size, pos, dir, height, colour)
        Widget._ctor(self, "TeemoPopupNumber")
        self.text = self:AddChild(Text(NUMBERFONT, size, tostring(val)))
        self.colour = colour or {1, 1, 1, 1}
        self.rise = 24    -- 上昇距離
        self.drop = 24    -- 下降距離
        self.speed = 68   -- 横移動速度
        self.progress = 0
        self:StartUpdating()  -- 毎フレーム OnUpdate を呼ぶ
    end
)

function TeemoPopupNumber:OnUpdate(dt)
    if self.progress < 1 then
        -- Phase 1: 上昇 + フェードイン
        self.progress = math.min(1, self.progress + dt * self.ts_step_1)
        -- イージング計算...
    elseif self.progress < 2 then
        -- Phase 2: 下降 + フェードアウト
        self.progress = math.min(2, self.progress + dt * self.ts_step_2)
        -- イージング計算...
    else
        self:Kill()  -- アニメーション完了 → 自動削除
        return
    end
    -- ワールド座標 → スクリーン座標に変換
    self:SetPosition(TheSim:GetScreenPos(self.pos:Get()))
end
```

---

## セーブ・ロードの実装

teemo では Net Variables の値とタイマーをセーブデータに保存しています。

```lua
-- master_postinit 内

local _OnSave = inst.OnSave
inst.OnSave = function(inst, data)
    if _OnSave then _OnSave(inst, data) end     -- 元の OnSave を呼ぶ
    data.noxious_trap_stacks = inst._noxiousTrapStacks:value()
    data.noxious_trap_timer = inst._noxiousTrapTimer
    data.flash_cooldown = inst._flashCooldown:value()
    data.ignite_cooldown = inst._igniteCooldown:value()
end

local _OnLoad = inst.OnLoad
inst.OnLoad = function(inst, data)
    if _OnLoad then _OnLoad(inst, data) end     -- 元の OnLoad を呼ぶ
    if data then
        if data.noxious_trap_stacks then
            inst._noxiousTrapStacks:set(data.noxious_trap_stacks)
        end
        if data.noxious_trap_timer then
            inst._noxiousTrapTimer = data.noxious_trap_timer
        end
        if data.flash_cooldown then
            inst._flashCooldown:set(data.flash_cooldown)
        end
        if data.ignite_cooldown then
            inst._igniteCooldown:set(data.ignite_cooldown)
        end
    end
end
```

> **ポイント**: `_noxiousTrapTimer`（ローカル変数）と `_noxiousTrapStacks:value()`（Net Variable の値）の両方を保存しています。Net Variable の `:value()` はサーバーの現在値を取得します。

Noxious Trap のセーブ・ロードも同様に:

```lua
-- noxious_trap.lua
local function onSave(inst, data)
    data.isSetTrap = inst.isSetTrap
    data.elapsed = inst.elapsed     -- 経過時間を保存
end

local function onLoad(inst, data)
    if data and data.isSetTrap then
        startTrap(inst)             -- 罠を再度有効化
    end
    if data and data.elapsed then
        inst.elapsed = data.elapsed -- 残り時間を復元
    end
end
```

---

## 能力一覧まとめ

| 能力 | 種類 | 持続時間 | クールダウン | 設定項目 |
|---|---|---|---|---|
| カモフラージュ | パッシブ | 静止中ずっと | なし（静止 3 秒で発動） | なし（組み込み） |
| Blind Dart（初撃） | 武器 | 即時 | 攻撃間隔 1.0 秒 | Hit Damage, Durability |
| ブラインド効果 | 命中時 | 3 秒 | 10 秒 | なし（組み込み） |
| 毒 DOT（Toxic Shot） | 命中時 | 4 秒 | なし | DOT Damage |
| Noxious Trap（爆発） | 設置型 | 5 分（罠の寿命） | 30 秒（スタック回復） | Hit Damage, DOT Damage |
| 罠の移動速度低下 | 範囲 | 4 秒 | なし | なし（-50% 固定） |
| Flash | サモナースペル | 即時 | 5 分 | Range, Cooldown |
| Ignite | サモナースペル | 5 秒 | 3 分 | Damage, Cooldown |
| キノコの達人 | パッシブ | 永続 | なし | On/Off |
| 毒による食料腐敗 | パッシブ | 永続 | なし | Spoilage %, Off |

---

## 参考リンク

- [Lua スクリプティング総合ガイド](README.md) — 本ガイドの概念解説編
- [DST-teemo GitHub リポジトリ](https://github.com/pinpikokun/DST-teemo) — ソースコード全体
- [Captain Teemo（Steam Workshop）](https://steamcommunity.com/sharedfiles/filedetails/?id=390684095)
- [← メインガイドに戻る](../README.md)

---

## ライセンス

このドキュメントは自由に利用・改変・再配布できます。DST MOD 制作にお役立てください。

---

<sub>Author: [pinpikokun](https://steamcommunity.com/profiles/76561198076111536/)</sub><br>
<sub>[![GitHub](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/pinpikokun/DST-teemo) [![Steam Workshop](https://img.shields.io/badge/Steam%20Workshop-Subscribe-blue?logo=steam)](https://steamcommunity.com/sharedfiles/filedetails/?id=390684095)</sub>
