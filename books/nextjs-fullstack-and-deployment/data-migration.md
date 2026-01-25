---
title: "データ層の移行"
---

## この章でやること

- localStorage から Prisma への移行
- API Routes の実装
- フロントエンドの修正

## 移行の方針

「Reactで作る！フロントエンド完結アプリ開発」のPart 3で作成したlocalStorage版を、Prisma を使ったサーバーサイドに移行します。

**変更点:**

| 項目 | Before (localStorage) | After (Prisma) |
|------|----------------------|----------------|
| データ保存 | ブラウザ | PostgreSQL |
| データ取得 | 直接読み込み | API 経由 |
| データ更新 | 直接書き込み | API 経由 |

## API Routes の作成

### 大会一覧 API

`app/api/competitions/route.ts`:

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { getCurrentUser } from "@/lib/auth";

// 大会一覧取得
export async function GET() {
  try {
    const competitions = await prisma.competition.findMany({
      orderBy: { createdAt: "desc" },
    });

    return NextResponse.json(competitions);
  } catch (error) {
    console.error("Error fetching competitions:", error);
    return NextResponse.json(
      { error: "Failed to fetch competitions" },
      { status: 500 }
    );
  }
}

// 大会作成
export async function POST(request: NextRequest) {
  try {
    const user = await getCurrentUser();
    if (!user) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const { name, sportType, season } = await request.json();

    if (!name || !sportType) {
      return NextResponse.json(
        { error: "Name and sportType are required" },
        { status: 400 }
      );
    }

    const competition = await prisma.competition.create({
      data: {
        name,
        sportType,
        season,
      },
    });

    return NextResponse.json(competition, { status: 201 });
  } catch (error) {
    console.error("Error creating competition:", error);
    return NextResponse.json(
      { error: "Failed to create competition" },
      { status: 500 }
    );
  }
}
```

### 大会詳細・更新・削除 API

`app/api/competitions/[id]/route.ts`:

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { getCurrentUser } from "@/lib/auth";

type Params = { params: Promise<{ id: string }> };

// 大会詳細取得
export async function GET(request: NextRequest, { params }: Params) {
  try {
    const { id } = await params;

    const competition = await prisma.competition.findUnique({
      where: { id },
      include: {
        matches: {
          orderBy: { startAt: "asc" },
        },
      },
    });

    if (!competition) {
      return NextResponse.json(
        { error: "Competition not found" },
        { status: 404 }
      );
    }

    return NextResponse.json(competition);
  } catch (error) {
    console.error("Error fetching competition:", error);
    return NextResponse.json(
      { error: "Failed to fetch competition" },
      { status: 500 }
    );
  }
}

// 大会更新
export async function PUT(request: NextRequest, { params }: Params) {
  try {
    const user = await getCurrentUser();
    if (!user) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const { id } = await params;
    const { name, sportType, season } = await request.json();

    const competition = await prisma.competition.update({
      where: { id },
      data: {
        name,
        sportType,
        season,
      },
    });

    return NextResponse.json(competition);
  } catch (error) {
    console.error("Error updating competition:", error);
    return NextResponse.json(
      { error: "Failed to update competition" },
      { status: 500 }
    );
  }
}

// 大会削除
export async function DELETE(request: NextRequest, { params }: Params) {
  try {
    const user = await getCurrentUser();
    if (!user) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const { id } = await params;

    await prisma.competition.delete({
      where: { id },
    });

    return NextResponse.json({ success: true });
  } catch (error) {
    console.error("Error deleting competition:", error);
    return NextResponse.json(
      { error: "Failed to delete competition" },
      { status: 500 }
    );
  }
}
```

### チーム API

`app/api/teams/route.ts`:

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { getCurrentUser } from "@/lib/auth";

export async function GET() {
  try {
    const teams = await prisma.team.findMany({
      orderBy: { name: "asc" },
    });

    return NextResponse.json(teams);
  } catch (error) {
    console.error("Error fetching teams:", error);
    return NextResponse.json(
      { error: "Failed to fetch teams" },
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  try {
    const user = await getCurrentUser();
    if (!user) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const { name, shortName } = await request.json();

    if (!name) {
      return NextResponse.json(
        { error: "Name is required" },
        { status: 400 }
      );
    }

    const team = await prisma.team.create({
      data: {
        name,
        shortName,
      },
    });

    return NextResponse.json(team, { status: 201 });
  } catch (error) {
    console.error("Error creating team:", error);
    return NextResponse.json(
      { error: "Failed to create team" },
      { status: 500 }
    );
  }
}
```

## フロントエンドの修正

### データ取得フック

`lib/hooks/use-competitions.ts`:

```typescript
"use client";

import { useState, useEffect, useCallback } from "react";
import { Competition } from "@/lib/types";

export function useCompetitions() {
  const [competitions, setCompetitions] = useState<Competition[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchCompetitions = useCallback(async () => {
    try {
      setLoading(true);
      const res = await fetch("/api/competitions");
      if (!res.ok) throw new Error("Failed to fetch");
      const data = await res.json();
      setCompetitions(data);
      setError(null);
    } catch (err) {
      setError("データの取得に失敗しました");
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchCompetitions();
  }, [fetchCompetitions]);

  return { competitions, loading, error, refetch: fetchCompetitions };
}
```

### ページコンポーネントの修正

`app/competitions/page.tsx`:

```tsx
"use client";

import { useState } from "react";
import Link from "next/link";
import { useCompetitions } from "@/lib/hooks/use-competitions";
import { Button, Card, Table } from "@/components/ui";
import { formatDate } from "@/lib/utils";
import { Competition } from "@/lib/types";

export default function CompetitionsPage() {
  const { competitions, loading, error, refetch } = useCompetitions();

  const handleDelete = async (id: string) => {
    if (!confirm("この大会を削除しますか？")) return;

    try {
      const res = await fetch(`/api/competitions/${id}`, {
        method: "DELETE",
      });

      if (!res.ok) throw new Error("Failed to delete");

      refetch(); // データを再取得
    } catch {
      alert("削除に失敗しました");
    }
  };

  if (loading) {
    return <div>読み込み中...</div>;
  }

  if (error) {
    return <div className="text-red-600">{error}</div>;
  }

  const columns = [
    { key: "name", header: "大会名" },
    {
      key: "sportType",
      header: "競技",
      render: (item: Competition) => {
        const types: Record<string, string> = {
          football: "フットサル",
          tabletennis: "卓球",
          other: "その他",
        };
        return types[item.sportType];
      },
    },
    { key: "season", header: "シーズン" },
    {
      key: "createdAt",
      header: "作成日",
      render: (item: Competition) => formatDate(item.createdAt),
    },
    {
      key: "actions",
      header: "操作",
      render: (item: Competition) => (
        <div className="flex gap-2">
          <Link href={`/competitions/${item.id}`}>
            <Button variant="secondary">詳細</Button>
          </Link>
          <Link href={`/competitions/${item.id}/edit`}>
            <Button variant="secondary">編集</Button>
          </Link>
          <Button variant="danger" onClick={() => handleDelete(item.id)}>
            削除
          </Button>
        </div>
      ),
    },
  ];

  return (
    <div>
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold">大会一覧</h1>
        <Link href="/competitions/new">
          <Button>新規作成</Button>
        </Link>
      </div>

      <Card>
        <Table
          columns={columns}
          data={competitions}
          emptyMessage="大会がありません"
        />
      </Card>
    </div>
  );
}
```

### フォームの修正

`app/competitions/new/page.tsx`:

```tsx
"use client";

import { useState } from "react";
import { useRouter } from "next/navigation";
import { Button, Input, Select, Card } from "@/components/ui";

export default function NewCompetitionPage() {
  const router = useRouter();

  const [name, setName] = useState("");
  const [sportType, setSportType] = useState<"football" | "tabletennis" | "other">("football");
  const [season, setSeason] = useState("");
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    setError("");

    try {
      const res = await fetch("/api/competitions", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          name,
          sportType,
          season: season || undefined,
        }),
      });

      if (!res.ok) {
        const data = await res.json();
        throw new Error(data.error || "Failed to create");
      }

      router.push("/competitions");
    } catch (err) {
      setError(err instanceof Error ? err.message : "作成に失敗しました");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <h1 className="text-2xl font-bold mb-6">大会の新規作成</h1>

      <Card>
        {error && (
          <div className="mb-4 p-3 bg-red-100 text-red-700 rounded">
            {error}
          </div>
        )}

        <form onSubmit={handleSubmit}>
          <Input
            label="大会名"
            name="name"
            value={name}
            onChange={(e) => setName(e.target.value)}
            required
          />

          <Select
            label="競技"
            name="sportType"
            value={sportType}
            onChange={(e) => setSportType(e.target.value as typeof sportType)}
            options={[
              { value: "football", label: "フットサル" },
              { value: "tabletennis", label: "卓球" },
              { value: "other", label: "その他" },
            ]}
          />

          <Input
            label="シーズン"
            name="season"
            value={season}
            onChange={(e) => setSeason(e.target.value)}
          />

          <div className="flex gap-2 mt-6">
            <Button type="submit" disabled={loading}>
              {loading ? "作成中..." : "作成"}
            </Button>
            <Button
              type="button"
              variant="secondary"
              onClick={() => router.back()}
            >
              キャンセル
            </Button>
          </div>
        </form>
      </Card>
    </div>
  );
}
```

## 移行のポイント

### 1. エラーハンドリング

API からエラーが返ってくる可能性があるため、適切にハンドリングします。

```typescript
try {
  const res = await fetch("/api/...");
  if (!res.ok) {
    const data = await res.json();
    throw new Error(data.error);
  }
  // 成功処理
} catch (error) {
  // エラー表示
}
```

### 2. ローディング状態

API 呼び出しは非同期なので、ローディング状態を表示します。

### 3. データの再取得

データを変更したら、一覧を再取得して表示を更新します。

## Git でコミット

```bash
git add .
git commit -m "localStorage から Prisma への移行"
```

## この章のまとめ

- API Routes を作成した
- フロントエンドを fetch ベースに修正した
- エラーハンドリングを追加した

## 確認してみよう

- [ ] 大会の一覧が表示される
- [ ] 大会を作成できる
- [ ] 大会を編集・削除できる
- [ ] ページ再読み込み後もデータが残っている
