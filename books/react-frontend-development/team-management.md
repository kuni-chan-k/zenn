---
title: "チーム管理機能の実装"
---

## この章でやること

- チームの一覧表示・作成・編集・削除
- Context API でデータを共有

## データ管理関数の追加

`lib/data.ts` にチーム用の関数を追加します。

```typescript
// --- チーム ---

export function getTeams(): Team[] {
  if (typeof window === "undefined") return [];
  const data = localStorage.getItem(STORAGE_KEYS.teams);
  return data ? JSON.parse(data) : [];
}

export function getTeam(id: string): Team | undefined {
  const teams = getTeams();
  return teams.find((t) => t.id === id);
}

export function saveTeam(
  data: Omit<Team, "id" | "createdAt" | "updatedAt">
): Team {
  const teams = getTeams();
  const now = getCurrentTimestamp();

  const newTeam: Team = {
    ...data,
    id: generateId(),
    createdAt: now,
    updatedAt: now,
  };

  teams.push(newTeam);
  localStorage.setItem(STORAGE_KEYS.teams, JSON.stringify(teams));

  return newTeam;
}

export function updateTeam(
  id: string,
  data: Partial<Omit<Team, "id" | "createdAt" | "updatedAt">>
): Team | undefined {
  const teams = getTeams();
  const index = teams.findIndex((t) => t.id === id);

  if (index === -1) return undefined;

  teams[index] = {
    ...teams[index],
    ...data,
    updatedAt: getCurrentTimestamp(),
  };

  localStorage.setItem(STORAGE_KEYS.teams, JSON.stringify(teams));

  return teams[index];
}

export function deleteTeam(id: string): boolean {
  const teams = getTeams();
  const filtered = teams.filter((t) => t.id !== id);

  if (filtered.length === teams.length) return false;

  localStorage.setItem(STORAGE_KEYS.teams, JSON.stringify(filtered));
  return true;
}
```

## チーム一覧ページ

`app/teams/page.tsx`:

```tsx
"use client";

import { useState, useEffect } from "react";
import Link from "next/link";
import { Team } from "@/lib/types";
import { getTeams, deleteTeam } from "@/lib/data";
import { Button, Card, Table } from "@/components/ui";
import { formatDate } from "@/lib/utils";

export default function TeamsPage() {
  const [teams, setTeams] = useState<Team[]>([]);

  useEffect(() => {
    setTeams(getTeams());
  }, []);

  const handleDelete = (id: string) => {
    if (!confirm("このチームを削除しますか？")) return;

    deleteTeam(id);
    setTeams(getTeams());
  };

  const columns = [
    { key: "name", header: "チーム名" },
    { key: "shortName", header: "略称" },
    {
      key: "createdAt",
      header: "登録日",
      render: (item: Team) => formatDate(item.createdAt),
    },
    {
      key: "actions",
      header: "操作",
      render: (item: Team) => (
        <div className="flex gap-2">
          <Link href={`/teams/${item.id}/edit`}>
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
        <h1 className="text-2xl font-bold">チーム一覧</h1>
        <Link href="/teams/new">
          <Button>新規作成</Button>
        </Link>
      </div>

      <Card>
        <Table
          columns={columns}
          data={teams}
          emptyMessage="チームがありません。「新規作成」から追加してください。"
        />
      </Card>
    </div>
  );
}
```

## チーム作成ページ

`app/teams/new/page.tsx`:

```tsx
"use client";

import { useState } from "react";
import { useRouter } from "next/navigation";
import { saveTeam } from "@/lib/data";
import { Button, Input, Card } from "@/components/ui";

export default function NewTeamPage() {
  const router = useRouter();

  const [name, setName] = useState("");
  const [shortName, setShortName] = useState("");

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    saveTeam({
      name,
      shortName: shortName || undefined,
    });

    router.push("/teams");
  };

  return (
    <div>
      <h1 className="text-2xl font-bold mb-6">チームの新規作成</h1>

      <Card>
        <form onSubmit={handleSubmit}>
          <Input
            label="チーム名"
            name="name"
            value={name}
            onChange={(e) => setName(e.target.value)}
            required
            placeholder="例: FC レッドスター"
          />

          <Input
            label="略称"
            name="shortName"
            value={shortName}
            onChange={(e) => setShortName(e.target.value)}
            placeholder="例: RST"
          />

          <div className="flex gap-2 mt-6">
            <Button type="submit">作成</Button>
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

## チーム編集ページ

`app/teams/[id]/edit/page.tsx`:

```tsx
"use client";

import { useState, useEffect } from "react";
import { useRouter, useParams } from "next/navigation";
import { getTeam, updateTeam } from "@/lib/data";
import { Button, Input, Card } from "@/components/ui";

export default function EditTeamPage() {
  const router = useRouter();
  const params = useParams();
  const id = params.id as string;

  const [name, setName] = useState("");
  const [shortName, setShortName] = useState("");
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const team = getTeam(id);
    if (team) {
      setName(team.name);
      setShortName(team.shortName || "");
    }
    setLoading(false);
  }, [id]);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    updateTeam(id, {
      name,
      shortName: shortName || undefined,
    });

    router.push("/teams");
  };

  if (loading) {
    return <div>読み込み中...</div>;
  }

  return (
    <div>
      <h1 className="text-2xl font-bold mb-6">チームの編集</h1>

      <Card>
        <form onSubmit={handleSubmit}>
          <Input
            label="チーム名"
            name="name"
            value={name}
            onChange={(e) => setName(e.target.value)}
            required
          />

          <Input
            label="略称"
            name="shortName"
            value={shortName}
            onChange={(e) => setShortName(e.target.value)}
          />

          <div className="flex gap-2 mt-6">
            <Button type="submit">保存</Button>
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

## サンプルチームの登録

動作確認のために、以下のチームを登録してみましょう。

1. **FC レッドスター**（略称: RST）
2. **ブルーウィングス**（略称: BWG）
3. **グリーンファイターズ**（略称: GFT）
4. **イエローサンダー**（略称: YTH）

## ダッシュボードの更新

ダッシュボードに実際の数を表示しましょう。

`app/page.tsx`:

```tsx
"use client";

import { useState, useEffect } from "react";
import { getCompetitions, getTeams, getMatches } from "@/lib/data";

export default function Home() {
  const [stats, setStats] = useState({
    competitions: 0,
    teams: 0,
    matches: 0,
  });

  useEffect(() => {
    setStats({
      competitions: getCompetitions().length,
      teams: getTeams().length,
      matches: getMatches().length,
    });
  }, []);

  return (
    <div>
      <h1 className="text-2xl font-bold mb-6">ダッシュボード</h1>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        <div className="bg-white p-6 rounded-lg shadow">
          <h2 className="text-lg font-semibold text-gray-700">大会数</h2>
          <p className="text-3xl font-bold text-blue-600 mt-2">
            {stats.competitions}
          </p>
        </div>

        <div className="bg-white p-6 rounded-lg shadow">
          <h2 className="text-lg font-semibold text-gray-700">チーム数</h2>
          <p className="text-3xl font-bold text-green-600 mt-2">
            {stats.teams}
          </p>
        </div>

        <div className="bg-white p-6 rounded-lg shadow">
          <h2 className="text-lg font-semibold text-gray-700">試合数</h2>
          <p className="text-3xl font-bold text-orange-600 mt-2">
            {stats.matches}
          </p>
        </div>
      </div>
    </div>
  );
}
```

`lib/data.ts` に `getMatches` 関数を追加（まだ空配列を返す）:

```typescript
// --- 試合 ---

export function getMatches(): Match[] {
  if (typeof window === "undefined") return [];
  const data = localStorage.getItem(STORAGE_KEYS.matches);
  return data ? JSON.parse(data) : [];
}
```

## Git でコミット

```bash
git add .
git commit -m "チーム管理機能を実装"
```

## この章のまとめ

- チームの CRUD を実装した
- 大会と同様のパターンで実装できた
- ダッシュボードに統計情報を表示した

## 確認してみよう

- [ ] チームを新規作成できる
- [ ] チーム一覧が表示される
- [ ] チームを編集できる
- [ ] チームを削除できる
- [ ] ダッシュボードに正しい数が表示される
