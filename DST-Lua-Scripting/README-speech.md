# DST キャラクターセリフ（Speech）ファイル作成ガイド — キャラクターに個性あるセリフを設定しよう

Don't Starve Together (DST) のキャラクター MOD で、アイテムを調べた時やイベント発生時に表示される **セリフ（Speech）** をカスタマイズする方法を解説します。セリフはキャラクターの個性を表現する最も重要な要素です。

> **HTML 版**: [README-speech.html](https://pinpikokun.github.io/DST-Mod-Guide/DST-Lua-Scripting/README-speech.html)

---

## 目次

1. [Speech ファイルとは](#speech-ファイルとは)
2. [Speech ファイルの入手と準備](#speech-ファイルの入手と準備)
3. [ファイルの全体構造](#ファイルの全体構造)
4. [主要カテゴリの解説](#主要カテゴリの解説)
5. [セリフの書き方パターン](#セリフの書き方パターン)
6. [modmain.lua での読み込み設定](#modmainlua-での読み込み設定)
7. [セリフ作成の実践テクニック](#セリフ作成の実践テクニック)
8. [よくあるトラブルと解決法](#よくあるトラブルと解決法)
9. [Tips・参考情報](#tips参考情報)
10. [参考リンク](#参考リンク)

---

## Speech ファイルとは

Speech ファイルとは、キャラクターがゲーム内で発するセリフデータを定義した Lua ファイルです。

- プレイヤーがアイテムや生物を **調べた時**（右クリック）
- **空腹・寒い・暑い** などの状態変化が起きた時
- **戦闘開始時** や **撤退時**
- **食事をした時**
- **暗闇に入った時**

など、さまざまな場面でキャラクターがセリフを表示します。このセリフをカスタマイズすることで、キャラクターに独自の性格や口調を持たせることができます。

### ファイルの配置場所

```
mods/my_character/scripts/speech_my_character.lua
```

> **なぜセリフが重要なのか？**: DST にはキャラクターボイスがありますが、プレイヤーが実際に「読む」のはセリフです。同じアイテムを調べても、ウィルソンは科学者らしいコメントをし、ウェンディは暗い発言をします。セリフこそがキャラクターの個性を伝える最大の手段です。

---

## Speech ファイルの入手と準備

オリジナルの Speech ファイルをゼロから書くのは非常に大変です（数千行あります）。ウィルソンの Speech ファイルをコピーして、セリフを書き換える方法が最も効率的です。

### Step 1: scripts.zip を展開する

DST のスクリプトは ZIP にバンドル（まとめて格納）されています。

```
C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\data\databundles\scripts.zip
```

> **見つけ方**:
>
> 1. Steam を開く
> 2. Don't Starve Together を右クリック
> 3. **管理** を選択
> 4. **ローカルファイルを閲覧** をクリック
> 5. `data` フォルダを開く
> 6. `databundles` フォルダを開く
> 7. `scripts.zip` を見つける

この ZIP を右クリック → **すべて展開** で展開してください。展開先はどこでも構いません（デスクトップなど分かりやすい場所で OK）。

### Step 2: speech_wilson.lua をコピーする

展開したフォルダの中に `scripts\speech_wilson.lua` があります。このファイルをコピーして、自分の MOD フォルダに配置します。

1. `speech_wilson.lua` をコピー（Ctrl+C）
2. `mods/my_character/scripts/` フォルダに貼り付け（Ctrl+V）
3. ファイル名を `speech_my_character.lua` に変更（右クリック → 名前の変更）

> **重要**: ファイル名の `my_character` 部分は、自分のキャラクター名（小文字）に合わせてください。modmain.lua の `require "speech_my_character"` と一致する必要があります。

### 最小構成（まずは動かしたい場合）

セリフのカスタマイズは後回しにして、まず MOD を動かしたい場合は以下の 1 行だけのファイルでも動作します。

```lua
return require("speech_wilson")
```

> **ポイント**: この方法はウィルソンのセリフをそのまま使います。MOD が正常に動くことを確認してから、セリフのカスタマイズに取り掛かるのがおすすめです。

---

## ファイルの全体構造

Speech ファイルは、1 つの大きな Lua テーブル（データのまとまり）を `return` で返す構造です。

```lua
return {
    -- ========== アクション失敗時のセリフ ==========
    ACTIONFAIL = {
        SHAVE = {
            AWAKEBEEFALO = "I'm not going to try that while it's awake.",
            GENERIC = "I can't shave that!",
        },
        STORE = {
            GENERIC = "It's full.",
            NOTALLOWED = "That doesn't go there.",
        },
        -- ... 他のアクション
    },

    -- ========== イベント告知 ==========
    ANNOUNCE_HUNGRY = "I need to eat!",
    ANNOUNCE_COLD = "Brrr! It's freezing!",
    -- ... 他のイベント

    -- ========== 戦闘セリフ ==========
    BATTLECRY = {
        GENERIC = {"Fight me!", "Take this!"},
        -- ...
    },

    -- ========== アイテム・生物の調査 ==========
    DESCRIBE = {
        CAMPFIRE = {
            EMBERS = "The fire is almost out.",
            GENERIC = "A nice fire.",
            HIGH = "That fire is getting out of hand!",
            LOW = "The fire is getting low.",
            NORMAL = "Nice and warm.",
            OUT = "I should start a new fire.",
        },
        -- ... 数百のアイテム・生物
    },

    -- ========== 食事時のセリフ ==========
    EAT_FOOD = {
        GENERIC = "Yum!",
        PAINFUL = "That was a bad idea.",
        SPOILED = "Yuck! That was spoiled!",
        STALE = "That wasn't the freshest.",
    },

    -- ... 他のカテゴリ
}
```

> **なぜこの構造なのか？**: DST のゲームエンジンは、状況に応じてこのテーブルから適切なキーを参照してセリフを取得します。テーブルの構造（キーの名前やネストの深さ）はゲームエンジンが期待する形式に合わせる必要があります。

---

## 主要カテゴリの解説

Speech ファイルに含まれるカテゴリは数十種類あります。ここでは特に重要なものを解説します。

### ACTIONFAIL — アクション失敗時

プレイヤーが何かを実行しようとして失敗した時に表示されるセリフです。

```lua
ACTIONFAIL = {
    SHAVE = {                                        -- 毛を剃ろうとした時
        AWAKEBEEFALO = "That beast is awake!",       -- 起きているビーファロー
        GENERIC = "I can't shave that!",             -- 剃れないもの
    },
    STORE = {                                        -- 収納しようとした時
        GENERIC = "It's full.",                      -- 容量いっぱい
        NOTALLOWED = "That doesn't go there.",       -- 入れられないもの
    },
    COOK = {                                         -- 調理しようとした時
        GENERIC = "I can't cook that.",              -- 調理できないもの
        TOOFAR = "I need to get closer.",            -- 遠すぎる
    },
    GIVE = {                                         -- 渡そうとした時
        GENERIC = "They don't want that.",           -- 受け取らない
        DEAD = "They can't use this anymore.",       -- 死んでいる
    },
    -- ... 他のアクション
},
```

### ANNOUNCE_* — イベント告知

ゲーム内でさまざまなイベントが発生した時にキャラクターが自動的に発するセリフです。

```lua
-- 生存に関する告知
ANNOUNCE_HUNGRY = "I need to eat!",                  -- お腹が空いた
ANNOUNCE_COLD = "Brrr! It's freezing!",              -- 寒い
ANNOUNCE_HOT = "It's so hot...",                     -- 暑い

-- 時間帯
ANNOUNCE_DUSK = "It'll be dark soon.",               -- 夕方になった

-- 危険の接近
ANNOUNCE_HOUNDS = "Something is coming!",            -- ハウンドの襲撃予告
ANNOUNCE_CHARLIE_ATTACK = "Something hit me!",       -- 暗闇で攻撃された

-- 明暗の変化
ANNOUNCE_ENTER_DARK = "I can't see a thing!",        -- 暗闇に入った
ANNOUNCE_ENTER_LIGHT = "That's better.",             -- 明るい場所に入った

-- 食事の反応
ANNOUNCE_EAT = {
    GENERIC = "Yum!",                                -- 通常
    PAINFUL = "Ugh... that was bad.",                 -- 体力を減らす食べ物
    SPOILED = "That was rotten!",                     -- 腐った食べ物
    STALE = "That wasn't very fresh.",                -- 鮮度が落ちた食べ物
    INVALID = "I can't eat that!",                    -- 食べられないもの
},

-- 睡眠関連
ANNOUNCE_NODANGERSLEEP = "It's too dangerous!",      -- 危険で眠れない
ANNOUNCE_NOHUNGERSLEEP = "I'm too hungry to sleep.", -- 空腹で眠れない
ANNOUNCE_NODAYSLEEP = "I'm not tired yet.",          -- 昼は眠れない
```

### BATTLECRY — 戦闘開始時の掛け声

戦闘を開始した時にキャラクターが叫ぶセリフです。配列（複数の候補）で指定すると、ランダムに 1 つが選ばれます。

```lua
BATTLECRY = {
    GENERIC = {                                      -- 通常の敵
        "Fight me!",
        "Take this!",
        "Prepare yourself!",
    },
    PIG = {                                          -- ブタへの掛け声
        "Sorry, pig!",
        "Nothing personal!",
    },
    PREY = {                                         -- 獲物（ウサギ等）への掛け声
        "Come here!",
        "Got you!",
    },
    SPIDER = {                                       -- クモへの掛け声
        "Die, spider!",
        "Ugh, spiders!",
    },
},
```

### COMBAT_QUIT — 戦闘撤退時

戦闘をやめた時のセリフです。こちらも配列で複数指定できます。

```lua
COMBAT_QUIT = {
    GENERIC = {
        "That was close.",
        "I should run.",
        "Not worth the fight.",
    },
},
```

### DESCRIBE — アイテム・生物の調査（最大のセクション）

**DESCRIBE は Speech ファイルの大部分を占める最大のカテゴリ** です。プレイヤーがアイテムや生物を調べた時のセリフを定義します。数百のアイテムがあり、それぞれに状態別のサブカテゴリがある場合もあります。

#### 単純な調査セリフ

```lua
DESCRIBE = {
    AXE = "A trusty axe.",                           -- オノ
    PICKAXE = "Good for mining.",                    -- つるはし
    TORCH = "It lights the way.",                    -- たいまつ
    FLINT = "A sharp piece of flint.",               -- フリント
    TWIGS = "Some twigs.",                            -- 小枝
    ROCKS = "Just some rocks.",                      -- 岩
    -- ...
},
```

#### 状態別のセリフ

一部のアイテムには複数の状態があり、状態ごとに異なるセリフを設定できます。

```lua
DESCRIBE = {
    CAMPFIRE = {                                     -- たき火
        EMBERS = "The fire is almost out.",          -- 残り火
        GENERIC = "A nice fire.",                    -- 通常
        HIGH = "That fire is out of hand!",          -- 大きすぎる
        LOW = "The fire is getting low.",            -- 小さくなってきた
        NORMAL = "Nice and warm.",                   -- 普通
        OUT = "I should start a new fire.",          -- 消えた
    },
    BEEFALO = {                                      -- ビーファロー
        GENERIC = "A big hairy beast.",              -- 通常
        FOLLOWER = "It likes me!",                   -- 懐いている
        NAKED = "It's been shaved.",                 -- 毛を刈られた
        SLEEPING = "It's sleeping.",                 -- 寝ている
    },
    -- ...
},
```

#### PLAYER（他のプレイヤーキャラクター）の調査

マルチプレイで他のプレイヤーを調べた時のセリフです。相手の行動によって異なるセリフが表示されます。

```lua
DESCRIBE = {
    PLAYER = {
        GENERIC = "Hi, %s!",                         -- 通常（%s にプレイヤー名が入る）
        ATTACKER = "%s looks aggressive...",          -- 攻撃的なプレイヤー
        MURDERER = "%s is a murderer!",              -- 殺人を犯したプレイヤー
        REVIVER = "%s is a good friend.",             -- 蘇生してくれたプレイヤー
        GHOST = "%s could use a heart.",              -- ゴースト状態
    },
    -- ...
},
```

> **ポイント**: `%s` はプレースホルダー（置き換え記号）です。ゲーム実行時に相手プレイヤーの名前に自動的に置き換えられます。

### EAT_FOOD — 食事時

食べ物を食べた時のセリフです。

```lua
EAT_FOOD = {
    GENERIC = "Yum!",                                -- 通常の食べ物
    PAINFUL = "That was a bad idea.",                 -- ダメージを受ける食べ物
    SPOILED = "That was awful!",                      -- 腐った食べ物
    STALE = "It could have been fresher.",            -- 鮮度が落ちた食べ物
    YUCKY = "I don't want to eat that!",             -- 食べたくないもの
},
```

### DESCRIBE_GENERIC — 汎用調査

DESCRIBE テーブルに該当するキーがないアイテムを調べた時のフォールバック（代替）セリフです。

```lua
DESCRIBE_GENERIC = "I'm not sure what that is.",
DESCRIBE_TOODARK = "It's too dark to see!",          -- 暗くて見えない時
DESCRIBE_SMOLDERING = "It's about to catch fire!",   -- くすぶっている時
```

---

## セリフの書き方パターン

DST の Speech ファイルでは、セリフの書き方に 4 つのパターンがあります。

### パターン 1: 単一文字列

最もシンプルな形式です。常に同じセリフが表示されます。

```lua
ANNOUNCE_HUNGRY = "I need to eat!",
```

### パターン 2: 配列（ランダム選択）

複数のセリフを配列で指定すると、表示されるたびにランダムで 1 つが選ばれます。セリフにバリエーションを持たせたい時に使います。

```lua
BATTLECRY = {
    GENERIC = {
        "Fight me!",
        "Prepare yourself!",
        "Take this!",
        "You asked for it!",
    },
},
```

> **なぜ配列を使うのか？**: 何度も同じセリフが表示されると単調になります。特に戦闘セリフなど頻繁に表示されるものは、配列で複数用意するとキャラクターが生き生きして見えます。

### パターン 3: ネストテーブル（状態分岐）

アイテムの状態に応じて異なるセリフを表示したい場合に使います。

```lua
CAMPFIRE = {
    EMBERS = "The fire is dying...",
    GENERIC = "A campfire.",
    HIGH = "Too big!",
    LOW = "Getting dim.",
    NORMAL = "Warm and cozy.",
    OUT = "It went out.",
},
```

> **重要**: ネストテーブルのキー名（`EMBERS`、`HIGH` 等）はゲームエンジンが決めた名前です。自分で新しいキーを追加しても機能しません。ウィルソンの Speech ファイルにあるキー構造をそのまま使ってください。

### パターン 4: プレースホルダー（%s）

`%s` を使うと、ゲーム実行時に動的な値（プレイヤー名やアイテム名など）に置き換えられます。

```lua
DESCRIBE = {
    PLAYER = {
        GENERIC = "Hey there, %s!",                  -- %s → プレイヤー名
        ATTACKER = "%s looks like trouble...",
    },
},
```

### エスケープ文字

セリフ内に引用符（`"`）を含めたい場合は、バックスラッシュ（`\`）でエスケープします。

```lua
DESCRIBE = {
    BOOK = "It says \"Do Not Read\" on the cover.",
},
```

> **注意**: エスケープを忘れると Lua の文法エラーになり、MOD がクラッシュします。

---

## modmain.lua での読み込み設定

Speech ファイルを作成したら、modmain.lua でゲームに読み込ませる設定が必要です。

### 基本設定

modmain.lua に以下のように記述します（既に [DST-Lua-Scripting ガイド](README.md) の手順で記述済みの場合は確認のみで OK です）。

```lua
-- セリフデータの読み込み
-- ★ MY_CHARACTER は大文字、speech_my_character は小文字であることに注意
STRINGS.CHARACTERS.MY_CHARACTER = GLOBAL.require "speech_my_character"
```

> **重要**: `STRINGS.CHARACTERS` のキーは **大文字**（`MY_CHARACTER`）、`require` のファイル名は **小文字**（`speech_my_character`）です。この対応を間違えるとセリフが表示されないか、クラッシュの原因になります。

### 他キャラクターからの呼ばれ方

マルチプレイで他のキャラクターがあなたのキャラクターを調べた時に表示されるセリフも設定します。

```lua
STRINGS.CHARACTERS.GENERIC.DESCRIBE.MY_CHARACTER = {
    GENERIC = "It's My Character!",
    ATTACKER = "That one looks dangerous...",
    MURDERER = "Murderer!",
    REVIVER = "A helpful soul.",
    GHOST = "My Character could use a heart.",
}
```

> **ポイント**: `GENERIC.DESCRIBE` は「すべてのキャラクター共通の調査セリフ」を意味します。ここに設定した内容が、他の全キャラクターがあなたのキャラクターを調べた時に表示されます。

---

## セリフ作成の実践テクニック

### キャラクターの性格設定を最初に決める

セリフを書き始める前に、キャラクターの **性格・口調** を決めておくと統一感が出ます。

- **丁寧語キャラ**: 「これは素敵なアイテムですね。」
- **乱暴なキャラ**: 「なんだこのガラクタは！」
- **怖がりなキャラ**: 「こ、これは怖くないぞ...たぶん。」
- **研究者タイプ**: 「興味深い標本だ。詳しく調べたい。」
- **食いしん坊キャラ**: 「食べられるかな？...やっぱり無理か。」

> **ヒント**: 口調を決めたら、3〜5 個のサンプルセリフを先に書いてみてください。「このキャラならどう言うか？」を考える基準になります。

### 段階的にカスタマイズする

数千行あるセリフを一度に全部書き換えるのは大変です。以下の順番で段階的に進めることをおすすめします。

#### ステージ 1: まず動かす（最小構成）

```lua
-- speech_my_character.lua
return require("speech_wilson")
```

ウィルソンのセリフをそのまま使い、MOD が正常に動くことを確認します。

#### ステージ 2: 重要なセリフだけ変更する

ウィルソンのセリフをベースにしつつ、よく目にするセリフだけ書き換えます。

特に変更すると効果が高いカテゴリ:

| カテゴリ | 理由 |
|---|---|
| `DESCRIBE.PLAYER` | マルチプレイで頻繁に見る |
| `BATTLECRY` | 戦闘のたびに表示される |
| `COMBAT_QUIT` | 戦闘撤退時に表示される |
| `ANNOUNCE_HUNGRY` | 空腹時に何度も表示される |
| `ANNOUNCE_COLD` / `ANNOUNCE_HOT` | 季節ごとに頻繁に表示される |
| `ANNOUNCE_HOUNDS` | 襲撃イベントで目立つ |
| `EAT_FOOD` | 食事のたびに表示される |
| `DESCRIBE_GENERIC` | 対応するセリフがないアイテム全部に使われる |

#### ステージ 3: DESCRIBE を充実させる

時間がある時に、アイテムや生物の調査セリフ（`DESCRIBE`）を少しずつカスタマイズしていきます。全部変える必要はありません。特にキャラクターの個性が出るものから優先的に変更しましょう。

> **初心者向けTip:** 全部のセリフを書き換えなくても、MOD は正常に動作します。ウィルソンのセリフファイルをコピーした場合、書き換えていない箇所はウィルソンのセリフがそのまま表示されるだけです。心配いりません。

### セリフの長さに注意する

ゲーム内でセリフはキャラクターの頭上に吹き出しとして表示されます。あまり長いセリフは途中で切れたり、読みにくくなります。

- **推奨**: 1 文〜2 文（英語で 10 単語程度まで）
- **NG**: 3 文以上の長いセリフ（吹き出しに収まらない可能性）

```lua
-- 良い例
DESCRIBE = {
    AXE = "A trusty axe. I like it.",
},

-- 悪い例（長すぎる）
DESCRIBE = {
    AXE = "This is a very fine axe made of the best materials I have ever seen in my entire life and I would like to keep it forever.",
},
```

---

## よくあるトラブルと解決法

### セリフが表示されない

| 症状 | 原因 | 解決法 |
|---|---|---|
| セリフが一切表示されない | modmain.lua に `STRINGS.CHARACTERS` の設定がない | modmain.lua に `STRINGS.CHARACTERS.MY_CHARACTER = GLOBAL.require "speech_my_character"` を追加 |
| セリフが一切表示されない | ファイル名と require のパスが不一致 | `scripts/speech_my_character.lua` のファイル名と `require "speech_my_character"` が一致しているか確認 |
| 特定の状況だけセリフが出ない | 該当するキーが Speech ファイルに存在しない | ウィルソンの Speech ファイルを参照して不足しているキーを追加 |

### MOD がクラッシュする

| 症状 | 原因 | 解決法 |
|---|---|---|
| `PANIC: Error loading module 'speech_my_character'` | ファイルが見つからない、または Lua 文法エラー | ファイルの存在確認 + 文法チェック（カンマ抜け、引用符の閉じ忘れ等） |
| `attempt to index a nil value` | テーブル構造のキー名が間違っている | ウィルソンの元ファイルとキー名を比較して修正 |
| 特定アイテムを調べるとクラッシュ | そのアイテムのセリフ定義に文法エラーがある | 該当箇所のカンマ、括弧、引用符を確認 |

### よくある文法エラー

#### カンマの抜け

```lua
-- NG: カンマが抜けている
DESCRIBE = {
    AXE = "An axe."      -- ← ここにカンマがない！
    PICKAXE = "A pick."
}

-- OK
DESCRIBE = {
    AXE = "An axe.",
    PICKAXE = "A pick.",
}
```

#### 引用符の閉じ忘れ

```lua
-- NG: 引用符が閉じていない
ANNOUNCE_HUNGRY = "I need to eat!,

-- OK
ANNOUNCE_HUNGRY = "I need to eat!",
```

#### 括弧の対応ミス

```lua
-- NG: 閉じ括弧が足りない
BATTLECRY = {
    GENERIC = {
        "Fight!",
        "Attack!",
    },
-- ← BATTLECRY の閉じ括弧がない！

-- OK
BATTLECRY = {
    GENERIC = {
        "Fight!",
        "Attack!",
    },
},
```

> **ヒント**: VSCode を使っている場合、括弧の上にカーソルを置くと対応する括弧がハイライトされます。この機能を使うと括弧の対応ミスを見つけやすくなります。

---

## Tips・参考情報

### 他のキャラクターの Speech ファイルを参考にする

ウィルソン以外のキャラクターの Speech ファイルも `scripts.zip` に含まれています。個性的なキャラクターの Speech ファイルを参考にすると、セリフ作成のヒントが得られます。

```
scripts/
├── speech_wilson.lua      ← ウィルソン（科学者、標準的）
├── speech_willow.lua      ← ウィロウ（火が好き）
├── speech_wendy.lua       ← ウェンディ（暗い性格）
├── speech_wolfgang.lua    ← ウルフガング（筋肉バカ）
├── speech_wickerbottom.lua ← ウィッカーボトム（知的）
├── speech_wes.lua         ← ウェス（しゃべらない）
└── ...                    ← 他のキャラクター
```

### 効率的な書き換え方法

VSCode の **検索と置換**（Ctrl+H）機能を使うと効率的に書き換えられます。

1. VSCode で `speech_my_character.lua` を開く
2. **Ctrl+H** で検索と置換パネルを開く
3. 書き換えたいセリフを検索して、新しいセリフに置換する

> **Tips**: 一括置換ではなく、1 つずつ確認しながら置換する方が安全です。意図しない箇所まで変更してしまうのを防げます。

### セリフの言語について

DST のセリフは **英語** で書くのが標準です。日本語でセリフを書くことも技術的には可能ですが、以下の点に注意してください。

- DST はゲーム内フォントの関係で、日本語が正しく表示されない場合があります
- Steam Workshop に公開する場合、英語の方が多くのプレイヤーに楽しんでもらえます
- 日本語を使う場合は、UTF-8 エンコーディングでファイルを保存してください

### このガイドで扱わないもの

| 内容 | 参照先ガイド |
|---|---|
| modmain.lua の書き方全般 | [DST-Lua-Scripting ガイド](README.md) |
| modinfo.lua のコンフィグオプション | [modinfo コンフィグガイド](README-modinfo-config.md) |
| キャラクターボイス（音声）の作成 | [DST-FMOD-Designer ガイド](../DST-FMOD-Designer/README.md) |
| AI 音声合成によるボイス生成 | [DST-Chatterbox ガイド](../DST-Chatterbox/README.md) |

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
