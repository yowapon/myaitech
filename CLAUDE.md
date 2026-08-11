# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## このリポジトリの性質

コードベースではなく、**公開されている学習教材のライブラリ**。ビルド工程もテストスイートも依存パッケージもない。`git push` がそのままデプロイになる（GitHub Pages が `main` / root から配信）。

- 公開URL: https://yowapon.github.io/myaitech/
- リポジトリ: https://github.com/yowapon/myaitech （**public**）

## 構成

```
index.html              # 扉ページ。分野別リンク集＋目次サイドバー
README.md               # GitHub上での入口。教材一覧テーブル
01.skills/*.zip         # 教材づくりに使う Claude Code スキルのパッケージ
02.kyozai/NNN.<テーマ>/
  ├── <名前>_企画.md     # 設計メモ。構成・図解案・出典を先に決める
  └── <名前>_教材.html   # 教材本体（1枚完結）
```

教材フォルダは追加順の連番（`001.` 〜）。**扉ページでは番号順ではなく分野別に並べる**ので、連番はあくまで作成順の記録でしかない。

## 教材の作り方

`01.skills/original-kyozai.zip` のスキル（`original-kyozai`）が正式な手順を持つ。ユーザーが教材作成を依頼したら、まずこのスキルを起動する。手順は **リサーチ → 企画.md → 教材.html** の順で、**HTMLから書き始めない**（見た目だけ整って中身が薄い教材になるため）。

### 教材HTMLの不変条件

これらは README と扉ページのフッターで公開的に宣言している内容なので、崩すと記述が嘘になる。

- **完全に自己完結**。CDN・外部スクリプト・外部フォント・外部画像を一切読み込まない。オフラインでファイルを直接開いても成立すること
- **図はインラインSVG**。色は `var(--accent)` などのCSS変数か `currentColor` を使う。16進数のハードコードはダークモードで図だけ浮く
- **図解 → 本文** の順。各セクションは図が先、文章は図の補足
- ライト／ダーク切替、目次サイドバー、3択クイズの即時採点を備える（テンプレート由来）

### 検証

```bash
python ~/.claude/skills/original-kyozai/scripts/check_kyozai.py <教材.html>
```

文字数・図の点数・目次とidの対応・クイズの正解フラグ・出典数・外部依存・SVGの色ハードコードを一括で見る。

全6本まとめて回すとき:

```bash
for f in 02.kyozai/*/*_教材.html; do echo "--- $f"; python ~/.claude/skills/original-kyozai/scripts/check_kyozai.py "$f"; done
```

**現状 003 / 005 / 006 は本文の文字数チェックだけ NG（それぞれ約6,200 / 10,000 / 11,400字）。** これはユーザーが分量増を明示的に指示した結果で、意図的な逸脱。それ以外の7項目は全教材で合格しており、文字数以外のNGが出たら本物の不備。

## 教材を追加するときの手順

`02.kyozai/` にフォルダを足すだけでは扉ページに出てこない。**3か所すべてを更新する**。

1. `02.kyozai/NNN.<テーマ>/` に `_企画.md` と `_教材.html` を作る
2. `index.html` — 該当カテゴリに `<a class="item" id="mNNN">` のカードを追加。新分野なら `<section class="cat" id="cat-xxx">` ごと追加
3. `index.html` — サイドバーの `<nav id="toc">` に `<a class="sub" href="#mNNN">` を追加
4. `index.html` — カテゴリの `<span class="count">` とヘッダーの `教材 N本` バッジを更新
5. `README.md` — 該当カテゴリのテーブルに1行、構成図のツリーにも1行

### 日本語パスの扱い

フォルダ名・ファイル名が日本語なので、`index.html` と `README.md` のリンクは**パーセントエンコードして書く**。手打ちすると必ず壊れるので生成する:

```bash
python -c "
import urllib.parse, pathlib
for p in sorted(pathlib.Path('02.kyozai').glob('*/*_教材.html')):
    print(urllib.parse.quote(p.as_posix()))
"
```

リンク切れの検証:

```bash
python -c "
import re, pathlib, urllib.parse
h = pathlib.Path('index.html').read_text(encoding='utf-8')
for href in re.findall(r'href=\"(02[^\"]+)\"', h):
    p = pathlib.Path(urllib.parse.unquote(href))
    print(('OK ' if p.exists() else 'NG '), p)
"
```

Windows 環境では Python の標準出力が cp932 になって日本語で落ちるので、`PYTHONIOENCODING=utf-8` を付けるか Bash ツール側でパイプする。

## Git 運用上の制約

**このリポジトリのローカル `user.email` は GitHub の noreply アドレスに固定されている。変更しないこと。**

```
user.name  = yowapon
user.email = 234526421+yowapon@users.noreply.github.com
```

リポジトリが public なので、生アドレスでコミットすると即座に公開される。グローバル設定が別の値でも、ローカル設定が優先されることを前提に確認する。

- 履歴の書き換え（force push / amend / rebase）は**到達不能オブジェクトを生み、GitHub上に残り続ける**。安易にやらない。公開済みのコミットを履歴から外しても消えるのは見た目だけで、実体の削除には GitHub Support への依頼が要る
- 通常のコミットは到達不能オブジェクトを作らないので、ファイルを取り消したいだけなら `git rm --cached` ＋ 新規コミットで済ませる
- `old/` はフォルダ名として `.gitignore` 済み（旧版バックアップを公開しないため）

## 公開前チェック

このリポジトリは public なので、外部から持ち込んだ教材を push する前に中身を確認する。特に他のAIツールで作った教材には、社内資料のトーンや所属を示す表現が混ざりやすい。

- メールアドレス・トークン・ローカルパス・電話番号・IPアドレス
- 会社名・所属を示す固有名詞、「弊社」「社員全員」など読者を特定組織の構成員として扱う表現
- 外部CDNの読み込み（自己完結の宣言に反する）
- 実在の企業・人物を挙げた事例の事実確認

## 文体

教材本文・企画メモ・扉ページ・README はすべて日本語。「です・ます」体で、前置きや「〜について見ていきましょう」の類は書かない。数字を出すときは必ず一次情報の出典URLを添える。

---

`~/.codex/config.toml` があります。Codex の設定（MCPサーバー・スラッシュコマンド等）を Claude Code に取り込めます。取り込む場合は `/import` と返信すると、対象がスキャンされて一覧表示されます。
