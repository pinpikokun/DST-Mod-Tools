# アイテム・武器防具アニメーション編

[← README.md に戻る](README.md)

アイテム（武器・防具・ツール・食べ物など）を MOD に追加するときに必要なアニメーションについて解説します。アイテムには「地面に置いたときの見た目」と「キャラクターが装備したときの見た目」の 2 種類があります。

---

## アイテムアニメーションとは

DST でアイテムを地面にドロップすると、そのアイテムの見た目が表示されます。この見た目が「アイテムアニメーション」です。

ほとんどのアイテムは `idle`（静止状態）のアニメーション 1 つだけで十分です。

### 構成

| 項目 | 値 |
|---|---|
| Bank | 独自の名前（アイテム名と同じにするのが一般的） |
| Build | 独自の名前（Bank と同じにするのが安全） |
| 必要なアニメーション | `idle`（静止状態） |

### Spriter での作り方

1. 新規プロジェクトを作成する
2. アイテムの画像（PNG）を 1 枚配置する
3. Entity 名（= Bank 名）をアイテム名に設定する
4. `idle` という名前のアニメーションを作成する
5. 画像のピボット（回転軸）を画像の中央下部に設定する

> **Tips**: アイテムアニメーションは基本的に静止画 1 枚で OK です。動く必要がなければ、1 フレームだけのアニメーションで問題ありません。

### Lua 設定例

```lua
-- prefab ファイル
local assets = {
    Asset("ANIM", "anim/my_weapon.zip"),
}

local function fn()
    local inst = CreateEntity()

    inst.entity:AddAnimState()
    inst.AnimState:SetBank("my_weapon")
    inst.AnimState:SetBuild("my_weapon")
    inst.AnimState:PlayAnimation("idle")

    -- ... 他の設定
    return inst
end
```

---

## 装備スワップアニメーションとは

キャラクターが武器やツールを手に持ったとき、キャラクターの手元に表示される見た目が「装備スワップアニメーション」です。

DST のキャラクターには `swap_object` というシンボル（レイヤー）があり、装備するとこのレイヤーの画像が差し替えられます。

```
キャラクターの手（swap_object レイヤー）
  ├── 何も持っていない → 空
  ├── 斧を装備 → 斧の画像に差し替え
  └── MOD武器を装備 → MOD武器の画像に差し替え
```

### 構成

| 項目 | 値 |
|---|---|
| Bank | `swap_object`（DST 本体の Bank をそのまま使用） |
| Build | `swap_<アイテム名>`（独自に作成） |
| シンボル名 | Build 名と同じにするのが一般的 |

> **Note**: Bank は必ず `swap_object` を使ってください。これは DST 本体が定義している Bank で、キャラクターの手に持つアイテムの動きを制御しています。

### Spriter での作り方

1. 新規プロジェクトを作成する（ファイル名を `swap_<アイテム名>` にする）
2. Entity 名を `swap_object` に設定する（**重要**: 独自名ではなく `swap_object` にする）
3. 手に持った状態のアイテム画像（PNG）を配置する
4. ピボットをキャラクターの手の位置に合わせる

> **Tips**: ピボット位置は、DST 本体の既存武器（斧やハンマーなど）の `.scml` を参考にすると合わせやすいです。

### Lua 設定例

```lua
-- prefab ファイルの assets
local assets = {
    Asset("ANIM", "anim/my_weapon.zip"),         -- 地面のアイテム
    Asset("ANIM", "anim/swap_my_weapon.zip"),     -- 装備時の見た目
}

-- 装備したとき
local function onequip(inst, owner)
    -- キャラクターの swap_object シンボルを差し替える
    owner.AnimState:OverrideSymbol("swap_object", "swap_my_weapon", "swap_my_weapon")
    -- 装備アニメーションのレイヤーを表示
    owner.AnimState:Show("ARM_carry")
    owner.AnimState:Hide("ARM_normal")
end

-- 装備を外したとき
local function onunequip(inst, owner)
    -- 差し替えを元に戻す
    owner.AnimState:ClearOverrideSymbol("swap_object")
    owner.AnimState:Hide("ARM_carry")
    owner.AnimState:Show("ARM_normal")
end
```

### OverrideSymbol の引数

```lua
owner.AnimState:OverrideSymbol("swap_object", "swap_my_weapon", "swap_my_weapon")
--                               ↑ 差し替え先      ↑ Build 名          ↑ シンボル名
--                          （キャラの手）   （ZIP の Build）    （ZIP 内のシンボル）
```

| 引数 | 説明 |
|---|---|
| 第 1 引数 | 差し替えるシンボル名（キャラクター側）。武器なら `"swap_object"` |
| 第 2 引数 | 差し替え元の Build 名（装備アニメーション ZIP の Build） |
| 第 3 引数 | 差し替え元のシンボル名（通常は Build 名と同じ） |

---

## 帽子・防具の装備

帽子や防具もスワップの仕組みは同じですが、差し替えるシンボルが異なります。

| 装備スロット | 差し替えるシンボル | 例 |
|---|---|---|
| 手（武器・ツール） | `swap_object` | 斧、ハンマー、杖 |
| 頭（帽子） | `swap_hat` | ヘルメット、花飾り |
| 胴体（防具） | `swap_body` | ログスーツ、バックパック |

```lua
-- 帽子の場合
owner.AnimState:OverrideSymbol("swap_hat", "swap_my_hat", "swap_my_hat")

-- 防具の場合
owner.AnimState:OverrideSymbol("swap_body", "swap_my_armor", "swap_my_armor")
```

---

## インベントリアイコンも忘れずに

アイテムを追加する場合、Spriter アニメーションとは別に **インベントリアイコン**（持ち物欄に表示される画像）が必要です。

インベントリアイコンは PNG → TEX + XML 形式で作成します。詳しくは [README.md の Appendix A](README.md#appendix-a-インベントリアイコンpng--tex--xml) を参照してください。

---

## まとめ

| 作るもの | Bank | Build | 用途 |
|---|---|---|---|
| アイテム（地面） | 独自 | 独自 | 地面に置いたときの見た目 |
| 装備スワップ（武器） | `swap_object`（流用） | `swap_<アイテム名>` | 手に持ったときの見た目 |
| 装備スワップ（帽子） | `swap_hat`（流用） | `swap_<帽子名>` | 頭に装備したときの見た目 |
| 装備スワップ（防具） | `swap_body`（流用） | `swap_<防具名>` | 胴体に装備したときの見た目 |
| インベントリアイコン | — | — | TEX + XML 形式（Spriter 不要） |

---

[← README.md に戻る](README.md) | [キャラクター編 ←](README-character.md) | [エフェクト編 →](README-effect.md)

---

## ライセンス

このドキュメントは自由に利用・改変・再配布できます。DST MOD 制作にお役立てください。

---

<sub>Author: [pinpikokun](https://steamcommunity.com/profiles/76561198076111536/)</sub><br>
<sub>[![GitHub](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/pinpikokun/DST-teemo) [![Steam Workshop](https://img.shields.io/badge/Steam%20Workshop-Subscribe-blue?logo=steam)](https://steamcommunity.com/sharedfiles/filedetails/?id=390684095)</sub>
