---
title: "レイアウトとナビゲーション"
---

## この章でやること

- ヘッダーコンポーネントの作成
- サイドバーの作成
- 全体レイアウトの構築

## レイアウトの設計

このアプリは以下のようなレイアウトにします。

```
┌─────────────────────────────────────┐
│           ヘッダー                    │
├────────────┬────────────────────────┤
│            │                        │
│ サイドバー   │      メインコンテンツ    │
│            │                        │
│ ・ダッシュボード│                     │
│ ・大会        │                     │
│ ・チーム      │                     │
│            │                        │
└────────────┴────────────────────────┘
```

## ヘッダーの作成

`components/layout/Header.tsx`:

```tsx
import Link from "next/link";

export default function Header() {
  return (
    <header className="bg-blue-600 text-white">
      <div className="px-4 py-3">
        <Link href="/" className="text-xl font-bold">
          大会管理サービス
        </Link>
      </div>
    </header>
  );
}
```

## サイドバーの作成

`components/layout/Sidebar.tsx`:

```tsx
"use client";

import Link from "next/link";
import { usePathname } from "next/navigation";

const navItems = [
  { href: "/", label: "ダッシュボード" },
  { href: "/competitions", label: "大会" },
  { href: "/teams", label: "チーム" },
];

export default function Sidebar() {
  const pathname = usePathname();

  return (
    <aside className="w-64 bg-gray-800 text-white min-h-screen">
      <nav className="p-4">
        <ul className="space-y-2">
          {navItems.map((item) => {
            const isActive = pathname === item.href;
            return (
              <li key={item.href}>
                <Link
                  href={item.href}
                  className={`block px-4 py-2 rounded ${
                    isActive
                      ? "bg-blue-600"
                      : "hover:bg-gray-700"
                  }`}
                >
                  {item.label}
                </Link>
              </li>
            );
          })}
        </ul>
      </nav>
    </aside>
  );
}
```

### ポイント解説

- `"use client"`: ブラウザで動く機能（`usePathname`）を使うため必要
- `usePathname()`: 現在のURL パスを取得する Next.js のフック
- `isActive`: 現在のページならスタイルを変える

## レイアウトの組み立て

`app/layout.tsx` を更新します。

```tsx
import type { Metadata } from "next";
import "./globals.css";
import Header from "@/components/layout/Header";
import Sidebar from "@/components/layout/Sidebar";

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
      <body>
        <div className="min-h-screen flex flex-col">
          <Header />
          <div className="flex flex-1">
            <Sidebar />
            <main className="flex-1 bg-gray-100 p-6">
              {children}
            </main>
          </div>
        </div>
      </body>
    </html>
  );
}
```

### Tailwind クラスの解説

| クラス | 意味 |
|--------|------|
| `min-h-screen` | 最小高さを画面いっぱいに |
| `flex` | フレックスボックスを有効化 |
| `flex-col` | 縦方向に並べる |
| `flex-1` | 残りのスペースを埋める |

## トップページの更新

`app/page.tsx`:

```tsx
export default function Home() {
  return (
    <div>
      <h1 className="text-2xl font-bold mb-6">ダッシュボード</h1>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        <div className="bg-white p-6 rounded-lg shadow">
          <h2 className="text-lg font-semibold text-gray-700">大会数</h2>
          <p className="text-3xl font-bold text-blue-600 mt-2">0</p>
        </div>

        <div className="bg-white p-6 rounded-lg shadow">
          <h2 className="text-lg font-semibold text-gray-700">チーム数</h2>
          <p className="text-3xl font-bold text-green-600 mt-2">0</p>
        </div>

        <div className="bg-white p-6 rounded-lg shadow">
          <h2 className="text-lg font-semibold text-gray-700">試合数</h2>
          <p className="text-3xl font-bold text-orange-600 mt-2">0</p>
        </div>
      </div>
    </div>
  );
}
```

## 大会ページ（仮）の作成

サイドバーのリンク先を作っておきましょう。

`app/competitions/page.tsx`:

```tsx
export default function CompetitionsPage() {
  return (
    <div>
      <h1 className="text-2xl font-bold mb-6">大会一覧</h1>
      <p className="text-gray-600">大会の管理ページです（実装中）</p>
    </div>
  );
}
```

`app/teams/page.tsx`:

```tsx
export default function TeamsPage() {
  return (
    <div>
      <h1 className="text-2xl font-bold mb-6">チーム一覧</h1>
      <p className="text-gray-600">チームの管理ページです（実装中）</p>
    </div>
  );
}
```

## 動作確認

ブラウザで以下を確認してください。

1. ヘッダーが上部に表示される
2. サイドバーが左側に表示される
3. サイドバーのリンクをクリックするとページが切り替わる
4. 現在のページのリンクがハイライトされる

## Git でコミット

```bash
git add .
git commit -m "レイアウトとナビゲーションを追加"
```

## この章のまとめ

- ヘッダーとサイドバーを作成した
- `usePathname` で現在のページを取得し、ナビゲーションをハイライト
- Flexbox でレイアウトを構築

## 確認してみよう

- [ ] ヘッダーが表示される
- [ ] サイドバーが表示される
- [ ] ナビゲーションリンクが動作する
- [ ] 現在のページがハイライトされる
