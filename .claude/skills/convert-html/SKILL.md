---
name: convert-html
description: |
  MD ファイルを HTML に変換する、または既存の HTML を MD の変更に合わせて更新するためのルール。
  「HTML に変換して」「HTML 版を作って」「MD を更新したから HTML も直して」「HTML を同期して」
  といったリクエストで使用する。MD と HTML の 1:1 対応を維持し、統一されたデザインシステムで
  HTML を生成・更新する。
---

# MD → HTML 変換ルール

## MD と HTML の 1:1 対応

- すべての `.md` ファイルに対応する `.html` ファイルを同じディレクトリに配置する。
- MD を修正したら、**必ず対応する HTML にも同じ内容を反映する**。
- HTML の新規作成時は、既存の HTML（例: DST-TextureConverter/README.html）のスタイルを参照して同じデザインシステムで作成する。

## HTML 共通仕様

- `<!DOCTYPE html>`, `<html lang="ja">`, UTF-8
- Google Fonts: `Noto Serif JP`（400, 700）, `Noto Sans JP`（300, 400, 500, 600）
- CSS カスタムプロパティによるウォームカラーパレット（`--bg-primary: #faf9f7` 等）
- フッター形式（全ファイル統一）:
  ```
  ガイド名 &#8212; このドキュメントは自由に利用・改変・再配布できます。DST MOD 制作にお役立てください。
  Author: pinpikokun ｜ GitHub ｜ Steam Workshop（shields.io バッジ画像）
  ```

## UI ナビゲーションパス（縦タイムライン方式）

UI 操作の手順（「Steam → 設定 → ストレージ」のような矢印で繋ぐナビゲーション）は、HTML では縦タイムライン方式で記述する。

```html
<ol class="nav-path">
  <li>Steam を開く</li>
  <li>Don't Starve Together を右クリック</li>
  <li><strong>管理</strong> を選択</li>
  <li><strong>ローカルファイルを閲覧</strong> をクリック</li>
  <li><code>mods</code> フォルダを開く</li>
</ol>
```

**CSS 仕様（`.nav-path`）:**

- 左側に縦線（`var(--border)`、2px）とドット（`var(--accent-warm)`、10px 円）を配置
- 各ステップは縦に並び、ドットが縦線上に表示される
- `position: absolute` で `::before`（縦線）と `::after`（ドット）を配置
- 先頭・末尾のアイテムで縦線の開始・終了位置を調整

**適用基準:**

- 3 ステップ以上の UI 操作手順に適用する
- 2 ステップ以下の短い手順はそのままでよい

## 変換の手順

1. 対象の MD ファイルを読む
2. 既存の HTML ファイルがあればそれも読み、デザインシステム（CSS 変数、フォント、レイアウト）を確認する
3. 新規作成の場合は、同じリポジトリ内の既存 HTML（例: `DST-TextureConverter/README.html`）をテンプレートとして参照する
4. MD の内容を HTML に変換し、上記の共通仕様に従って出力する
5. 変換後、MD と HTML の内容に差異がないか確認する
