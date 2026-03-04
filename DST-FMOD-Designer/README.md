# DST FMOD Designer テンプレート

Don't Starve Together (DST) キャラクター MOD 用の FMOD Designer プロジェクトテンプレートです。
キャラクターボイスの定義からサウンドバンクのビルドまでの手順を解説します。

> **このテンプレートの使い方:** このドキュメント内で `my_character` と表記されている部分は、あなたの MOD 名に置き換えてください。
> 例: MOD名が `awesome_knight` なら、`DST-my_character` → `DST-awesome_knight` のように読み替えます。

> **注意:** リポジトリに含まれている `DST-my_character.fdp` はサンプルファイル（構成の参考用）です。実際の MOD 制作では、このファイルを直接使わず **FMOD Designer で新規プロジェクトを作成** してください。手順は「セットアップ手順」セクションを参照してください。

---

## 必要なツール

- **FMOD Designer**（バージョン 4 プロジェクト形式）

> **注意:** 「FMOD Studio」ではありません。DST が対応しているのは旧バージョンの **FMOD Designer** です。間違えやすいので注意してください。

### FMOD Designer のインストール方法

Steam から入手します。

1. Steam ライブラリを開き、フィルタで **「ツール」** を選択する
2. **「Don't Starve Mod Tools」** をインストールする
3. インストール後、Don't Starve Mod Tools を起動すると使用するツールの選択画面が表示されるので **FMOD Designer** を選ぶ

> **ヒント:** Steam ライブラリのフィルタで「ツール」が見つからない場合は、ライブラリ上部の検索欄に「Don't Starve Mod Tools」と入力してください。

---

## プロジェクト構造

```
DST-FMOD-Designer/
├── DST-my_character.fdp           # FMOD Designer プロジェクトファイル（メイン）
├── DST-my_character.fev           # ビルド済みイベントファイル（自動生成）
├── DST-my_character_bank00.fsb    # ビルド済みサウンドバンク（自動生成）
├── README.html                # HTML 版ドキュメント（詳細・図表付き）
├── README.md                  # このドキュメント
├── .gitignore
├── .fsbcache/                 # FMOD のビルドキャッシュ（自動生成・削除可）
└── wav/                       # WAV ファイル格納フォルダ
    ├── talk_LP/               # 通常会話ボイス（DST 標準）
    ├── hurt/                  # ダメージ時ボイス（DST 標準）
    ├── ghost_LP/              # 幽霊状態ボイス（DST 標準）
    ├── emote/                 # エモートボイス（DST 標準）
    ├── death_voice/           # 死亡時ボイス（DST 標準）
    ├── attack/                # 攻撃時ボイス（独自追加）
    └── spwn/                  # ゲーム開始・復活時ボイス（独自追加）
```

> **WAV ファイルについて:** 各 `wav/` サブフォルダに、そのイベントで再生したい WAV ファイルを配置してください。複数ファイルを配置すると、再生時にランダムで選ばれます。

> **ヒント:** 上記のイベントはあくまで一例です。`attack` や `spwn` のように、MOD の Lua コードから `SoundEmitter` で呼び出せば任意のアクションに対して独自のイベントを自由に追加できます。例えば「特殊能力発動時」「アイテム使用時」など、好きなタイミングでボイスを再生させることが可能です。

---

## セットアップ手順

このテンプレートをあなたの MOD 用にカスタマイズする手順です。

### 1. WAV ファイルの配置

各イベントに対応する `wav/` サブフォルダに WAV ファイルを配置します。

- フォーマット: **WAV**（16bit / 44100Hz または 48000Hz 推奨）
- 各フォルダに **1 つ以上** の WAV ファイルが必要です
- 複数配置するとランダムに選ばれるため、バリエーションを増やすと自然に聞こえます

> **ヒント:** WAV ファイル名に日本語や空白を含めないでください。半角英数字とアンダースコアのみを使用することを推奨します（例: `attack_01.wav`）。

### 2. FMOD Designer で新規プロジェクトを作成する

1. Steam ライブラリから **Don't Starve Mod Tools** を起動し、ツール選択画面で **FMOD Designer** を選ぶ
2. FMOD Designer が起動したら、左上メニューから **File → New Project** を選択する
3. 保存先とファイル名を指定してプロジェクトを作成する（例: `DST-my_character.fdp`）
4. 新規プロジェクト作成時に `untitled` というイベントグループが自動生成されているので、右クリック → **Delete** で削除する

> **ヒント:** FMOD Designer は Don't Starve Mod Tools 経由でのみ起動できます。単体のインストーラーは提供されていません。

> **メモ:** プロジェクトのファイル名に特定の命名規則はありません。`DST-` プレフィックスは必須ではないので `my_character.fdp` のように自由に命名できます。

### 3. プロジェクト構成の設定

以下の「FMOD Designer プロジェクト構成」セクションに従って設定を行います。

---

## FMOD Designer プロジェクト構成設定

ゲームで声を正しく再生させるには、以下の構成を **正確に** 設定する必要があります。

### イベントグループ階層

FMOD Designer の Event Groups パネルで、以下のツリー構造を作成します。

> **メモ:** 新規プロジェクト作成時に `untitled` というイベントグループが自動生成されています。これは不要なので、右クリック → **Delete** で削除してください。

> **操作方法:**
> - **Event Group の追加:** 「Groups」タブ内で右クリック → **Add Event Group** を選択
> - **Simple Event の追加:** 作成した Event Group フォルダの上で右クリック → **Add Simple Event** を選択

```
dontstarve/                                  ← Event Group（DST 固定。名前を変えないこと）
  └── characters/                            ← Event Group（DST 固定。名前を変えないこと）
        └── DST-my_character/                    ← Event Group（あなたの MOD 名）
              ├── talk_LP                    ← Simple Event
              ├── hurt                       ← Simple Event
              ├── ghost_LP                   ← Simple Event
              ├── emote                      ← Simple Event
              ├── death_voice                ← Simple Event
              ├── attack                     ← Simple Event
              └── spwn                       ← Simple Event
```

> **重要:** グループ名・イベント名の大文字小文字・スペルが 1 文字でも違うとゲームで音が鳴らなくなります。
> 特に `dontstarve`（小文字・スペースなし）と `characters` は固定です。変更しないでください。

> **初心者向け:** 3 階層目の `DST-my_character` は、あなたの MOD の Lua コードで指定するキャラクター名と一致させてください。
> 具体的には `modmain.lua` などで FMOD サウンドパスとして参照される名前です。

### WAV ファイルをイベントに追加する

作成した Simple Event に WAV ファイルを割り当てます。

1. 左ペインの「Groups」タブで、追加したい **Simple Event**（例: `talk_LP`）をクリックして選択する
2. 画面中央のペインに **Playlist** が表示される
3. エクスプローラー等から WAV ファイルを **Playlist にドラッグ＆ドロップ** する

> **ヒント:** 1 つのイベントに複数の WAV ファイルを登録できます。複数登録すると、再生時に Playlist Behavior の設定（後述）に従ってどれが再生されるか選ばれます。

### サウンドバンク

サウンドバンクとは、イベントで使用する WAV ファイルをひとつの `.fsb` ファイルにまとめてパッケージ化したものです。
ゲームはこの `.fsb` ファイルから音声データを読み込むため、**すべてのイベントをサウンドバンクに割り当てないとゲーム内で音が鳴りません。**

> **注意:** サウンドバンクはプロジェクト作成時に自動で作成されます。**バンク名を手動で変更しないでください。** 名前を変えるとビルド時に生成される `.fsb` ファイル名も変わり、ゲーム側で正しく読み込めなくなる場合があります。

イメージとしては以下のような関係です：

```
WAV ファイル群 → サウンドバンク（.fsb）に格納 → ゲームが .fsb を読み込んで再生
```

イベントに WAV ファイルを追加すると、その WAV ファイルは自動的にサウンドバンクに登録されます。基本的に手動での操作は不要です。

自動生成されるバンク名はプロジェクトファイル名に基づきます（例: `DST-my_character.fdp` → `DST-my_character_bank00`）。

サウンドバンクの設定値（参考）：

| 項目 | 値 |
|---|---|
| バンク名 | `DST-my_character_bank00`（自動生成） |
| フォーマット | PCM |
| サンプルレート | 48000 Hz |
| バンクタイプ | DecompressedSample |

> **メモ:** バンク名を変更したい場合は、画面上部の **Banks** タブをクリック → 左ペインで変更したいバンク名を選択 → 右ペインの **Name** 欄で変更できます。ただし、変更するとビルド後の `.fsb` ファイル名も変わるため、MOD の Lua コード側で参照するパスも合わせて変更する必要があります。

### 各イベントの設定

> **私はPlayback Mode（ループ設定）と Playlist Behavior（ランダム設定）、Volume（音量）以外はすべてデフォルト値のままで利用しています。細かい設定がしたい場合は調べるか、実際に試して聞き比べて調整してください。**

#### Volume について

> **操作方法:** Simple Event 名（`talk_LP` 等）を選択し、一番右側のペインの上から 4 つ目に Volume の項目があります。

Volume のデフォルトは 0 dB（最大音量）ですが、ゲーム内の他のサウンドとのバランスを考慮して **-10 dB 程度** に下げることを推奨します。
実際の値は WAV ファイルの録音レベルによって調整してください。ゲーム内で実際に聴きながら微調整するのがベストです。

#### Playback Mode（イベントごとに異なる）

| イベント名 | 推奨 Playback Mode | 説明 |
|---|---|---|
| `talk_LP` | **※ 下記参照** | ボイスの種類によって異なる |
| `hurt` | **Oneshot** | ダメージ時に 1 回再生 |
| `ghost_LP` | **Repeating Loop** | 幽霊状態中ずっとループ再生 |
| `emote` | **Oneshot** | エモート時に 1 回再生 |
| `death_voice` | **Oneshot** | 死亡時に 1 回再生 |
| `attack` | **Oneshot** | 攻撃時に 1 回再生 |
| `spwn` | **Oneshot** | ゲーム開始・復活時に 1 回再生 |

> **ヒント:** イベント名に `_LP` が付いているもの（`talk_LP`、`ghost_LP`）の `LP` は **Loop（ループ）** の略で、DST 側が繰り返し再生を想定しているイベントであることを意味します。

#### talk_LP の Playback Mode 選択ガイド

DST の標準キャラクターでは `_LP` サフィックスを持つイベント（`talk_LP`、`ghost_LP`）は通常 **Repeating Loop** を使用します。
これは標準キャラクターの「声」が楽器音のような短いフレーズのループだからです。

しかし、**ボイスライン（実際のセリフ音声）** を使う場合は `talk_LP` を **Oneshot** に設定してください。
ループにすると同じセリフが延々と繰り返されてしまいます。

| ボイスの種類 | 推奨設定 | 例 |
|---|---|---|
| 楽器音・効果音系 | **Repeating Loop** | ピコピコ音、楽器の音フレーズなど（DST 標準キャラと同じ方式） |
| セリフ・ボイスライン系 | **Oneshot** | 実際の音声セリフ、録音された声など |

#### Playlist Behavior について

Playlist Behavior は、イベントに複数の WAV ファイルが登録されている場合の再生順序を決める設定です。

| 設定値 | 動作 |
|---|---|
| **Random** | 毎回完全ランダムに選ぶ。同じ音が連続して選ばれることがある |
| **Shuffle** | ランダムだが、全ファイルを一巡するまで同じ音を繰り返さない（トランプのシャッフルと同じ概念） |
| **Sequential** | 登録順に順番に再生する |

**Shuffle** を推奨します。同じボイスが連続して鳴るのを防ぎつつ、出現順が固定にならないため自然に聞こえます。

> **初心者向け:** Playlist Behavior の設定場所が分かりにくいです。イベントを選択した状態で、画面下部の Sound Definition（サウンド定義）エリアに表示されるプロパティ内にあります。

---

## 各イベントの説明

各イベントがゲーム内でいつ再生されるかの一覧です。`wav/` の各サブフォルダに WAV ファイルを配置し、FMOD Designer でイベントに割り当ててください。

| イベント名 | 再生タイミング | WAV 配置先 |
|---|---|---|
| `talk_LP` | キャラクターの吹き出しが表示されたとき（最も頻繁に使用） | `wav/talk_LP/` |
| `hurt` | キャラクターがダメージを受けたとき | `wav/hurt/` |
| `ghost_LP` | 死亡後の幽霊状態中（ループ再生） | `wav/ghost_LP/` |
| `emote` | プレイヤーがエモートアクションを実行したとき | `wav/emote/` |
| `death_voice` | キャラクターが死亡した瞬間 | `wav/death_voice/` |
| `attack` | キャラクターが攻撃したとき | `wav/attack/` |
| `spwn` | ゲーム開始時やリスポーン時 | `wav/spwn/` |

> **ヒント:** すべてのイベントに WAV を割り当てる必要はありません。使わないイベントはそのまま空にしておけば、ゲーム内でそのタイミングでは音が鳴らないだけです。
> まずは `talk_LP` と `hurt` だけ設定して動作確認するのがおすすめです。

---

## カスタムイベントの追加

上記の標準イベント以外にも、MOD の Lua コードと連携すれば独自のイベントを追加できます。
例: 特定のアイテム使用時、特殊能力発動時など。追加したイベントを Lua 側から `SoundEmitter` で呼び出すことで再生できます。
このテンプレートでは `attack`（攻撃時）と `spwn`（ゲーム開始・復活時）が独自追加イベントの例にあたります。
詳しい Lua コードの書き方は「[付録: SoundEmitter の Lua コーディング例](#付録-soundemitter-の-lua-コーディング例)」を参照してください。

### プロジェクトの保存

すべての設定が完了したら、左上メニューの **File → Save Project** でプロジェクトを保存してください。

---

## ビルド手順

1. FMOD Designer で `DST-my_character.fdp` を開く
2. メニューから **File > Build** を実行する
3. 生成された `.fev` と `_bank00.fsb` を MOD の `sound/` フォルダにコピーする
4. メニューから **File > Save** でプロジェクトを保存し、FMOD Designer を閉じる

> **ビルドの度に** `.fev` と `.fsb` ファイルが更新されます。WAV ファイルの追加・変更後は必ず再ビルドしてください。

---

## 出力ファイルの配置先

ビルドで生成されたファイルを、MOD フォルダの `sound/` ディレクトリに配置します。

```
your_mod_folder/
└── sound/
    ├── DST-my_character.fev
    └── DST-my_character_bank00.fsb
```

> **初心者向け:** MOD フォルダは通常、Steam の DST ワークショップフォルダ内にあります。
> `sound/` フォルダが存在しない場合は、自分で作成してください。

---

## よくあるトラブルと対処法

| 症状 | 原因 | 対処法 |
|---|---|---|
| ゲーム内で音が鳴らない | イベントグループ階層の名前が間違っている | `dontstarve/characters/あなたのMOD名/` が正確か確認。大文字小文字も完全一致が必要 |
| ゲーム内で音が鳴らない | `.fev` / `.fsb` を MOD の `sound/` にコピーし忘れ | ビルド後のファイルを MOD フォルダにコピーしたか確認 |
| ゲーム内で音が鳴らない | WAV ファイルがサウンドバンクに含まれていない | Sound Banks パネルでイベントがバンクに割り当てられているか確認 |
| ビルド時にエラーが出る | WAV ファイルのパスが壊れている | WAV ファイルが `wav/` フォルダ内に存在するか確認。ファイル名を変更した場合は再度割り当てが必要 |
| 同じ音声が連続で再生される | Playlist Behavior が Random になっている | **Shuffle** に変更する |
| 音量が大きすぎる / 小さすぎる | Volume の設定値が適切でない | イベントの Volume を調整する（推奨: -10 dB 前後） |
| talk_LP が延々ループする | Playback Mode が Repeating Loop になっている | ボイスライン系の場合は **Oneshot** に変更する |

---

## 付録: SoundEmitter の Lua コーディング例

DST で FMOD イベントを再生するには、エンティティの `SoundEmitter` コンポーネントを使用します。

### 基本: サウンドバンクの読み込み

MOD の初期化時に、ビルドした `.fev` ファイルをゲームに登録する必要があります。

```lua
-- modmain.lua
RemapSoundEvent("dontstarve/characters/my_character", "DST-my_character")
```

> **注意:** 第 1 引数はイベントグループのパス、第 2 引数は `.fev` ファイル名（拡張子なし）です。

### 基本: 1 回だけ再生する（PlaySound）

```lua
-- inst は対象のエンティティ（キャラクターなど）
inst.SoundEmitter:PlaySound("dontstarve/characters/my_character/attack")
```

`PlaySound` は指定したイベントの音声を 1 回再生します。Oneshot のイベントに使用します。

### 基本: ループ再生する（PlaySound + 名前付き）

```lua
-- ループ再生を開始（名前を付けて管理する）
inst.SoundEmitter:PlaySound("dontstarve/characters/my_character/talk_LP", "talking")

-- ループ再生を停止
inst.SoundEmitter:KillSound("talking")
```

ループイベントを再生する場合は、第 2 引数に名前（任意の文字列）を付けます。停止するときは `KillSound` にその名前を渡します。

### 実用例: 攻撃時にボイスを再生する

```lua
-- modmain.lua
local function OnAttack(inst, data)
    inst.SoundEmitter:PlaySound("dontstarve/characters/my_character/attack")
end

AddPrefabPostInit("my_character", function(inst)
    if not TheWorld.ismastersim then return end
    inst:ListenForEvent("onattackother", OnAttack)
end)
```

### 実用例: スポーン時にボイスを再生する

```lua
-- modmain.lua
local function OnSpawn(inst)
    inst.SoundEmitter:PlaySound("dontstarve/characters/my_character/spwn")
end

AddPrefabPostInit("my_character", function(inst)
    if not TheWorld.ismastersim then return end
    inst:DoTaskInTime(0, OnSpawn)  -- 1フレーム後に再生（初期化完了を待つ）
end)
```

### SoundEmitter 主要メソッド一覧

| メソッド | 説明 |
|---|---|
| `PlaySound(event, name, volume)` | イベントを再生する。`name` はループ管理用（省略可）。`volume` は 0〜1 の範囲（省略可） |
| `KillSound(name)` | `name` で指定したループ再生を停止する |
| `KillAllSounds()` | エンティティの全サウンドを停止する |
| `SetVolume(name, volume)` | 再生中のサウンドの音量を変更する（0〜1） |

### イベントパスの構造

```
dontstarve/characters/<MOD名>/<イベント名>
```

- `dontstarve/characters/` — DST 固定のプレフィックス
- `<MOD名>` — FMOD Designer の 3 階層目のイベントグループ名
- `<イベント名>` — Simple Event の名前（`talk_LP`、`attack` など）

> **重要:** このパスは FMOD Designer のイベントグループ階層と **完全に一致** させる必要があります。1 文字でも違うと音が鳴りません。

---

## ライセンス

このドキュメントは自由に利用・改変・再配布できます。DST MOD 制作にお役立てください。

---

<sub>Author: [pinpikokun](https://steamcommunity.com/profiles/76561198076111536/)</sub><br>
<sub>[![GitHub](https://img.shields.io/badge/GitHub-Repository-black?logo=github)](https://github.com/pinpikokun/DST-teemo) [![Steam Workshop](https://img.shields.io/badge/Steam%20Workshop-Subscribe-blue?logo=steam)](https://steamcommunity.com/sharedfiles/filedetails/?id=390684095)</sub>
