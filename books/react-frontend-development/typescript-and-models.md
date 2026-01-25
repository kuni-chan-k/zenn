---
title: "TypeScriptとデータモデル"
---

## この章でやること

- TypeScript の基礎を学ぶ
- アプリで使うデータの型を定義する
- 型を使ってコードの安全性を高める

## TypeScript とは

TypeScript は、**JavaScript に「型」を追加した言語**です。

「型」とは、データの種類を明示することです。

```typescript
// JavaScript - 何でも入る
let name = "田中";
name = 123;  // エラーにならない（バグの原因に）

// TypeScript - 型を指定
let name: string = "田中";
name = 123;  // エラー！ string に number は入れられない
```

型があると、**間違いを事前に防げる**のがメリットです。

## 基本的な型

### プリミティブ型

```typescript
// 文字列
const name: string = "田中";

// 数値
const age: number = 25;

// 真偽値
const isActive: boolean = true;

// null と undefined
const empty: null = null;
const notSet: undefined = undefined;
```

### 配列

```typescript
// 文字列の配列
const names: string[] = ["田中", "鈴木", "佐藤"];

// 数値の配列
const scores: number[] = [80, 90, 75];
```

### オブジェクト

```typescript
// オブジェクトの型を定義
type User = {
  name: string;
  age: number;
  email: string;
};

// 型に従ったオブジェクトを作成
const user: User = {
  name: "田中",
  age: 25,
  email: "tanaka@example.com",
};
```

## type と interface

オブジェクトの型を定義するには `type` または `interface` を使います。

```typescript
// type を使う方法
type Team = {
  id: string;
  name: string;
};

// interface を使う方法
interface Team {
  id: string;
  name: string;
}
```

どちらもほぼ同じことができます。この本では `type` を使います。

## オプショナルプロパティ

`?` をつけると、あってもなくてもよいプロパティになります。

```typescript
type Team = {
  id: string;
  name: string;
  shortName?: string;  // あってもなくてもOK
};

// OK
const team1: Team = { id: "1", name: "FC レッドスター" };
const team2: Team = { id: "2", name: "ブルーウィングス", shortName: "BW" };
```

## ユニオン型

複数の型のいずれかを表します。

```typescript
// 文字列または数値
type ID = string | number;

// 特定の文字列のいずれか
type MatchStatus = "scheduled" | "finished" | "cancelled";
```

## このアプリのデータモデル

それでは、大会管理アプリで使うデータの型を定義しましょう。

`lib/types.ts`:

```typescript
// 大会
export type Competition = {
  id: string;
  name: string;
  sportType: "football" | "tabletennis" | "other";
  season?: string;
  createdAt: string;
  updatedAt: string;
};

// チーム
export type Team = {
  id: string;
  name: string;
  shortName?: string;
  createdAt: string;
  updatedAt: string;
};

// 試合
export type Match = {
  id: string;
  competitionId: string;
  homeTeamId: string;
  awayTeamId: string;
  homeScore: number | null;
  awayScore: number | null;
  status: "scheduled" | "finished";
  startAt: string;
  venue?: string;
  createdAt: string;
  updatedAt: string;
};

// 順位表の1行
export type Standing = {
  teamId: string;
  teamName: string;
  played: number;       // 試合数
  won: number;          // 勝利
  drawn: number;        // 引分
  lost: number;         // 敗北
  goalsFor: number;     // 得点
  goalsAgainst: number; // 失点
  goalDiff: number;     // 得失点差
  points: number;       // 勝点
  rank: number;         // 順位
};
```

### モデルの解説

#### Competition（大会）

| プロパティ | 型 | 説明 |
|-----------|------|------|
| id | string | 一意の識別子 |
| name | string | 大会名 |
| sportType | "football" など | 競技種類 |
| season | string（省略可） | シーズン（例: "2024春"） |
| createdAt | string | 作成日時 |
| updatedAt | string | 更新日時 |

#### Team（チーム）

| プロパティ | 型 | 説明 |
|-----------|------|------|
| id | string | 一意の識別子 |
| name | string | チーム名 |
| shortName | string（省略可） | 略称 |

#### Match（試合）

| プロパティ | 型 | 説明 |
|-----------|------|------|
| id | string | 一意の識別子 |
| competitionId | string | 所属する大会の ID |
| homeTeamId | string | ホームチームの ID |
| awayTeamId | string | アウェイチームの ID |
| homeScore | number または null | ホームの得点 |
| awayScore | number または null | アウェイの得点 |
| status | "scheduled" など | 試合状態 |
| startAt | string | 開始日時 |
| venue | string（省略可） | 会場 |

#### Standing（順位）

試合結果から計算される、順位表の1行分のデータです。

## ID の生成

データを作成するときに、一意の ID が必要です。簡単な方法として、`crypto.randomUUID()` を使います。

```typescript
const newId = crypto.randomUUID();
// 例: "550e8400-e29b-41d4-a716-446655440000"
```

## 日時の扱い

日時は ISO 8601 形式の文字列で扱います。

```typescript
const now = new Date().toISOString();
// 例: "2024-01-15T10:30:00.000Z"
```

## ユーティリティ関数

`lib/utils.ts`:

```typescript
// 新しいIDを生成
export function generateId(): string {
  return crypto.randomUUID();
}

// 現在の日時をISO文字列で取得
export function getCurrentTimestamp(): string {
  return new Date().toISOString();
}

// 日時を表示用にフォーマット
export function formatDate(isoString: string): string {
  const date = new Date(isoString);
  return date.toLocaleDateString("ja-JP", {
    year: "numeric",
    month: "long",
    day: "numeric",
  });
}

// 日時を入力フォーム用にフォーマット
export function formatDateTimeForInput(isoString: string): string {
  const date = new Date(isoString);
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, "0");
  const day = String(date.getDate()).padStart(2, "0");
  const hours = String(date.getHours()).padStart(2, "0");
  const minutes = String(date.getMinutes()).padStart(2, "0");
  return `${year}-${month}-${day}T${hours}:${minutes}`;
}
```

## 型を使う練習

型を使って、大会データを作成してみましょう。

```typescript
import { Competition } from "@/lib/types";
import { generateId, getCurrentTimestamp } from "@/lib/utils";

// 新しい大会を作成
const newCompetition: Competition = {
  id: generateId(),
  name: "春季フットサルリーグ 2024",
  sportType: "football",
  season: "2024春",
  createdAt: getCurrentTimestamp(),
  updatedAt: getCurrentTimestamp(),
};

console.log(newCompetition);
```

型を指定しているので、以下のような間違いは事前にエラーになります。

```typescript
// ❌ エラー: name は string なのに number を渡している
const wrong: Competition = {
  id: generateId(),
  name: 123,  // Type 'number' is not assignable to type 'string'
  ...
};

// ❌ エラー: sportType は指定された値のみ
const wrong2: Competition = {
  id: generateId(),
  name: "テスト",
  sportType: "soccer",  // "football" | "tabletennis" | "other" じゃない
  ...
};
```

## Git でコミット

```bash
git add .
git commit -m "型定義とユーティリティ関数を追加"
```

## この章のまとめ

- TypeScript は JavaScript に型を追加した言語
- 型があると間違いを事前に防げる
- `type` でオブジェクトの型を定義する
- 大会、チーム、試合、順位の型を定義した

## 確認してみよう

- [ ] 基本的な型（string, number, boolean）を理解した
- [ ] オブジェクトの型を `type` で定義できる
- [ ] オプショナル（`?`）とユニオン（`|`）を理解した
- [ ] `lib/types.ts` にデータモデルを作成した
