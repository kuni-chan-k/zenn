---
title: "プロジェクトのセットアップ"
---

## この章でやること

- Next.js + TypeScript プロジェクトの作成
- Tailwind CSS の導入
- ディレクトリ構成の設計

## プロジェクトの作成

ターミナルで以下のコマンドを実行します。

```bash
npx create-next-app@latest match-manager
```

今回は以下のように答えてください（前の本とは違う設定があります）。

```
✔ Would you like to use TypeScript? … Yes        ← 今回はYes！
✔ Would you like to use ESLint? … Yes
✔ Would you like to use Tailwind CSS? … Yes     ← 今回はYes！
✔ Would you like your code inside a `src/` directory? … No
✔ Would you like to use App Router? (recommended) … Yes
✔ Would you like to use Turbopack for `next dev`? … No
✔ Would you like to customize the import alias (@/* by default)? … No
```

プロジェクトに移動して、開発サーバーを起動します。

```bash
cd match-manager
npm run dev
```

`http://localhost:3000` でページが表示されることを確認してください。

## ディレクトリ構成

このプロジェクトでは、以下のような構成にします。

```
match-manager/
├── app/                    # ページ
│   ├── layout.tsx
│   ├── page.tsx
│   ├── competitions/       # 大会関連ページ
│   ├── teams/              # チーム関連ページ
│   └── matches/            # 試合関連ページ
├── components/             # 共通コンポーネント
│   ├── ui/                 # 汎用UIコンポーネント
│   └── layout/             # レイアウト関連
├── lib/                    # ユーティリティ・ロジック
│   ├── types.ts            # 型定義
│   └── data.ts             # データ管理
└── ...
```

## 不要なファイルの整理

初期状態のファイルを整理しましょう。

### app/page.tsx

中身を以下に書き換えます。

```tsx
export default function Home() {
  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold">大会管理サービス</h1>
      <p className="mt-4 text-gray-600">フットサル大会の管理ができます。</p>
    </div>
  );
}
```

### app/globals.css

以下だけ残して、他は削除します。

```css
@import "tailwindcss";
```

### app/layout.tsx

以下のように修正します。

```tsx
import type { Metadata } from "next";
import "./globals.css";

export const metadata: Metadata = {
  title: "大会管理サービス",
  description: "スポーツ大会の管理ができるアプリです",
};

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

## Tailwind CSS の確認

Tailwind CSS が動いているか確認しましょう。

`app/page.tsx` のスタイルを変えてみます。

```tsx
export default function Home() {
  return (
    <div className="min-h-screen bg-gray-100 p-8">
      <h1 className="text-3xl font-bold text-blue-600">大会管理サービス</h1>
      <p className="mt-4 text-gray-600">フットサル大会の管理ができます。</p>

      <button className="mt-6 bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600">
        ボタンのテスト
      </button>
    </div>
  );
}
```

- 背景がグレーになる
- 見出しが青くなる
- ボタンにスタイルが適用される

これらが確認できれば、Tailwind CSS は正しく動いています。

## フォルダの作成

必要なフォルダを先に作っておきましょう。

```bash
mkdir -p components/ui
mkdir -p components/layout
mkdir -p lib
```

## Git でコミット

```bash
git add .
git commit -m "プロジェクトの初期セットアップ"
```

## この章のまとめ

- Next.js + TypeScript + Tailwind CSS でプロジェクトを作成
- 不要なファイルを整理
- ディレクトリ構成を設計

## 確認してみよう

- [ ] プロジェクトが作成できた
- [ ] `npm run dev` でエラーなく起動する
- [ ] Tailwind CSS のスタイルが適用されている
- [ ] Git でコミットした
