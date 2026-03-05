# エフェクトアニメーション編

[← README.md に戻る](README.md)

エフェクト（視覚効果）など、キャラクターやアイテム以外のアニメーションについて解説します。これらは MOD に演出を加えるために重要な要素です。

> **HTML 版**: [README-effect.html](https://pinpikokun.github.io/DST-Mod-Guide/DST-Spriter/README-effect.html)

---

## エフェクトアニメーションとは

DST の「エフェクト」とは、バフ・デバフの表示、爆発、煙、光などの **一時的な視覚効果** のことです。エフェクトはキャラクターやアイテムとは独立した存在（Prefab）として生成され、表示後に自動的に消えるものが多いです。

### 主なエフェクトの種類

| エフェクト | 説明 | 例 |
|---|---|---|
| バフ/デバフ表示 | キャラクターに付与される効果の表示 | シールド、毒、凍結 |
| 爆発/煙 | 一瞬だけ表示される効果 | トラップの爆発、毒煙 |
| ループエフェクト | 条件を満たす間ずっと表示される効果 | バリア、範囲表示 |

---

## 既存 Bank の流用パターン

エフェクトアニメーションの大きな特徴は、**DST 本体の Bank をそのまま流用できる** ことです。Bank（動きの型）は既存のものを使い、Build（見た目）だけ自分で差し替えれば、動きのタイミングや表示方法が自動的に正しくなります。

### よく使われる既存 Bank

| Bank 名 | 元の用途 | 流用例 |
|---|---|---|
| `forcefield` | バリアシールド | バフ/デバフの表示 |
| `poopcloud` | 肥料の煙 | 毒煙、爆発エフェクト |
| `splash` | 水しぶき | 衝撃エフェクト |
| `sparks` | 火花 | ヒットエフェクト |

### 流用の具体例

```
Bank "forcefield" (DST本体のバリアの動き)
  └── Build "my_shield_effect" (独自: シールド効果の見た目)

Bank "poopcloud" (DST本体の煙の動き)
  ├── Build "my_poison_smoke" (独自: 毒煙エフェクトの見た目)
  └── Build "my_trap_explosion" (独自: トラップ爆発の見た目)
```

> **Tips**: 既存 Bank を流用するには、その Bank のアニメーション名（`idle`、`open`、`idle_loop` 等）を調べ、同じ名前のアニメーションとして Build を作ります。Bank が持つアニメーション名は、DST 本体の Spriter ファイルを参照するか、Lua コードから推測できます。

---

## エフェクトの Spriter での作り方

### シンプルな 1 フレームエフェクト

煙や爆発など、1 枚の画像を表示するだけのエフェクトはとても簡単です。

1. 新規プロジェクトを作成する
2. エフェクト画像（PNG）を 1 枚配置する
3. Entity 名を流用する Bank 名に設定する（例: `poopcloud`）
4. `idle` という名前のアニメーションを作成する

### 開始 → ループのエフェクト

バリアやシールドのように、「出現 → ループ表示」するエフェクトは 2 つのアニメーションが必要です。

| アニメーション名 | 説明 |
|---|---|
| `open` | 出現時のアニメーション（1回再生） |
| `idle_loop` | ループ表示（繰り返し再生） |

### Lua 設定例

```lua
-- 1フレームエフェクト（煙エフェクトの例）
local assets = {
    Asset("ANIM", "anim/my_smoke_effect.zip"),
}

local function fn()
    local inst = CreateEntity()
    inst.entity:AddAnimState()

    inst.AnimState:SetBank("poopcloud")
    inst.AnimState:SetBuild("my_smoke_effect")
    inst.AnimState:PlayAnimation("idle")

    -- アニメーション終了後に自動削除
    inst:ListenForEvent("animover", inst.Remove)

    return inst
end
```

```lua
-- 開始→ループエフェクト（シールド効果の例）
local assets = {
    Asset("ANIM", "anim/my_shield_effect.zip"),
}

local function fn()
    local inst = CreateEntity()
    inst.entity:AddAnimState()

    inst.AnimState:SetBank("forcefield")
    inst.AnimState:SetBuild("my_shield_effect")
    inst.AnimState:PlayAnimation("open")                -- まず出現アニメーション
    inst.AnimState:PushAnimation("idle_loop", true)     -- 続けてループ

    return inst
end
```

---

## まとめ

| 作るもの | Bank | Build | ポイント |
|---|---|---|---|
| バフ/デバフ表示 | `forcefield` 等（流用） | 独自 | `open` → `idle_loop` のパターン |
| 爆発/煙 | `poopcloud` 等（流用） | 独自 | `idle` 1 フレームでOK。`animover` で自動削除 |

### 既存 Bank を調べる方法

使いたいエフェクトに近い DST 本体のエフェクトがあれば、以下の手順で Bank を調べられます。

1. DST 本体の `scripts/prefabs/` フォルダから近いエフェクトの prefab ファイルを探す
2. `SetBank("xxx")` の部分を確認する → これが Bank 名
3. `PlayAnimation("yyy")` の部分を確認する → これが使えるアニメーション名

> **Tips**: DST 本体のスクリプトは ZIP にバンドルされています。`C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\data\databundles\scripts.zip` を展開すると `scripts\prefabs\` フォルダが見つかります。テキストエディタで開いて `SetBank` を検索すると、どんな Bank が使われているか確認できます。

---

[← README.md に戻る](README.md) | [アイテム・武器防具編 ←](README-item-equipment.md)

---

## ライセンス

このドキュメントは自由に利用・改変・再配布できます。DST MOD 制作にお役立てください。

---

<sub>Author: [pinpikokun](https://steamcommunity.com/profiles/76561198076111536/)</sub><br>
<sub>[![GitHub](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/pinpikokun/DST-teemo) [![Steam Workshop](https://img.shields.io/badge/Steam%20Workshop-Subscribe-blue?logo=steam)](https://steamcommunity.com/sharedfiles/filedetails/?id=390684095)</sub>
