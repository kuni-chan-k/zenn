---
title: "高度な機能"
---

## この章でやること

- トーナメント戦の自動進出機能
- 総当り戦のリーグ形式
- ユーザーロールによる権限管理

## トーナメント戦の自動進出

トーナメント戦では、第1回戦の勝者が自動的に第2回戦に進出します。

### スキーマの拡張

`prisma/schema.prisma` に追加：

```prisma
model Match {
  // 既存のフィールド...

  round     Int      @default(1)  // ラウンド番号（1回戦、2回戦...）
  bracket   String?               // トーナメント位置（"A1", "A2"など）
  winnerId  String?  @map("winner_id")  // 勝者のチームID

  // リレーション
  winner    Team?    @relation("Winner", fields: [winnerId], references: [id])
}
```

### 自動進出ロジック

`lib/tournament.ts`:

```typescript
import { prisma } from "./prisma";
import { Match } from "@/lib/types";

// トーナメントの次のラウンドを生成
export async function generateNextRound(competitionId: string): Promise<void> {
  // 現在のラウンドの試合を取得
  const currentMatches = await prisma.match.findMany({
    where: {
      competitionId,
      status: "finished",
      winnerId: { not: null },
    },
    orderBy: { bracket: "asc" },
  });

  // 現在のラウンド番号を取得
  const currentRound = Math.max(...currentMatches.map(m => m.round));

  // 同じラウンドの完了した試合を取得
  const roundMatches = currentMatches.filter(m => m.round === currentRound);

  // ペアにして次のラウンドの試合を作成
  const pairs: Match[][] = [];
  for (let i = 0; i < roundMatches.length; i += 2) {
    if (roundMatches[i + 1]) {
      pairs.push([roundMatches[i], roundMatches[i + 1]]);
    }
  }

  // 次のラウンドの試合を作成
  for (let i = 0; i < pairs.length; i++) {
    const [match1, match2] = pairs[i];

    if (!match1.winnerId || !match2.winnerId) continue;

    await prisma.match.create({
      data: {
        competitionId,
        homeTeamId: match1.winnerId,
        awayTeamId: match2.winnerId,
        round: currentRound + 1,
        bracket: `R${currentRound + 1}-${i + 1}`,
        status: "scheduled",
        startAt: new Date(), // 実際には適切な日時を設定
      },
    });
  }
}

// 試合結果を登録して勝者を設定
export async function recordMatchResult(
  matchId: string,
  homeScore: number,
  awayScore: number
): Promise<void> {
  const match = await prisma.match.findUnique({
    where: { id: matchId },
  });

  if (!match) throw new Error("Match not found");

  // 勝者を判定
  let winnerId: string | null = null;
  if (homeScore > awayScore) {
    winnerId = match.homeTeamId;
  } else if (awayScore > homeScore) {
    winnerId = match.awayTeamId;
  }
  // 引き分けの場合は winnerId = null

  // 結果を更新
  await prisma.match.update({
    where: { id: matchId },
    data: {
      homeScore,
      awayScore,
      status: "finished",
      winnerId,
    },
  });
}
```

## 総当り戦の生成

総当り戦（リーグ戦）では、すべてのチームが互いに対戦します。

`lib/round-robin.ts`:

```typescript
import { prisma } from "./prisma";

// 総当り戦の試合を自動生成
export async function generateRoundRobinMatches(
  competitionId: string,
  teamIds: string[]
): Promise<void> {
  const matches: { homeTeamId: string; awayTeamId: string }[] = [];

  // すべての組み合わせを生成
  for (let i = 0; i < teamIds.length; i++) {
    for (let j = i + 1; j < teamIds.length; j++) {
      matches.push({
        homeTeamId: teamIds[i],
        awayTeamId: teamIds[j],
      });
    }
  }

  // データベースに保存
  await prisma.match.createMany({
    data: matches.map((match, index) => ({
      competitionId,
      homeTeamId: match.homeTeamId,
      awayTeamId: match.awayTeamId,
      round: Math.floor(index / (teamIds.length / 2)) + 1,
      status: "scheduled",
      startAt: new Date(), // 実際には適切な日時を設定
    })),
  });
}
```

## ユーザーロールによる権限管理

### ロールの定義

```typescript
// lib/types.ts
export type UserRole = "user" | "organizer" | "admin";

export const PERMISSIONS = {
  // 大会関連
  "competition:create": ["organizer", "admin"],
  "competition:edit": ["organizer", "admin"],
  "competition:delete": ["admin"],

  // 試合関連
  "match:create": ["organizer", "admin"],
  "match:edit": ["organizer", "admin"],
  "match:delete": ["admin"],
} as const;
```

### 権限チェックユーティリティ

`lib/permissions.ts`:

```typescript
import { UserRole, PERMISSIONS } from "./types";

type Permission = keyof typeof PERMISSIONS;

export function hasPermission(
  userRole: UserRole,
  permission: Permission
): boolean {
  const allowedRoles = PERMISSIONS[permission];
  return allowedRoles.includes(userRole);
}

export function requirePermission(
  userRole: UserRole,
  permission: Permission
): void {
  if (!hasPermission(userRole, permission)) {
    throw new Error(`Permission denied: ${permission}`);
  }
}
```

### API での権限チェック

```typescript
import { getCurrentUser } from "@/lib/auth";
import { hasPermission } from "@/lib/permissions";

export async function DELETE(request: NextRequest, { params }: Params) {
  const user = await getCurrentUser();

  if (!user) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  if (!hasPermission(user.role as UserRole, "competition:delete")) {
    return NextResponse.json({ error: "Forbidden" }, { status: 403 });
  }

  // 削除処理...
}
```

### フロントエンドでの権限チェック

```tsx
"use client";

import { useAuth } from "@/lib/hooks/use-auth";
import { hasPermission } from "@/lib/permissions";

export function CompetitionActions({ competitionId }: { competitionId: string }) {
  const { user } = useAuth();

  const canEdit = user && hasPermission(user.role, "competition:edit");
  const canDelete = user && hasPermission(user.role, "competition:delete");

  return (
    <div className="flex gap-2">
      {canEdit && (
        <Link href={`/competitions/${competitionId}/edit`}>
          <Button variant="secondary">編集</Button>
        </Link>
      )}
      {canDelete && (
        <Button variant="danger" onClick={handleDelete}>
          削除
        </Button>
      )}
    </div>
  );
}
```

## match-flow の参考コード

実際の実装は match-flow リポジトリを参考にしてください：

```
/Users/kunimitsu/workspace/サービス開発/match-flow/
├── app/api/           # API Routes
├── components/        # コンポーネント
├── lib/              # ユーティリティ
└── types/            # 型定義
```

特に以下のファイルが参考になります：

- `app/api/matches/route.ts` - 試合管理 API
- `components/events/MatchList.tsx` - 試合一覧コンポーネント
- `lib/auth.ts` - 認証ユーティリティ

## Git でコミット

```bash
git add .
git commit -m "高度な機能を実装"
```

## この章のまとめ

- トーナメント戦の自動進出機能を実装した
- 総当り戦の試合自動生成を実装した
- ユーザーロールによる権限管理を実装した

## 確認してみよう

- [ ] トーナメント戦で試合結果を入力すると、次のラウンドが生成される
- [ ] 総当り戦で全チームの対戦が生成される
- [ ] ユーザーロールによってボタンの表示が変わる
- [ ] 権限のない操作がエラーになる
