---
title: "Gitの基礎"
---

## この章でやること

- Git とは何かを理解する
- Git のインストール
- 基本操作（init, add, commit）を覚える

## Git とは

Git は、**コードの変更履歴を管理する**ツールです。

Word で文書を書いていて「昨日の状態に戻したい」と思ったことはありませんか？Git を使うと、コードを好きなタイミングで「保存ポイント」として記録し、いつでもその時点に戻れます。

### なぜ Git を使うのか

1. **やり直しができる** - 間違えても過去の状態に戻せる
2. **変更履歴が残る** - いつ、何を変えたかがわかる
3. **チーム開発で必須** - 複数人で同じコードを編集できる

個人で練習するときも Git を使う習慣をつけておくと、後で役立ちます。

## Git のインストール

### Windows の場合

1. [Git 公式サイト](https://git-scm.com/) にアクセス
2. 「Download for Windows」をクリック
3. ダウンロードしたファイルを開いてインストール（すべてデフォルトでOK）

### Mac の場合

Mac には Git が最初から入っていることが多いです。ターミナルで以下を実行してください。

```bash
git --version
```

バージョンが表示されればインストール済みです。表示されない場合は、以下を実行するとインストールが始まります。

```bash
xcode-select --install
```

## Git の初期設定

Git を使う前に、自分の名前とメールアドレスを設定します。これは「誰がコードを変更したか」を記録するためです。

ターミナルで以下を実行してください（名前とメールは自分のものに変えてください）。

```bash
git config --global user.name "あなたの名前"
git config --global user.email "your-email@example.com"
```

## Git の基本操作

Git の操作は、主に3つのステップで行います。

### 1. リポジトリの作成（init）

「リポジトリ」は、Git で管理するフォルダのことです。

```bash
# 練習用フォルダを作成
mkdir git-practice
cd git-practice

# Git リポジトリとして初期化
git init
```

`Initialized empty Git repository...` と表示されれば成功です。

### 2. 変更をステージング（add）

ファイルを作成・変更したら、まず「ステージング」という準備エリアに追加します。

```bash
# ファイルを作成（Mac/Linux）
echo "Hello Git" > hello.txt

# Windows の場合は VSCode でファイルを作成してください

# ステージングに追加
git add hello.txt
```

### 3. 変更を記録（commit）

ステージングしたファイルを「コミット」して、変更を確定します。

```bash
git commit -m "最初のコミット"
```

`-m` の後ろに、変更内容を表すメッセージを書きます。

## 状態を確認する（status）

今の状態を確認するには `git status` を使います。

```bash
git status
```

- **赤い文字** - まだステージングされていないファイル
- **緑の文字** - ステージングされたファイル（コミット待ち）
- **nothing to commit** - すべてコミット済み

## 変更履歴を見る（log）

これまでのコミット履歴を見るには `git log` を使います。

```bash
git log
```

`q` キーで終了できます。

## 実際にやってみよう

以下の手順で練習してみましょう。

1. `git-practice` フォルダに `index.html` を作成

```html
<!DOCTYPE html>
<html>
<head>
  <title>Git Practice</title>
</head>
<body>
  <h1>Hello!</h1>
</body>
</html>
```

2. Git で記録

```bash
git add index.html
git commit -m "index.htmlを追加"
```

3. ファイルを変更

```html
<!DOCTYPE html>
<html>
<head>
  <title>Git Practice</title>
</head>
<body>
  <h1>Hello World!</h1>
  <p>Git の練習中です</p>
</body>
</html>
```

4. 変更を記録

```bash
git add index.html
git commit -m "見出しとテキストを追加"
```

5. 履歴を確認

```bash
git log
```

2つのコミットが記録されているはずです。

## この章のまとめ

- Git はコードの変更履歴を管理するツール
- `git init` でリポジトリを作成
- `git add` で変更をステージング
- `git commit -m "メッセージ"` で変更を記録
- `git status` で状態確認、`git log` で履歴確認

## 確認してみよう

- [ ] `git --version` でバージョンが表示される
- [ ] 練習用フォルダで `git init` ができた
- [ ] ファイルを作成して `git add` → `git commit` できた
- [ ] `git log` で履歴が見られる

:::message
この本では Git の基本操作のみを扱います。GitHub（インターネット上にコードを保存するサービス）については、次のステップで学びましょう。
:::
