---
title: "試合管理機能の実装"
---

## この章でやること

- 試合の登録
- スコアの入力
- 試合一覧の表示

## データ管理関数の追加

`lib/data.ts` に試合用の関数を追加します。

```typescript
// --- 試合 ---

export function getMatches(): Match[] {
  if (typeof window === "undefined") return [];
  const data = localStorage.getItem(STORAGE_KEYS.matches);
  return data ? JSON.parse(data) : [];
}

export function getMatchesByCompetition(competitionId: string): Match[] {
  const matches = getMatches();
  return matches.filter((m) => m.competitionId === competitionId);
}

export function getMatch(id: string): Match | undefined {
  const matches = getMatches();
  return matches.find((m) => m.id === id);
}

export function saveMatch(
  data: Omit<Match, "id" | "createdAt" | "updatedAt">
): Match {
  const matches = getMatches();
  const now = getCurrentTimestamp();

  const newMatch: Match = {
    ...data,
    id: generateId(),
    createdAt: now,
    updatedAt: now,
  };

  matches.push(newMatch);
  localStorage.setItem(STORAGE_KEYS.matches, JSON.stringify(matches));

  return newMatch;
}

export function updateMatch(
  id: string,
  data: Partial<Omit<Match, "id" | "createdAt" | "updatedAt">>
): Match | undefined {
  const matches = getMatches();
  const index = matches.findIndex((m) => m.id === id);

  if (index === -1) return undefined;

  matches[index] = {
    ...matches[index],
    ...data,
    updatedAt: getCurrentTimestamp(),
  };

  localStorage.setItem(STORAGE_KEYS.matches, JSON.stringify(matches));

  return matches[index];
}

export function deleteMatch(id: string): boolean {
  const matches = getMatches();
  const filtered = matches.filter((m) => m.id !== id);

  if (filtered.length === matches.length) return false;

  localStorage.setItem(STORAGE_KEYS.matches, JSON.stringify(filtered));
  return true;
}
```

## 試合作成ページ

大会詳細から試合を作成できるようにします。

`app/competitions/[id]/matches/new/page.tsx`:

```tsx
"use client";

import { useState, useEffect } from "react";
import { useRouter, useParams } from "next/navigation";
import { getCompetition, getTeams, saveMatch } from "@/lib/data";
import { Team, Competition } from "@/lib/types";
import { Button, Input, Select, Card } from "@/components/ui";
import { formatDateTimeForInput, getCurrentTimestamp } from "@/lib/utils";

export default function NewMatchPage() {
  const router = useRouter();
  const params = useParams();
  const competitionId = params.id as string;

  const [competition, setCompetition] = useState<Competition | null>(null);
  const [teams, setTeams] = useState<Team[]>([]);
  const [homeTeamId, setHomeTeamId] = useState("");
  const [awayTeamId, setAwayTeamId] = useState("");
  const [startAt, setStartAt] = useState(formatDateTimeForInput(getCurrentTimestamp()));
  const [venue, setVenue] = useState("");

  useEffect(() => {
    setCompetition(getCompetition(competitionId) || null);
    setTeams(getTeams());
  }, [competitionId]);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    if (homeTeamId === awayTeamId) {
      alert("ホームとアウェイには異なるチームを選択してください");
      return;
    }

    saveMatch({
      competitionId,
      homeTeamId,
      awayTeamId,
      homeScore: null,
      awayScore: null,
      status: "scheduled",
      startAt: new Date(startAt).toISOString(),
      venue: venue || undefined,
    });

    router.push(`/competitions/${competitionId}`);
  };

  const teamOptions = teams.map((team) => ({
    value: team.id,
    label: team.name,
  }));

  if (!competition) {
    return <div>大会が見つかりません</div>;
  }

  return (
    <div>
      <h1 className="text-2xl font-bold mb-6">試合の登録</h1>
      <p className="text-gray-600 mb-6">{competition.name}</p>

      <Card>
        <form onSubmit={handleSubmit}>
          <Select
            label="ホームチーム"
            name="homeTeamId"
            value={homeTeamId}
            onChange={(e) => setHomeTeamId(e.target.value)}
            options={teamOptions}
            required
          />

          <Select
            label="アウェイチーム"
            name="awayTeamId"
            value={awayTeamId}
            onChange={(e) => setAwayTeamId(e.target.value)}
            options={teamOptions}
            required
          />

          <Input
            label="試合日時"
            name="startAt"
            type="datetime-local"
            value={startAt}
            onChange={(e) => setStartAt(e.target.value)}
            required
          />

          <Input
            label="会場"
            name="venue"
            value={venue}
            onChange={(e) => setVenue(e.target.value)}
            placeholder="例: 市民体育館"
          />

          <div className="flex gap-2 mt-6">
            <Button type="submit">登録</Button>
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

## 試合編集ページ（スコア入力）

`app/competitions/[id]/matches/[matchId]/edit/page.tsx`:

```tsx
"use client";

import { useState, useEffect } from "react";
import { useRouter, useParams } from "next/navigation";
import { getMatch, getTeams, updateMatch } from "@/lib/data";
import { Match, Team } from "@/lib/types";
import { Button, Input, Select, Card } from "@/components/ui";
import { formatDateTimeForInput } from "@/lib/utils";

export default function EditMatchPage() {
  const router = useRouter();
  const params = useParams();
  const competitionId = params.id as string;
  const matchId = params.matchId as string;

  const [match, setMatch] = useState<Match | null>(null);
  const [teams, setTeams] = useState<Team[]>([]);
  const [homeScore, setHomeScore] = useState<string>("");
  const [awayScore, setAwayScore] = useState<string>("");
  const [status, setStatus] = useState<"scheduled" | "finished">("scheduled");
  const [startAt, setStartAt] = useState("");
  const [venue, setVenue] = useState("");
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const matchData = getMatch(matchId);
    const teamsData = getTeams();

    if (matchData) {
      setMatch(matchData);
      setHomeScore(matchData.homeScore?.toString() || "");
      setAwayScore(matchData.awayScore?.toString() || "");
      setStatus(matchData.status);
      setStartAt(formatDateTimeForInput(matchData.startAt));
      setVenue(matchData.venue || "");
    }

    setTeams(teamsData);
    setLoading(false);
  }, [matchId]);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    const homeScoreNum = homeScore === "" ? null : parseInt(homeScore, 10);
    const awayScoreNum = awayScore === "" ? null : parseInt(awayScore, 10);

    updateMatch(matchId, {
      homeScore: homeScoreNum,
      awayScore: awayScoreNum,
      status,
      startAt: new Date(startAt).toISOString(),
      venue: venue || undefined,
    });

    router.push(`/competitions/${competitionId}`);
  };

  if (loading) {
    return <div>読み込み中...</div>;
  }

  if (!match) {
    return <div>試合が見つかりません</div>;
  }

  const getTeamName = (teamId: string) => {
    const team = teams.find((t) => t.id === teamId);
    return team?.name || "不明";
  };

  return (
    <div>
      <h1 className="text-2xl font-bold mb-6">試合の編集</h1>

      <Card>
        <div className="mb-6 p-4 bg-gray-50 rounded">
          <div className="text-center">
            <span className="font-bold">{getTeamName(match.homeTeamId)}</span>
            <span className="mx-4">vs</span>
            <span className="font-bold">{getTeamName(match.awayTeamId)}</span>
          </div>
        </div>

        <form onSubmit={handleSubmit}>
          <div className="grid grid-cols-2 gap-4">
            <Input
              label={`${getTeamName(match.homeTeamId)} の得点`}
              name="homeScore"
              type="number"
              value={homeScore}
              onChange={(e) => setHomeScore(e.target.value)}
              placeholder="0"
            />

            <Input
              label={`${getTeamName(match.awayTeamId)} の得点`}
              name="awayScore"
              type="number"
              value={awayScore}
              onChange={(e) => setAwayScore(e.target.value)}
              placeholder="0"
            />
          </div>

          <Select
            label="試合状態"
            name="status"
            value={status}
            onChange={(e) => setStatus(e.target.value as typeof status)}
            options={[
              { value: "scheduled", label: "予定" },
              { value: "finished", label: "終了" },
            ]}
          />

          <Input
            label="試合日時"
            name="startAt"
            type="datetime-local"
            value={startAt}
            onChange={(e) => setStartAt(e.target.value)}
          />

          <Input
            label="会場"
            name="venue"
            value={venue}
            onChange={(e) => setVenue(e.target.value)}
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

## 大会詳細ページに試合一覧を追加

`app/competitions/[id]/page.tsx` を更新します。

```tsx
"use client";

import { useState, useEffect } from "react";
import { useParams } from "next/navigation";
import Link from "next/link";
import { Competition, Match, Team } from "@/lib/types";
import { getCompetition, getMatchesByCompetition, getTeams } from "@/lib/data";
import { Button, Card, Table } from "@/components/ui";
import { formatDate } from "@/lib/utils";

export default function CompetitionDetailPage() {
  const params = useParams();
  const id = params.id as string;

  const [competition, setCompetition] = useState<Competition | null>(null);
  const [matches, setMatches] = useState<Match[]>([]);
  const [teams, setTeams] = useState<Team[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setCompetition(getCompetition(id) || null);
    setMatches(getMatchesByCompetition(id));
    setTeams(getTeams());
    setLoading(false);
  }, [id]);

  if (loading) {
    return <div>読み込み中...</div>;
  }

  if (!competition) {
    return <div>大会が見つかりません</div>;
  }

  const getTeamName = (teamId: string) => {
    const team = teams.find((t) => t.id === teamId);
    return team?.name || "不明";
  };

  const matchColumns = [
    {
      key: "match",
      header: "対戦",
      render: (item: Match) => (
        <span>
          {getTeamName(item.homeTeamId)} vs {getTeamName(item.awayTeamId)}
        </span>
      ),
    },
    {
      key: "score",
      header: "スコア",
      render: (item: Match) =>
        item.status === "finished"
          ? `${item.homeScore} - ${item.awayScore}`
          : "-",
    },
    {
      key: "status",
      header: "状態",
      render: (item: Match) => (
        <span
          className={`px-2 py-1 rounded text-xs ${
            item.status === "finished"
              ? "bg-green-100 text-green-800"
              : "bg-gray-100 text-gray-800"
          }`}
        >
          {item.status === "finished" ? "終了" : "予定"}
        </span>
      ),
    },
    {
      key: "startAt",
      header: "日時",
      render: (item: Match) => formatDate(item.startAt),
    },
    {
      key: "actions",
      header: "操作",
      render: (item: Match) => (
        <Link href={`/competitions/${id}/matches/${item.id}/edit`}>
          <Button variant="secondary">編集</Button>
        </Link>
      ),
    },
  ];

  return (
    <div>
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold">{competition.name}</h1>
        <Link href={`/competitions/${id}/edit`}>
          <Button variant="secondary">大会を編集</Button>
        </Link>
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-6">
        <Card title="基本情報">
          <dl className="space-y-2">
            <div>
              <dt className="text-sm text-gray-500">競技</dt>
              <dd>
                {competition.sportType === "football" && "フットサル"}
                {competition.sportType === "tabletennis" && "卓球"}
                {competition.sportType === "other" && "その他"}
              </dd>
            </div>
            {competition.season && (
              <div>
                <dt className="text-sm text-gray-500">シーズン</dt>
                <dd>{competition.season}</dd>
              </div>
            )}
          </dl>
        </Card>

        <Card title="順位表">
          <p className="text-gray-500">次の章で実装します</p>
        </Card>
      </div>

      <Card title="試合一覧">
        <div className="mb-4">
          <Link href={`/competitions/${id}/matches/new`}>
            <Button>試合を追加</Button>
          </Link>
        </div>
        <Table
          columns={matchColumns}
          data={matches}
          emptyMessage="試合がありません"
        />
      </Card>
    </div>
  );
}
```

## 動作確認

1. 大会詳細ページを開く
2. 「試合を追加」をクリック
3. ホーム/アウェイチームを選択して登録
4. 試合一覧に表示されることを確認
5. 「編集」からスコアを入力
6. 状態を「終了」に変更

## Git でコミット

```bash
git add .
git commit -m "試合管理機能を実装"
```

## この章のまとめ

- 試合の登録・編集機能を実装した
- スコア入力と試合状態の管理ができるようになった
- 大会詳細ページに試合一覧を表示した

## 確認してみよう

- [ ] 試合を登録できる
- [ ] 同じチーム同士の試合は登録できない
- [ ] スコアを入力できる
- [ ] 試合状態を変更できる
- [ ] 大会詳細に試合一覧が表示される
