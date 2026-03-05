---
name: create-mod
description: |
  DST（Don't Starve Together）のキャラクター MOD を AI エージェントで新規作成するためのワークフロー。
  「MOD を作って」「キャラクター MOD を新規作成」「DST の MOD を作りたい」といったリクエストで使用する。
  キャラクター名を受け取り、スクリプト作成・仮画像配置・TEX 変換・アニメーション ZIP ビルドまで、
  人間の介入なしでゲーム内で動作する MOD を完成させる。
---

# DST キャラクター MOD 新規作成ワークフロー

AI エージェント経由でキャラクター MOD の新規作成を依頼された場合は、このスキルの手順に従って進めること。
[DST-AI-Agent/README.md](../../DST-AI-Agent/README.md) も合わせて参照すること。

## 基本方針

- AI がすべてのファイルを作成・配置し、**人間の介入なしで「ゲーム内で動作する MOD」を完成させる**。
- 画像は**ダミー画像テンプレートを使用する**。まず動く MOD を作ることを最優先する。
- 画像の差し替え（オリジナルの顔・ポートレート等）は **MOD 完成後のオプションステップ** として案内する。
- `work/` フォルダを作業領域として使用し、中間ファイル（仮画像 PNG、sample_build コピー等）をそこに配置する。
- **外部ライブラリを追加インストールしない。** すべての処理は、DST Mod Tools 同梱のツール（TextureConverter、scml.exe、buildanimation.py、同梱の Python 2.7）または Windows 標準機能（PowerShell の `System.Drawing`）のみで行う。`pip install` 等で外部パッケージを追加してはならない。

## work フォルダ

- 場所: リポジトリ直下の `work/<キャラクター名>/`
- `work/` は `.gitignore` に登録済みのため、Git にコミットされない。
- 用途: sample_build のコピー、仮画像 PNG の生成、TEX 変換の中間ファイル。

```
work/<キャラクター名>/
├── portraits/                    ← 仮画像 PNG（TEX 変換元）
│   ├── modicon.png
│   ├── <キャラクター名>_selectscreen.png
│   ├── <キャラクター名>_saveslot.png
│   ├── <キャラクター名>_bigportrait.png
│   ├── <キャラクター名>_avatar.png
│   ├── <キャラクター名>_avatar_ghost.png
│   └── <キャラクター名>_map_icon.png
├── build.scml                    ← Spriter プロジェクト（sample_build からコピー）
├── face/                         ← 顔パーツ（sample_build からコピー）
├── hair/                         ← 髪パーツ
├── torso/                        ← 胴体パーツ
└── ...（その他の sample_build パーツフォルダ）
```

## Phase 1: スクリプトとフォルダ構成の作成

1. **MOD フォルダを作成する**
   - 場所: `C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\mods\<キャラクター名>\`
   - サブフォルダ: `anim/`, `bigportraits/`, `images/avatars/`, `images/map_icons/`, `images/saveslot_portraits/`, `images/selectscreen_portraits/`, `scripts/prefabs/`, `sound/`

2. **Lua スクリプトを作成する**
   - `modinfo.lua` — MOD 基本情報（メインガイド README.md の Step 2 テンプレートに準拠）
   - `modmain.lua` — メインスクリプト（メインガイド README.md の Step 3 テンプレートに準拠）。**必ず `PrefabFiles` テーブルにキャラクター名を宣言すること**（宣言しないと prefab が読み込まれずゲームがフリーズする）
   - `scripts/prefabs/<キャラクター名>.lua` — キャラクター定義（メインガイド README.md の Step 4 テンプレートに準拠）
   - `scripts/speech_<キャラクター名>.lua` — セリフファイル（メインガイド README.md の Step 5 テンプレートに準拠）

3. **XML アトラスファイルを作成する**（7 個）
   - `modicon.xml`（MOD 直下）
   - `<キャラクター名>.xml`（`bigportraits/`）
   - `avatar_<キャラクター名>.xml`（`images/avatars/`）
   - `avatar_ghost_<キャラクター名>.xml`（`images/avatars/`）
   - `<キャラクター名>.xml`（`images/map_icons/`）
   - `<キャラクター名>.xml`（`images/saveslot_portraits/`）
   - `<キャラクター名>.xml`（`images/selectscreen_portraits/`）

## Phase 2: 仮画像の生成と TEX 変換

1. **`work/<キャラクター名>/portraits/` フォルダを作成する**

2. **ダミーポートレート画像を `work/<キャラクター名>/portraits/` に配置する**
   - `DST-AI-Agent/dummy_portraits/` にダミーのポートレート画像が用意されている。これは人間が作成したテンプレートで、ユーザーが後から自分で描く際の参考になる。
   - 以下のファイルをコピーし、キャラクター名にリネームして配置する。**AI が独自に仮画像を生成してはならない。**

   | コピー元（dummy_portraits/） | コピー先（portraits/） | サイズ | 用途 |
   |---|---|---|---|
   | `modicon.png` | `modicon.png` | 128 x 128 | MOD アイコン |
   | `selectscreen.png` | `<キャラクター名>_selectscreen.png` | 256 x 512 | キャラクター選択画面 |
   | `saveslot.png` | `<キャラクター名>_saveslot.png` | 128 x 128 | セーブスロット |
   | `bigportrait.png` | `<キャラクター名>_bigportrait.png` | 512 x 512 | 大ポートレート |
   | `avatar.png` | `<キャラクター名>_avatar.png` | 64 x 64 | アバター |
   | `avatar_ghost.png` | `<キャラクター名>_avatar_ghost.png` | 64 x 64 | ゴーストアバター |
   | `map_icon.png` | `<キャラクター名>_map_icon.png` | 64 x 64 | マップアイコン |

3. **アニメーションのキャラクター固有パーツをダミー画像に差し替える**
   - sample_build をコピーした時点では Wilson の画像が入っている。このまま使うとゲーム内・キャラクター選択画面ともに Wilson の外見が表示されてしまう。
   - `DST-AI-Agent/dummy_parts/` に以下のダミー画像フォルダが用意されている。これらは輪郭と表情の参考になるよう人間が作成したもので、ユーザーが後から自分で描く際のテンプレートになる。
     - `face/` — 顔パーツ（face-0.png 〜 face-13.png）
     - `hair/` — 髪パーツ
     - `hair_hat/` — 帽子着用時の髪パーツ
     - `headbase/` — 頭の輪郭（素頭）
     - `headbase_hat/` — 頭の輪郭（帽子着用時）
   - `DST-AI-Agent/dummy_parts/` 内の各フォルダの全 PNG を `work/<キャラクター名>/` の対応するフォルダに上書きコピーする。**AI が独自に仮画像を生成してはならない。**

4. **TextureConverter で全ポートレート PNG を TEX に変換する**
   - TextureConverter のパス: `C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\tools\bin\TextureConverter.exe`
   - コマンド形式:
     ```bash
     TextureConverter.exe -i <入力>.png -o <出力>.tex -f bc3 -p opengl --mipmap --premultiply
     ```

5. **変換した TEX ファイルを MOD フォルダの正しい場所に配置する**
   - `modicon.tex` → MOD 直下
   - `selectscreen` → `images/selectscreen_portraits/<キャラクター名>.tex`
   - `saveslot` → `images/saveslot_portraits/<キャラクター名>.tex`
   - `bigportrait` → `bigportraits/<キャラクター名>.tex`
   - `avatar` → `images/avatars/avatar_<キャラクター名>.tex`
   - `avatar_ghost` → `images/avatars/avatar_ghost_<キャラクター名>.tex`
   - `map_icon` → `images/map_icons/<キャラクター名>.tex`

## Phase 3: アニメーション ZIP の作成

1. **work/<キャラクター名>/ に sample_build の中身をコピーする**
   - コピー元: `C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Mod Tools\mod_tools\sample_build\`
   - コピー先: `work/<キャラクター名>/`（`build.scml` + 全パーツフォルダ）

2. **work の中身を `exported/<キャラクター名>/` にコピーし、scml ファイルをリネームする**
   - コピー先: `mod_tools\exported\<キャラクター名>\`
   - **重要: `build.scml` を `<キャラクター名>.scml` にリネームすること。** scml ファイル名がそのまま ZIP 内部の「ビルド名」になる。`MakePlayerCharacter("<キャラクター名>", ...)` はビルド名がキャラクター名と一致することを前提にしているため、ファイル名が異なるとキャラクターが透明になる。
   - work フォルダが画像編集の起点（ユーザーが見る場所）であり、exported はビルド用の一時コピーである。**画像の生成・編集は必ず work フォルダで行い、ビルド時に work → exported にコピーする**こと。exported に直接ファイルを生成・編集してはならない。

3. **scml.exe + buildanimation.py でアニメーション ZIP をビルドする**
   - autocompiler.exe は GUI ツールのため AI エージェントからは安定して動作しない場合がある。代わりに手動ビルドコマンドを使用する。
   ```bash
   MOD_TOOLS="C:/Program Files (x86)/Steam/steamapps/common/Don't Starve Mod Tools/mod_tools"
   cd "$MOD_TOOLS"
   # Step 1: scml.exe で中間 ZIP を生成
   ./scml.exe "exported/<キャラクター名>/<キャラクター名>.scml" "exported/<キャラクター名>"
   # Step 2: buildanimation.py で最終 ZIP を生成
   ./buildtools/windows/Python27/python.exe tools/scripts/buildanimation.py \
     "exported/<キャラクター名>/<キャラクター名>.zip" \
     --outputdir="exported/<キャラクター名>" --force --ignoreexceptions
   ```
   - 生成された `mod_tools\exported\<キャラクター名>\anim\<キャラクター名>.zip` を MOD の `anim/<キャラクター名>.zip` にコピーする。

4. **ゴーストアニメーション ZIP を配置する**
   - コピー元: `C:\Program Files (x86)\Steam\steamapps\common\Don't Starve Together\data\anim\ghost_wilson_build.zip`
   - コピー先: MOD の `anim/ghost_<キャラクター名>_build.zip`

## Phase 4: 最終確認

1. MOD フォルダ内の全ファイルの存在を確認する。
2. `modmain.lua` の `Assets` テーブルに記載された全ファイルが実在するか検証する。
3. ファイル名の大文字・小文字の一致を確認する。
4. 完成したフォルダ構成を人間に報告する。

## Phase 5: 画像の差し替え（オプション・MOD 完成後）

MOD が動作する状態になった後、人間がオリジナルの画像に差し替えたい場合は以下の流れで進める。

### ポートレート等の画像差し替え

1. AI が `work/<キャラクター名>/portraits/` 内のファイルのフルパスとサイズ一覧を人間に提示し、「このファイルを画像編集ソフトで編集してください」と促す。
2. 人間が画像編集ソフト（ペイント、GIMP 等）でファイルを修正する。
3. 人間が AI に「画像を差し替えたから圧縮してファイルを差し替えて」と指示する。
4. AI が TextureConverter で TEX に再変換し、MOD フォルダの正しい場所に上書き配置する。

### アニメーション画像の差し替え

1. AI が `work/<キャラクター名>/` 内の sample_build ファイル（特に `face/` フォルダ）のフルパスを人間に提示し、「このファイルを画像編集ソフトで編集してください」と促す。
2. 人間が画像編集ソフトでファイルを修正する。
3. 人間が AI に「画像を差し替えたから圧縮してファイルを差し替えて」と指示する。
4. AI が autocompiler で ZIP を再ビルドし、MOD の `anim/<キャラクター名>.zip` を上書きする。
