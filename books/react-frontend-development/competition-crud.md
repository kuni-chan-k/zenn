---
title: "大会管理機能の実装"
---

## この章でやること

- 大会の一覧表示
- 大会の新規作成
- 大会の編集・削除
- データの保存（localStorage）

## データの管理方法

このアプリでは、データを **localStorage** に保存します。

localStorage は、ブラウザにデータを保存できる仕組みです。ページを閉じてもデータが残ります。

:::message
本番のアプリでは、サーバーやデータベースにデータを保存します。この本では学習のため localStorage を使いますが、後で Firebase などに差し替えることもできます。
:::

## データ管理の仕組みを作る

`lib/data.ts`:

```typescript
import { Competition, Team, Match } from "./types";
import { generateId, getCurrentTimestamp } from "./utils";

// localStorage のキー
const STORAGE_KEYS = {
  competitions: "match-manager-competitions",
  teams: "match-manager-teams",
  matches: "match-manager-matches",
} as const;

// --- 大会 ---

export function getCompetitions(): Competition[] {
  if (typeof window === "undefined") return [];
  const data = localStorage.getItem(STORAGE_KEYS.competitions);
  return data ? JSON.parse(data) : [];
}

export function getCompetition(id: string): Competition | undefined {
  const competitions = getCompetitions();
  return competitions.find((c) => c.id === id);
}

export function saveCompetition(
  data: Omit<Competition, "id" | "createdAt" | "updatedAt">
): Competition {
  const competitions = getCompetitions();
  const now = getCurrentTimestamp();

  const newCompetition: Competition = {
    ...data,
    id: generateId(),
    createdAt: now,
    updatedAt: now,
  };

  competitions.push(newCompetition);
  localStorage.setItem(STORAGE_KEYS.competitions, JSON.stringify(competitions));

  return newCompetition;
}

export function updateCompetition(
  id: string,
  data: Partial<Omit<Competition, "id" | "createdAt" | "updatedAt">>
): Competition | undefined {
  const competitions = getCompetitions();
  const index = competitions.findIndex((c) => c.id === id);

  if (index === -1) return undefined;

  competitions[index] = {
    ...competitions[index],
    ...data,
    updatedAt: getCurrentTimestamp(),
  };

  localStorage.setItem(STORAGE_KEYS.competitions, JSON.stringify(competitions));

  return competitions[index];
}

export function deleteCompetition(id: string): boolean {
  const competitions = getCompetitions();
  const filtered = competitions.filter((c) => c.id !== id);

  if (filtered.length === competitions.length) return false;

  localStorage.setItem(STORAGE_KEYS.competitions, JSON.stringify(filtered));
  return true;
}
```

### ポイント解説

- `Omit<Competition, "id" | "createdAt">`: Competition 型から id と createdAt を除いた型
- `Partial<...>`: すべてのプロパティを省略可能にする
- `JSON.parse` / `JSON.stringify`: オブジェクトと文字列の相互変換

## 大会一覧ページ

`app/competitions/page.tsx`:

```tsx
"use client";

import { useState, useEffect } from "react";
import Link from "next/link";
import { Competition } from "@/lib/types";
import { getCompetitions, deleteCompetition } from "@/lib/data";
import { Button, Card, Table } from "@/components/ui";
import { formatDate } from "@/lib/utils";

export default function CompetitionsPage() {
  const [competitions, setCompetitions] = useState<Competition[]>([]);

  useEffect(() => {
    setCompetitions(getCompetitions());
  }, []);

  const handleDelete = (id: string) => {
    if (!confirm("この大会を削除しますか？")) return;

    deleteCompetition(id);
    setCompetitions(getCompetitions());
  };

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
          emptyMessage="大会がありません。「新規作成」から追加してください。"
        />
      </Card>
    </div>
  );
}
```

## 大会作成ページ

`app/competitions/new/page.tsx`:

```tsx
"use client";

import { useState } from "react";
import { useRouter } from "next/navigation";
import { saveCompetition } from "@/lib/data";
import { Button, Input, Select, Card } from "@/components/ui";

export default function NewCompetitionPage() {
  const router = useRouter();

  const [name, setName] = useState("");
  const [sportType, setSportType] = useState<"football" | "tabletennis" | "other">("football");
  const [season, setSeason] = useState("");

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    saveCompetition({
      name,
      sportType,
      season: season || undefined,
    });

    router.push("/competitions");
  };

  return (
    <div>
      <h1 className="text-2xl font-bold mb-6">大会の新規作成</h1>

      <Card>
        <form onSubmit={handleSubmit}>
          <Input
            label="大会名"
            name="name"
            value={name}
            onChange={(e) => setName(e.target.value)}
            required
            placeholder="例: 春季フットサルリーグ 2024"
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
            required
          />

          <Input
            label="シーズン"
            name="season"
            value={season}
            onChange={(e) => setSeason(e.target.value)}
            placeholder="例: 2024春"
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

## 大会編集ページ

`app/competitions/[id]/edit/page.tsx`:

```tsx
"use client";

import { useState, useEffect } from "react";
import { useRouter, useParams } from "next/navigation";
import { getCompetition, updateCompetition } from "@/lib/data";
import { Button, Input, Select, Card } from "@/components/ui";

export default function EditCompetitionPage() {
  const router = useRouter();
  const params = useParams();
  const id = params.id as string;

  const [name, setName] = useState("");
  const [sportType, setSportType] = useState<"football" | "tabletennis" | "other">("football");
  const [season, setSeason] = useState("");
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const competition = getCompetition(id);
    if (competition) {
      setName(competition.name);
      setSportType(competition.sportType);
      setSeason(competition.season || "");
    }
    setLoading(false);
  }, [id]);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    updateCompetition(id, {
      name,
      sportType,
      season: season || undefined,
    });

    router.push("/competitions");
  };

  if (loading) {
    return <div>読み込み中...</div>;
  }

  return (
    <div>
      <h1 className="text-2xl font-bold mb-6">大会の編集</h1>

      <Card>
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
            required
          />

          <Input
            label="シーズン"
            name="season"
            value={season}
            onChange={(e) => setSeason(e.target.value)}
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

## 大会詳細ページ

`app/competitions/[id]/page.tsx`:

```tsx
"use client";

import { useState, useEffect } from "react";
import { useParams } from "next/navigation";
import Link from "next/link";
import { Competition } from "@/lib/types";
import { getCompetition } from "@/lib/data";
import { Button, Card } from "@/components/ui";
import { formatDate } from "@/lib/utils";

export default function CompetitionDetailPage() {
  const params = useParams();
  const id = params.id as string;

  const [competition, setCompetition] = useState<Competition | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const data = getCompetition(id);
    setCompetition(data || null);
    setLoading(false);
  }, [id]);

  if (loading) {
    return <div>読み込み中...</div>;
  }

  if (!competition) {
    return <div>大会が見つかりません</div>;
  }

  const sportTypeLabels: Record<string, string> = {
    football: "フットサル",
    tabletennis: "卓球",
    other: "その他",
  };

  return (
    <div>
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold">{competition.name}</h1>
        <Link href={`/competitions/${id}/edit`}>
          <Button>編集</Button>
        </Link>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        <Card title="基本情報">
          <dl className="space-y-2">
            <div>
              <dt className="text-sm text-gray-500">競技</dt>
              <dd>{sportTypeLabels[competition.sportType]}</dd>
            </div>
            {competition.season && (
              <div>
                <dt className="text-sm text-gray-500">シーズン</dt>
                <dd>{competition.season}</dd>
              </div>
            )}
            <div>
              <dt className="text-sm text-gray-500">作成日</dt>
              <dd>{formatDate(competition.createdAt)}</dd>
            </div>
          </dl>
        </Card>

        <Card title="順位表">
          <p className="text-gray-500">試合結果を登録すると順位表が表示されます</p>
        </Card>
      </div>

      <div className="mt-6">
        <Card title="試合一覧">
          <p className="text-gray-500">試合がありません</p>
        </Card>
      </div>
    </div>
  );
}
```

## 動作確認

1. 「大会」ページを開く
2. 「新規作成」をクリックして大会を作成
3. 一覧に表示されることを確認
4. 「編集」で内容を変更
5. 「削除」で削除
6. ページを再読み込みしてもデータが残っていることを確認

## Git でコミット

```bash
git add .
git commit -m "大会管理機能を実装"
```

## この章のまとめ

- localStorage でデータを永続化した
- 大会の CRUD（作成・読取・更新・削除）を実装した
- 動的ルーティング（`[id]`）でページを作成した

## 確認してみよう

- [ ] 大会を新規作成できる
- [ ] 大会一覧が表示される
- [ ] 大会を編集できる
- [ ] 大会を削除できる
- [ ] ページを再読み込みしてもデータが残る
