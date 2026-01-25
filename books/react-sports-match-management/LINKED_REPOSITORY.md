# 連携リポジトリについて

> **重要**: この教材は実装リポジトリ（match-flow）と連携しています。
> 教材を更新する際は、実装との整合性を確認してください。

---

## 連携先リポジトリ

| 項目 | 値 |
|------|-----|
| リポジトリ | `/Users/kunimitsu/workspace/サービス開発/match-flow/` |
| プロジェクト名 | Table Tennis Hub（卓球大会管理ツール） |
| 技術スタック | Next.js 16 + TypeScript + Tailwind CSS v4 + Prisma |

---

## 教材と実装の対応関係

### Part 1: 基礎編（localStorage版）

| 章 | ファイル | 対応する実装コード |
|----|----------|-------------------|
| 1 | introduction.md | - |
| 2 | project-setup.md | 初期設定全般 |
| 3 | layout-and-navigation.md | `app/layout.tsx`, `components/layout/` |
| 4 | ui-components.md | `components/ui/` |
| 5 | typescript-and-models.md | `types/` |
| 6 | competition-crud.md | `app/s/[sport]/events/`, `app/api/events/` |
| 7 | team-management.md | - |
| 8 | match-management.md | `app/api/matches/`, `components/events/MatchList.tsx` |
| 9 | standings-calculation.md | 順位計算ロジック |

### Part 2: 発展編（Prisma + 認証）

| 章 | ファイル | 対応する実装コード |
|----|----------|-------------------|
| 10 | database-setup.md | `prisma/schema.prisma`, `lib/db.ts` |
| 11 | auth-implementation.md | `lib/auth.ts`, `app/api/auth/` |
| 12 | data-migration.md | API Routes 全般 |
| 13 | advanced-features.md | `src/core/domain/` |
| 14 | deployment-and-next-steps.md | デプロイ設定 |

---

## 変更時の確認事項

### 教材更新前に確認すること

1. **実装コードを確認**
   ```bash
   # 実装リポジトリを開く
   code /Users/kunimitsu/workspace/サービス開発/match-flow/
   ```

2. **コードスニペットが実装と一致しているか確認**
   - 教材に書くコードは、実装で動作確認済みのものを使用
   - 命名規則やコードスタイルを統一

3. **実装を変更した場合は教材も更新**
   - match-flow 側で API やコンポーネントを変更した場合
   - 該当する章のコードスニペットを更新

---

## 実装リポジトリの構造

```
match-flow/
├── app/                    # Next.js App Router
│   ├── api/               # API Routes ← 教材で解説
│   ├── s/[sport]/events/  # 大会ページ ← 教材で解説
│   └── ...
├── components/             # コンポーネント
│   ├── ui/                # UIコンポーネント ← 教材で解説
│   ├── events/            # 大会関連 ← 教材で解説
│   └── ...
├── lib/                    # ユーティリティ
│   ├── auth.ts            # 認証 ← 教材で解説
│   ├── db.ts              # DB接続 ← 教材で解説
│   └── ...
├── prisma/                 # Prisma スキーマ ← 教材で解説
├── src/                    # ドメイン層（教材では発展課題として紹介）
└── ...
```

---

## 教材で扱う範囲

### 教材で詳しく解説するもの

- UIコンポーネント（Button, Input, Card, Select, Table）
- レイアウト構成（Navbar, layout.tsx）
- CRUD操作（大会、チーム、試合）
- 順位表計算ロジック
- Prisma スキーマ定義
- JWT認証（ログイン/サインアップ/ログアウト）
- API Routes（基本的なCRUD）

### 教材では「発展課題」として紹介のみ

以下は match-flow 独自の高度な実装。教材では紹介のみ：

- `src/core/domain/bracket/` - トーナメント自動生成
- `src/core/domain/roundrobin/` - 総当たり戦生成
- `src/sports/table-tennis/` - 卓球固有のスコア処理
- `src/lib/sports/registry.ts` - マルチスポーツ対応
- 通知機能
- 選手管理
- プロフィール管理

---

## コードスニペット作成時の注意

1. **実装から抜粋する場合**
   - 教材用に簡略化してOK（エラーハンドリングの省略など）
   - ただし、動作するコードであること

2. **教材独自のコードを書く場合**
   - 実装と矛盾しない範囲で
   - 将来的に実装へ取り込む可能性を考慮

3. **命名規則**
   - 教材: `Competition`（大会）
   - 実装: `Event`（大会）
   - 教材ではわかりやすさを優先

---

## 関連ファイル

実装リポジトリ側にも連携ドキュメントがあります：

- `/Users/kunimitsu/workspace/サービス開発/match-flow/LINKED_DOCUMENTATION.md`
- 各ディレクトリの `.BOOK_SYNC.md`
