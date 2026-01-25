---
title: "Part 2: データベースのセットアップ"
---

:::message
ここからは**発展編**です。前半（Part 1）で作成したlocalStorage版のアプリを、本格的なデータベースに対応させます。
:::

## この章でやること

- PostgreSQL のセットアップ
- Prisma ORM の導入
- データベーススキーマの定義

## なぜデータベースが必要？

localStorage の問題点：

| 問題 | 説明 |
|------|------|
| ブラウザ限定 | 他のデバイスからアクセスできない |
| 容量制限 | 約5MBまで |
| 複数ユーザー不可 | 自分のブラウザのデータしか見えない |
| バックアップ不可 | ブラウザのデータを消すと消える |

本番のアプリでは、**サーバー側のデータベース**にデータを保存します。

## PostgreSQL とは

PostgreSQL は、世界中で使われている**リレーショナルデータベース**です。

- 無料で使える
- 信頼性が高い
- 多くのホスティングサービスで利用可能

## Prisma とは

Prisma は、データベースを TypeScript から簡単に操作するための **ORM（Object-Relational Mapping）**です。

```typescript
// Prisma を使うと、SQLを書かずにデータベース操作ができる
const competitions = await prisma.competition.findMany();
const newTeam = await prisma.team.create({
  data: { name: "FC レッドスター" }
});
```

## ローカル環境のセットアップ

### 1. PostgreSQL のインストール

**Mac（Homebrew）**:
```bash
brew install postgresql@15
brew services start postgresql@15
```

**Windows**:
PostgreSQL公式サイトからインストーラーをダウンロードしてインストール。

**Docker を使う場合（推奨）**:
```bash
docker run --name match-manager-db -e POSTGRES_PASSWORD=password -p 5432:5432 -d postgres:15
```

### 2. データベースの作成

```bash
# PostgreSQL に接続
psql -U postgres

# データベース作成
CREATE DATABASE match_manager;
\q
```

## Prisma のインストール

```bash
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```

これで以下のファイルが生成されます：

```
prisma/
└── schema.prisma    # スキーマ定義ファイル

.env                 # 環境変数ファイル
```

## 環境変数の設定

`.env` ファイルを編集します：

```env
DATABASE_URL="postgresql://postgres:password@localhost:5432/match_manager"
```

:::message alert
`.env` ファイルには機密情報が含まれます。`.gitignore` に追加されていることを確認してください。
:::

## スキーマの定義

`prisma/schema.prisma` を編集します：

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// 大会
model Competition {
  id        String   @id @default(uuid())
  name      String
  sportType String   @map("sport_type")
  season    String?
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  matches Match[]

  @@map("competitions")
}

// チーム
model Team {
  id        String   @id @default(uuid())
  name      String
  shortName String?  @map("short_name")
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  homeMatches Match[] @relation("HomeTeam")
  awayMatches Match[] @relation("AwayTeam")

  @@map("teams")
}

// 試合
model Match {
  id            String      @id @default(uuid())
  competitionId String      @map("competition_id")
  homeTeamId    String      @map("home_team_id")
  awayTeamId    String      @map("away_team_id")
  homeScore     Int?        @map("home_score")
  awayScore     Int?        @map("away_score")
  status        String      @default("scheduled")
  startAt       DateTime    @map("start_at")
  venue         String?
  createdAt     DateTime    @default(now()) @map("created_at")
  updatedAt     DateTime    @updatedAt @map("updated_at")

  competition Competition @relation(fields: [competitionId], references: [id], onDelete: Cascade)
  homeTeam    Team        @relation("HomeTeam", fields: [homeTeamId], references: [id])
  awayTeam    Team        @relation("AwayTeam", fields: [awayTeamId], references: [id])

  @@map("matches")
}
```

### スキーマの解説

| 記法 | 意味 |
|------|------|
| `@id` | 主キー |
| `@default(uuid())` | UUIDを自動生成 |
| `@default(now())` | 現在日時を自動設定 |
| `@updatedAt` | 更新時に自動更新 |
| `@map("...")` | 実際のカラム名を指定 |
| `@@map("...")` | 実際のテーブル名を指定 |
| `@relation` | テーブル間のリレーション |

## マイグレーションの実行

スキーマをデータベースに反映します：

```bash
npx prisma migrate dev --name init
```

成功すると、以下が表示されます：

```
✔ Generated Prisma Client

Your database is now in sync with your schema.
```

## Prisma Client の使い方

`lib/prisma.ts` を作成：

```typescript
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: ["query"],
  });

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = prisma;
}
```

### 使用例

```typescript
import { prisma } from "@/lib/prisma";

// 全件取得
const competitions = await prisma.competition.findMany();

// 条件付き取得
const team = await prisma.team.findUnique({
  where: { id: "xxx" },
});

// 作成
const newCompetition = await prisma.competition.create({
  data: {
    name: "春季フットサルリーグ",
    sportType: "football",
  },
});

// 更新
await prisma.competition.update({
  where: { id: "xxx" },
  data: { name: "新しい名前" },
});

// 削除
await prisma.competition.delete({
  where: { id: "xxx" },
});
```

## Prisma Studio

データベースの中身をGUIで確認できます：

```bash
npx prisma studio
```

ブラウザで `http://localhost:5555` が開きます。

## Git でコミット

```bash
git add .
git commit -m "Prisma + PostgreSQL のセットアップ"
```

## この章のまとめ

- PostgreSQL をセットアップした
- Prisma ORM を導入した
- データベーススキーマを定義した
- マイグレーションを実行した

## 確認してみよう

- [ ] PostgreSQL が起動している
- [ ] `npx prisma migrate dev` が成功した
- [ ] `npx prisma studio` でテーブルが見える
