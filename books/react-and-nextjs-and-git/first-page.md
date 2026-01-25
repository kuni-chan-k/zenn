---
title: "最初のページを作ろう"
---

## この章でやること

- 新しいページを追加する
- ページ間のリンクを作る
- コンポーネントを分けて整理する

## ページの追加方法

Next.js（App Router）では、**フォルダを作ってその中に `page.tsx` を置く**とページになります。

```
app/
├── page.tsx          → http://localhost:3000/
├── about/
│   └── page.tsx      → http://localhost:3000/about
└── contact/
    └── page.tsx      → http://localhost:3000/contact
```

## About ページを作ろう

`app` フォルダ内に `about` フォルダを作り、その中に `page.tsx` を作成します。

```
app/
├── page.tsx
└── about/
    └── page.tsx  ← これを作る
```

`app/about/page.tsx`:

```tsx
export default function AboutPage() {
  return (
    <div>
      <h1>About</h1>
      <p>このサイトについてのページです。</p>
    </div>
  );
}
```

ファイルを保存したら、`http://localhost:3000/about` にアクセスしてみてください。

## ページ間のリンク

Next.js では、`Link` コンポーネントを使ってページ間を移動します。

`app/page.tsx` を編集して、About ページへのリンクを追加しましょう。

```tsx
import Link from 'next/link';

export default function Home() {
  return (
    <div>
      <h1>ホーム</h1>
      <p>ようこそ！</p>

      <nav>
        <Link href="/about">Aboutページへ</Link>
      </nav>
    </div>
  );
}
```

`app/about/page.tsx` にもホームへ戻るリンクを追加します。

```tsx
import Link from 'next/link';

export default function AboutPage() {
  return (
    <div>
      <h1>About</h1>
      <p>このサイトについてのページです。</p>

      <Link href="/">ホームに戻る</Link>
    </div>
  );
}
```

リンクをクリックして、ページ間を移動できることを確認してください。

:::message
`<a>` タグではなく `<Link>` を使うと、ページ全体を読み込まずに高速に遷移できます。
:::

## コンポーネントを分ける

ページが増えてくると、共通の部品を別ファイルに分けると便利です。

### components フォルダを作る

プロジェクトのルートに `components` フォルダを作り、共通コンポーネントを置きます。

```
my-first-app/
├── app/
├── components/       ← 新しく作る
│   └── Header.tsx
└── ...
```

### ヘッダーコンポーネントを作る

`components/Header.tsx`:

```tsx
import Link from 'next/link';

export default function Header() {
  return (
    <header style={{
      borderBottom: '1px solid #ccc',
      padding: '10px 0',
      marginBottom: '20px'
    }}>
      <nav style={{ display: 'flex', gap: '20px' }}>
        <Link href="/">ホーム</Link>
        <Link href="/about">About</Link>
      </nav>
    </header>
  );
}
```

### layout.tsx でヘッダーを読み込む

`app/layout.tsx` を編集して、全ページにヘッダーを表示します。

```tsx
import Header from '../components/Header';
import './globals.css';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="ja">
      <body>
        <Header />
        <main>{children}</main>
      </body>
    </html>
  );
}
```

これで、すべてのページにヘッダーが表示されるようになりました。

:::message
`{ children }: { children: React.ReactNode }` は TypeScript の型定義です。
「children は React のノードである」という意味で、Next.js のレイアウトでは必須の型です。
:::

### 各ページからナビゲーションを削除

ヘッダーにナビゲーションを移動したので、各ページのリンクは削除します。

`app/page.tsx`:

```tsx
export default function Home() {
  return (
    <div>
      <h1>ホーム</h1>
      <p>ようこそ！</p>
    </div>
  );
}
```

`app/about/page.tsx`:

```tsx
export default function AboutPage() {
  return (
    <div>
      <h1>About</h1>
      <p>このサイトについてのページです。</p>
    </div>
  );
}
```

## useState を使ったページを作る

前の章で学んだ `useState` を使ったページも作ってみましょう。

`app/counter/page.tsx`:

```tsx
'use client';  // ← これが必要！

import { useState } from 'react';

export default function CounterPage() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h1>カウンター</h1>
      <p style={{ fontSize: '48px' }}>{count}</p>
      <button onClick={() => setCount(count - 1)}>-1</button>
      <button onClick={() => setCount(0)}>リセット</button>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

:::message alert
`useState` などのフックを使うコンポーネントでは、ファイルの先頭に `'use client';` が必要です。

Next.js（App Router）では、デフォルトでサーバーで実行されるため、ブラウザで動く機能を使うときはこの宣言が必要になります。
:::

ヘッダーにもリンクを追加しましょう。

`components/Header.tsx`:

```tsx
import Link from 'next/link';

export default function Header() {
  return (
    <header style={{
      borderBottom: '1px solid #ccc',
      padding: '10px 0',
      marginBottom: '20px'
    }}>
      <nav style={{ display: 'flex', gap: '20px' }}>
        <Link href="/">ホーム</Link>
        <Link href="/about">About</Link>
        <Link href="/counter">カウンター</Link>
      </nav>
    </header>
  );
}
```

`http://localhost:3000/counter` にアクセスして、カウンターが動くことを確認してください。

## 最終的なファイル構成

```
my-first-app/
├── app/
│   ├── layout.tsx        # 全ページ共通レイアウト
│   ├── page.tsx          # ホームページ
│   ├── globals.css       # 全体スタイル
│   ├── about/
│   │   └── page.tsx      # Aboutページ
│   └── counter/
│       └── page.tsx      # カウンターページ
├── components/
│   └── Header.tsx        # ヘッダーコンポーネント
└── ...
```

## Git でコミット

最後に、今日の作業をコミットしておきましょう。

```bash
git add .
git commit -m "ページとコンポーネントを追加"
```

## この章のまとめ

- フォルダに `page.tsx` を置くとページになる
- `Link` コンポーネントでページ間を移動する
- 共通コンポーネントは別ファイルに分ける
- `useState` を使うときは `'use client';` が必要

## 確認してみよう

- [ ] About ページを作成できた
- [ ] Link でページ間を移動できる
- [ ] Header コンポーネントを作って全ページに表示できた
- [ ] カウンターページが動作する

---

## おめでとうございます！

これで「React & Next.js 超入門」は完了です！

あなたは以下のことができるようになりました。

- ✅ 開発環境を整える（Node.js, VSCode）
- ✅ Git でコードを管理する
- ✅ React / TypeScript の基本概念を理解する（コンポーネント、props、state、型）
- ✅ Next.js でアプリを作成・起動する
- ✅ ページとコンポーネントを作る

次のステップとして、「**Reactで作る！スポーツ大会管理サービス**」に進んで、実践的なアプリ開発に挑戦しましょう！
