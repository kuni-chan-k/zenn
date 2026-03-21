---
title: "順位表の実装"
---

## この章でやること

- 勝点計算ロジックの実装
- 順位表の表示
- 同率時のソート処理

## 勝点のルール

フットサル（サッカー）の一般的な勝点ルールは以下の通りです。

| 結果 | 勝点 |
|------|------|
| 勝利 | 3点 |
| 引分 | 1点 |
| 敗北 | 0点 |

同じ勝点の場合は、以下の順で順位を決めます。

1. 得失点差（得点 - 失点）が大きい方が上
2. 総得点が多い方が上
3. チーム名の昇順

## 順位計算関数の実装

`lib/standings.ts`:

```typescript
import { Match, Standing, Team } from "./types";

type StandingInput = {
  teams: Team[];
  matches: Match[];
};

export function calculateStandings({ teams, matches }: StandingInput): Standing[] {
  // 終了した試合のみを対象にする
  const finishedMatches = matches.filter((m) => m.status === "finished");

  // 各チームの成績を初期化
  const standingsMap = new Map<string, Standing>();

  teams.forEach((team) => {
    standingsMap.set(team.id, {
      teamId: team.id,
      teamName: team.name,
      played: 0,
      won: 0,
      drawn: 0,
      lost: 0,
      goalsFor: 0,
      goalsAgainst: 0,
      goalDiff: 0,
      points: 0,
      rank: 0,
    });
  });

  // 試合結果を集計
  finishedMatches.forEach((match) => {
    const homeStanding = standingsMap.get(match.homeTeamId);
    const awayStanding = standingsMap.get(match.awayTeamId);

    if (!homeStanding || !awayStanding) return;
    if (match.homeScore === null || match.awayScore === null) return;

    // 試合数を加算
    homeStanding.played += 1;
    awayStanding.played += 1;

    // 得点・失点を加算
    homeStanding.goalsFor += match.homeScore;
    homeStanding.goalsAgainst += match.awayScore;
    awayStanding.goalsFor += match.awayScore;
    awayStanding.goalsAgainst += match.homeScore;

    // 勝敗を判定
    if (match.homeScore > match.awayScore) {
      // ホーム勝利
      homeStanding.won += 1;
      homeStanding.points += 3;
      awayStanding.lost += 1;
    } else if (match.homeScore < match.awayScore) {
      // アウェイ勝利
      awayStanding.won += 1;
      awayStanding.points += 3;
      homeStanding.lost += 1;
    } else {
      // 引分
      homeStanding.drawn += 1;
      homeStanding.points += 1;
      awayStanding.drawn += 1;
      awayStanding.points += 1;
    }
  });

  // 得失点差を計算
  standingsMap.forEach((standing) => {
    standing.goalDiff = standing.goalsFor - standing.goalsAgainst;
  });

  // 順位でソート
  const sortedStandings = Array.from(standingsMap.values()).sort((a, b) => {
    // 1. 勝点で比較（降順）
    if (b.points !== a.points) {
      return b.points - a.points;
    }
    // 2. 得失点差で比較（降順）
    if (b.goalDiff !== a.goalDiff) {
      return b.goalDiff - a.goalDiff;
    }
    // 3. 総得点で比較（降順）
    if (b.goalsFor !== a.goalsFor) {
      return b.goalsFor - a.goalsFor;
    }
    // 4. チーム名で比較（昇順）
    return a.teamName.localeCompare(b.teamName, "ja");
  });

  // 順位を付与
  sortedStandings.forEach((standing, index) => {
    standing.rank = index + 1;
  });

  return sortedStandings;
}
```

### ポイント解説

1. **Map を使った集計**: 各チームの成績を効率的に管理
2. **nullチェック**: スコアが未入力の試合は除外
3. **多段ソート**: 勝点 → 得失点差 → 得点 → チーム名の順
4. **localeCompare**: 日本語のチーム名を正しくソート

## 順位表コンポーネント

`components/StandingsTable.tsx`:

```tsx
import { Standing } from "@/lib/types";

type StandingsTableProps = {
  standings: Standing[];
};

export default function StandingsTable({ standings }: StandingsTableProps) {
  if (standings.length === 0) {
    return (
      <p className="text-gray-500 text-center py-4">
        チームがありません
      </p>
    );
  }

  return (
    <div className="overflow-x-auto">
      <table className="w-full border-collapse text-sm">
        <thead>
          <tr className="bg-gray-100">
            <th className="px-3 py-2 text-left">順位</th>
            <th className="px-3 py-2 text-left">チーム</th>
            <th className="px-3 py-2 text-center">試合</th>
            <th className="px-3 py-2 text-center">勝</th>
            <th className="px-3 py-2 text-center">分</th>
            <th className="px-3 py-2 text-center">負</th>
            <th className="px-3 py-2 text-center">得点</th>
            <th className="px-3 py-2 text-center">失点</th>
            <th className="px-3 py-2 text-center">得失</th>
            <th className="px-3 py-2 text-center font-bold">勝点</th>
          </tr>
        </thead>
        <tbody>
          {standings.map((standing) => (
            <tr
              key={standing.teamId}
              className="border-b hover:bg-gray-50"
            >
              <td className="px-3 py-2 font-bold">{standing.rank}</td>
              <td className="px-3 py-2">{standing.teamName}</td>
              <td className="px-3 py-2 text-center">{standing.played}</td>
              <td className="px-3 py-2 text-center">{standing.won}</td>
              <td className="px-3 py-2 text-center">{standing.drawn}</td>
              <td className="px-3 py-2 text-center">{standing.lost}</td>
              <td className="px-3 py-2 text-center">{standing.goalsFor}</td>
              <td className="px-3 py-2 text-center">{standing.goalsAgainst}</td>
              <td className="px-3 py-2 text-center">
                {standing.goalDiff > 0 && "+"}
                {standing.goalDiff}
              </td>
              <td className="px-3 py-2 text-center font-bold text-blue-600">
                {standing.points}
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

## 大会詳細ページに順位表を追加

`app/competitions/[id]/page.tsx` を更新します。

```tsx
"use client";

import { useState, useEffect } from "react";
import { useParams } from "next/navigation";
import Link from "next/link";
import { Competition, Match, Team, Standing } from "@/lib/types";
import { getCompetition, getMatchesByCompetition, getTeams } from "@/lib/data";
import { calculateStandings } from "@/lib/standings";
import { Button, Card, Table } from "@/components/ui";
import StandingsTable from "@/components/StandingsTable";
import { formatDate } from "@/lib/utils";

export default function CompetitionDetailPage() {
  const params = useParams();
  const id = params.id as string;

  const [competition, setCompetition] = useState<Competition | null>(null);
  const [matches, setMatches] = useState<Match[]>([]);
  const [teams, setTeams] = useState<Team[]>([]);
  const [standings, setStandings] = useState<Standing[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const competitionData = getCompetition(id);
    const matchesData = getMatchesByCompetition(id);
    const teamsData = getTeams();

    setCompetition(competitionData || null);
    setMatches(matchesData);
    setTeams(teamsData);

    // 順位表を計算
    if (teamsData.length > 0) {
      setStandings(calculateStandings({ teams: teamsData, matches: matchesData }));
    }

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
          <StandingsTable standings={standings} />
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

1. 大会を作成
2. チームを4つ登録
3. 試合を登録してスコアを入力
4. 順位表が自動計算されることを確認

### テストデータ例

| 試合 | スコア |
|------|--------|
| レッドスター vs ブルーウィングス | 3 - 1 |
| グリーンファイターズ vs イエローサンダー | 2 - 2 |
| レッドスター vs グリーンファイターズ | 2 - 0 |
| ブルーウィングス vs イエローサンダー | 1 - 3 |

この結果で順位表が正しく計算されることを確認してください。

## Git でコミット

```bash
git add .
git commit -m "順位表の計算と表示を実装"
```

## この章のまとめ

- 勝点計算ロジックを実装した
- 同率時のソート処理を実装した
- 順位表コンポーネントを作成した

## 確認してみよう

- [ ] 試合結果を入力すると順位表が更新される
- [ ] 勝点が正しく計算される（勝利3点、引分1点）
- [ ] 同じ勝点の場合、得失点差でソートされる
- [ ] 予定の試合は順位計算に含まれない

## チャレンジ課題

このアプリでは、すべての競技をチーム対抗戦として扱っています。しかし、卓球のシングルスのように**個人戦**で使いたいケースもあるでしょう。

以下のような拡張に挑戦してみてください。

- Competition に `format: "team" | "individual"` フィールドを追加する
- 個人戦の場合、「チーム」を「選手」として表示を切り替える
- 卓球のセット制スコア（例: 3-1）に対応する

:::message
正解は1つではありません。自分なりの設計を考えて実装してみましょう。型定義の変更から始めると、修正すべき箇所が TypeScript のエラーとして見えてくるので取り組みやすいです。
:::
