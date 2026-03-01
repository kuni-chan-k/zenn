---
title: "Next.jsプロジェクトを作ろう"
---

## この章でやること

- Next.js とは何かを理解する
- Next.js + TypeScript プロジェクトを作成する
- ディレクトリ構成を理解する

## Next.js とは

Next.js は、**React を使った Web アプリを簡単に作るためのフレームワーク**です。

React だけでもアプリは作れますが、Next.js を使うと以下のメリットがあります。

- ファイルを作るだけでページが追加できる
- 画像の最適化が自動で行われる
- サーバーでの処理も書ける
- 本番環境へのデプロイが簡単

現在、React でアプリを作る場合は Next.js を使うのが主流です。

### なぜ Next.js を使うのか？

**React だけでもアプリは作れますが...**

React だけでも Web アプリは作れますが、以下のような機能を自分で実装する必要があります。

- ページのルーティング（URL とページの対応）
- 画像の最適化
- サーバーサイドでの処理
- デプロイの設定

**Next.js を使うと...**

Next.js は、これらの機能が最初から組み込まれています。

- **ファイルベースのルーティング** - フォルダを作るだけでページが追加できる
- **自動最適化** - 画像やコードが自動で最適化される
- **フルスタック開発** - サーバー側の処理も書ける
- **簡単なデプロイ** - Vercel にデプロイするだけで公開できる

**学習コスト**

Next.js は React の知識があれば、すぐに始められます。新しい概念を覚える必要はほとんどありません。

**業界標準**

多くの企業で Next.js が採用されており、学習しておくと就職・転職にも有利です。

## プロジェクトの作成

ターミナルを開いて、プロジェクトを作りたいフォルダに移動してください。

```bash
cd Desktop  # 例: デスクトップに移動
```

次に、以下のコマンドを実行します。

```bash
npx create-next-app@latest my-first-app
```

いくつか質問されるので、以下のように答えてください。

```
✔ Would you like to use TypeScript? … Yes        ← TypeScript を使う！
✔ Would you like to use ESLint? … Yes
✔ Would you like to use Tailwind CSS? … No       ← この本では使わないので No
✔ Would you like your code inside a `src/` directory? … No
✔ Would you like to use App Router? (recommended) … Yes  ← App Router を使う
✔ Would you like to use Turbopack for `next dev`? … No または Yes（どちらでもOK）
✔ Would you like to customize the import alias (@/* by default)? … No
```

:::message
質問の内容や順序は、Next.js のバージョンによって異なる場合があります。基本的には「TypeScript を使う」「App Router を使う」を選択すれば問題ありません。
:::

:::message
TypeScript を使うと、VSCode が間違いを教えてくれたり、補完が効いたりして、開発がとても楽になります。TypeScript の基礎はこの後の章で学びます。
:::

しばらく待つと、プロジェクトが作成されます。

## プロジェクトの起動

作成したプロジェクトに移動して、開発サーバーを起動しましょう。

```bash
cd my-first-app
npm run dev
```

ターミナルに以下のような表示が出たら成功です。

```
▲ Next.js 15.x.x
- Local: http://localhost:3000
```

ブラウザで `http://localhost:3000` を開いてください。Next.js のウェルカムページが表示されれば成功です！

:::message
開発サーバーを止めるには、ターミナルで `Ctrl + C` を押します。
:::

## VSCode でプロジェクトを開く

VSCode でプロジェクトを開きましょう。

1. VSCode を起動
2. 「ファイル」→「フォルダーを開く」
3. `my-first-app` フォルダを選択

または、ターミナルで以下を実行しても開けます。

```bash
code .
```

## ディレクトリ構成

プロジェクトには様々なファイルがありますが、最初は以下だけ覚えれば大丈夫です。

```
my-first-app/
├── app/                  # ← ページを作る場所
│   ├── layout.tsx        # ← 全ページ共通のレイアウト
│   ├── page.tsx          # ← トップページ
│   └── globals.css       # ← 全体のスタイル
├── public/               # ← 画像などを置く場所
├── package.json          # ← プロジェクトの設定
├── tsconfig.json         # ← TypeScript の設定
└── ...（その他の設定ファイル）
```

### package.json とは

`package.json` は、プロジェクトの設定ファイルです。主に以下の情報が書かれています：

- **プロジェクト名とバージョン**
- **使用しているライブラリの一覧**（`dependencies`）
- **開発時に使うツール**（`devDependencies`）
- **実行できるコマンド**（`scripts`）

```json
{
  "name": "my-first-app",
  "version": "0.1.0",
  "dependencies": {
    "react": "^18.2.0",
    "next": "15.0.0"
  },
  "scripts": {
    "dev": "next dev",
    "build": "next build"
  }
}
```

**npm とは？**
- Node.js と一緒にインストールされるパッケージマネージャーです
- `npm install` でライブラリをインストールできます
- `npm run dev` で開発サーバーを起動できます

`create-next-app` を実行すると、必要なライブラリが自動的にインストールされます。

### .tsx と .ts の違い

TypeScript のファイルには2種類の拡張子があります。

| 拡張子 | 用途 |
|--------|------|
| `.tsx` | JSX（HTML のような書き方）を含むファイル |
| `.ts` | JSX を含まない純粋な TypeScript ファイル |

React のコンポーネントは `.tsx` を使います。

## page.tsx を編集してみよう

`app/page.tsx` を開いて、中身をすべて削除し、以下に書き換えてください。

```tsx
export default function Home() {
  return (
    <div>
      <h1>はじめての Next.js</h1>
      <p>ページが表示されました！</p>
    </div>
  );
}
```

ファイルを保存すると、ブラウザが自動で更新されます（ホットリロード）。

「はじめての Next.js」と表示されれば成功です！

## layout.tsx について

`app/layout.tsx` は、すべてのページに共通するレイアウトを定義するファイルです。

```tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="ja">
      <body>{children}</body>
    </html>
  );
}
```

`{ children }: { children: React.ReactNode }` は TypeScript の書き方です。「children は React のノード（コンポーネントやテキスト）である」という意味です。

TypeScript の詳細はこの後の章で学びます。

## globals.css について

`app/globals.css` は、全体に適用されるスタイルです。

初期状態ではたくさんのスタイルが書かれていますが、一度中身をすべて削除して、シンプルにしましょう。

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  padding: 20px;
}
```

## Git でコミットしておこう

ここまでの変更を Git で保存しておきましょう。

```bash
git add .
git commit -m "プロジェクトの初期設定"
```

:::message
`git add .` は、すべての変更をステージングするコマンドです。
:::

## この章のまとめ

- Next.js は React を使ったフレームワーク
- `npx create-next-app` でプロジェクトを作成（TypeScript を有効に）
- `npm run dev` で開発サーバーを起動
- `.tsx` ファイルで React + TypeScript のコードを書く
- `app/page.tsx` がトップページ
- `app/layout.tsx` が全ページ共通のレイアウト

## トラブルシューティング

うまくいかないときは、以下を確認してみましょう。

### `npm run dev` が起動しない

**エラー: `command not found: npm`**
- Node.js がインストールされていない可能性があります
- `node -v` でバージョンを確認してください

**エラー: `Cannot find module`**
- プロジェクトフォルダで `npm install` を実行してください

```bash
cd my-first-app
npm install
npm run dev
```

### ポート3000が既に使われている

**エラー: `Port 3000 is already in use`**

別のポートを使うか、既に動いているプロセスを止めます。

```bash
# 別のポートを使う場合
npm run dev -- -p 3001

# または、既に動いているプロセスを止める
# Mac: アクティビティモニタで Node プロセスを終了
# Windows: タスクマネージャーで Node プロセスを終了
```

### ブラウザにエラーが表示される

1. **ブラウザの開発者ツールを開く**
   - `F12` キーを押す
   - または右クリック → 「検証」

2. **Console タブを確認**
   - エラーメッセージが表示されます
   - エラーの内容をコピーして検索すると、解決策が見つかることがあります

3. **VSCode の問題パネルを確認**
   - `Cmd + Shift + M`（Mac）/ `Ctrl + Shift + M`（Windows）
   - TypeScript のエラーが表示されます

### モジュールが見つからないエラー

**エラー: `Cannot find module 'next'`**

依存関係がインストールされていない可能性があります。

```bash
npm install
```

### ホットリロードが動かない

ファイルを保存してもブラウザが更新されない場合：

1. ブラウザをリロード（`Cmd + R` / `Ctrl + R`）
2. 開発サーバーを再起動（`Ctrl + C` で停止 → `npm run dev` で再起動）

## 確認してみよう

- [ ] Next.js + TypeScript プロジェクトを作成できた
- [ ] `npm run dev` で開発サーバーが起動する
- [ ] `http://localhost:3000` でページが表示される
- [ ] `page.tsx` を編集して、表示が変わることを確認した
