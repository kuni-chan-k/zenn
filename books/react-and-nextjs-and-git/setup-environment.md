---
title: "開発環境を整えよう"
---

## この章でやること

- Node.js のインストール
- VSCode のインストールと設定
- ターミナルの基本操作

## Node.js とは

Node.js は、JavaScript をパソコン上で動かすためのソフトウェアです。

通常、JavaScript はブラウザ（Chrome や Safari）の中でしか動きませんが、Node.js をインストールすると、パソコン上で直接 JavaScript を実行できるようになります。

React の開発には Node.js が必須なので、まずはこれをインストールしましょう。

## Node.js のインストール

### Windows の場合

1. [Node.js 公式サイト](https://nodejs.org/ja/) にアクセス
2. 「LTS」と書かれたボタンをクリックしてダウンロード
3. ダウンロードしたファイルを開いてインストール（すべて「次へ」でOK）

### Mac の場合

1. [Node.js 公式サイト](https://nodejs.org/ja/) にアクセス
2. 「LTS」と書かれたボタンをクリックしてダウンロード
3. ダウンロードしたファイルを開いてインストール

### インストール確認

ターミナル（Mac）またはコマンドプロンプト（Windows）を開いて、以下を入力してください。

```bash
node -v
```

`v20.x.x` のようなバージョン番号が表示されれば成功です。

## VSCode のインストール

VSCode（Visual Studio Code）は、プログラムを書くためのエディタです。無料で使えて、多くの開発者が使っています。

### インストール手順

1. [VSCode 公式サイト](https://code.visualstudio.com/) にアクセス
2. 「Download」ボタンをクリック
3. ダウンロードしたファイルを開いてインストール

### おすすめの拡張機能

VSCode を開いたら、左側の「拡張機能」アイコン（四角が4つ並んだマーク）をクリックして、以下を検索してインストールしてください。

- **Japanese Language Pack** - VSCode を日本語化
- **ES7+ React/Redux/React-Native snippets** - React のコード補完
- **Prettier** - コードを自動整形

## ターミナルの基本操作

ターミナル（Mac）やコマンドプロンプト（Windows）は、文字でパソコンを操作するツールです。

React 開発では頻繁に使うので、基本的な操作を覚えておきましょう。

### ターミナルの開き方

**Mac の場合**
- Spotlight（Cmd + Space）で「ターミナル」と検索

**Windows の場合**
- スタートメニューで「cmd」と検索して「コマンドプロンプト」を開く
- または「PowerShell」でもOK

### よく使うコマンド

| コマンド | 意味 | 例 |
|---------|------|-----|
| `cd フォルダ名` | フォルダに移動 | `cd Desktop` |
| `cd ..` | 一つ上のフォルダに移動 | `cd ..` |
| `ls`（Mac）/ `dir`（Windows） | ファイル一覧を表示 | `ls` / `dir` |
| `mkdir フォルダ名` | フォルダを作成 | `mkdir my-project` |

:::message
Mac と Windows でコマンドが異なる場合があります。Mac は `ls`、Windows は `dir` でファイル一覧を表示します。
:::

### 練習してみよう

1. ターミナルを開く
2. `cd Desktop` でデスクトップに移動
3. `mkdir react-practice` でフォルダを作成
4. `cd react-practice` で作成したフォルダに移動
5. `ls`（Mac）または `dir`（Windows）で中身を確認（まだ空のはず）

## この章のまとめ

- Node.js をインストールした
- VSCode をインストールして拡張機能を追加した
- ターミナルの基本操作を学んだ

## 確認してみよう

- [ ] `node -v` でバージョンが表示される
- [ ] VSCode が起動する
- [ ] ターミナルで `cd` と `mkdir` が使える
