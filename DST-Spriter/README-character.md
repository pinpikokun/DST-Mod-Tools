# キャラクターアニメーション編

[← README.md に戻る](README.md)

キャラクター MOD を作るときに必要なアニメーションについて解説します。DST のキャラクターアニメーションは他の種類に比べて最も複雑ですが、基本の仕組みを理解すれば作成できます。

> **HTML 版**: [README-character.html](https://pinpikokun.github.io/DST-Mod-Guide/DST-Spriter/README-character.html)

---

## キャラクターアニメーションとは

DST のキャラクター（ウィルソンやウェンディなど）は、**身体のパーツごとに分かれた画像**（シンボル）を Spriter 上で組み合わせて動かしています。

たとえば、1 体のキャラクターは以下のようなパーツに分かれています。

| パーツ名（シンボル） | 説明 |
|---|---|
| `head` | 頭 |
| `face` | 顔の表情 |
| `hair` | 髪 |
| `torso` | 胴体 |
| `arm_upper` | 上腕 |
| `arm_lower` | 前腕 |
| `hand` | 手 |
| `leg` | 脚 |
| `foot` | 足 |
| `swap_object` | 手に持っているもの（装備時に差し替えられる） |

> **Tips**: この構造のおかげで、「動き（Bank）はウィルソンと同じだけど、見た目（Build）は自分のキャラ」という作り方ができます。多くのキャラクター MOD はこの方法を使っています。

---

## 必要なアニメーション（ポーズ）一覧

DST のキャラクターには多数のアニメーションポーズが必要です。以下は主なものです。

### 基本ポーズ（最低限必要）

| アニメーション名 | 説明 |
|---|---|
| `idle_loop` | 待機（立っている状態） |
| `walk_loop` | 歩行 |
| `run_loop` | 走行 |
| `attack` | 攻撃 |
| `hit` | ダメージを受ける |
| `death` | 死亡 |
| `pickup` | アイテムを拾う |
| `eat` / `eat_loop` | 食べる |
| `chop_loop` / `chop_pre` | 木を切る |
| `mine_loop` / `mine_pre` | 採掘する |

### その他のポーズ（状況に応じて）

| アニメーション名 | 説明 |
|---|---|
| `build_loop` / `build_pre` | クラフトする |
| `give` | アイテムを渡す |
| `sleep_loop` / `sleep_pre` | 寝る |
| `yawn` | あくび |
| `frozen` | 凍結 |
| `channel_loop` | チャネリング（釣り等） |
| `emote_*` | エモート（ダンス等） |

> **Note**: すべてのポーズを自分で一から作る必要はありません。DST の既存キャラクター（ウィルソン等）の Bank をそのまま使い、Build（見た目）だけ自分で作るのが一般的です。これにより、必要なポーズのフレーム構成が自動的に正しくなります。

---

## Bank と Build の関係

キャラクターアニメーションでは、Bank と Build の関係が特に重要です。

```
Bank "wilson" （ウィルソンの動き）
  ├── Build "wilson" （ウィルソンの見た目）← DST 本体
  ├── Build "willow" （ウィローの見た目）← DST 本体
  └── Build "my_character" （独自キャラの見た目）← MOD で追加
```

多くのキャラクター MOD は **Bank はウィルソン（`wilson`）のものを使い**、Build だけ独自に作成しています。これにより、すべての動きが自動的にウィルソンと同じように動作します。

### 独自 Bank を使う場合

独自の動き（独自の攻撃モーションなど）を追加したい場合は、自分で Bank を定義することもできます。

---

## ゴーストアニメーション

キャラクターが死亡すると、ゴースト（幽霊）姿になります。

### 既存の "ghost" Bank を流用する方法

DST 本体にはゴースト用の Bank（`ghost`）がすでに存在します。Build（見た目）だけ自分で作れば、ゴーストの動きは自動的に正しくなります。

| 項目 | 値 |
|---|---|
| Bank | `ghost`（DST 本体の Bank をそのまま使用） |
| Build | `ghost_<キャラ名>_build`（自分で作成） |
| ZIP ファイル名 | `ghost_<キャラ名>_build.zip` |

### Lua 設定例

```lua
-- prefab ファイルの assets
local assets = {
    Asset("ANIM", "anim/my_character.zip"),
    Asset("ANIM", "anim/ghost_my_character_build.zip"),
}

-- キャラクター定義内の skin_modes
inst.components.skinner.skin_modes = {
    {
        type = "ghost_skin",
        anim_bank = "ghost",
        idle_anim = "idle",
        scale = 0.75,
        offset = { 0, -25 },
    },
}
```

---

## スキンシステム

DST にはキャラクターの見た目を切り替える「スキン」システムがあります。これも Bank / Build の仕組みを活用しています。

### 仕組み

```
Bank "my_character"（動きの型）
  ├── Build "my_character"        ← デフォルトスキン
  └── Build "my_character_formal" ← フォーマルスキン（追加スキン）
```

スキンを切り替えるとは、**同じ Bank に対して Build を差し替える** ことです。スキンごとに新しい ZIP を用意して `anim/` に配置し、Lua 側でスキン切り替え時に `SetBuild()` を呼ぶだけです。

> **Tips**: スキンを作るには、デフォルト Build と同じシンボル構成（パーツ名・レイヤー構成）で別の見た目を描くだけです。動きの定義は不要です。

---

## まとめ

| 作るもの | Bank | Build | 備考 |
|---|---|---|---|
| メインキャラ | 独自 or `wilson` | 独自 | 全ポーズのシンボルが必要 |
| ゴースト | `ghost`（流用） | 独自 | ゴースト用パーツだけ作ればOK |
| スキン | メインと同じ | 独自 | シンボル構成を合わせる |

---

[← README.md に戻る](README.md) | [アイテム・武器防具編 →](README-item-equipment.md)

---

## ライセンス

このドキュメントは自由に利用・改変・再配布できます。DST MOD 制作にお役立てください。

---

<sub>Author: [pinpikokun](https://steamcommunity.com/profiles/76561198076111536/)</sub><br>
<sub>[![GitHub](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/pinpikokun/DST-teemo) [![Steam Workshop](https://img.shields.io/badge/Steam%20Workshop-Subscribe-blue?logo=steam)](https://steamcommunity.com/sharedfiles/filedetails/?id=390684095)</sub>
