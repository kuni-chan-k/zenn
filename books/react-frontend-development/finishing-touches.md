---
title: "仕上げとUX改善"
---

## この章でやること

- アプリ全体のUX改善
- フィルタ機能の追加
- 確認ダイアログの実装
- 次のステップの紹介

## フィルタ機能の追加

大会一覧にフィルタ機能を追加して、大会を探しやすくしましょう。

### 大会ステータスでフィルタ

```tsx
'use client';

import { useState } from 'react';
import { Competition } from '@/types';

type StatusFilter = 'all' | 'upcoming' | 'ongoing' | 'completed';

export default function CompetitionList({ competitions }: { competitions: Competition[] }) {
  const [statusFilter, setStatusFilter] = useState<StatusFilter>('all');

  const filteredCompetitions = competitions.filter((competition) => {
    if (statusFilter === 'all') return true;
    return competition.status === statusFilter;
  });

  return (
    <div>
      <div className="mb-4 flex gap-2">
        <button
          onClick={() => setStatusFilter('all')}
          className={statusFilter === 'all' ? 'bg-blue-500 text-white' : 'bg-gray-200'}
        >
          すべて
        </button>
        <button
          onClick={() => setStatusFilter('upcoming')}
          className={statusFilter === 'upcoming' ? 'bg-blue-500 text-white' : 'bg-gray-200'}
        >
          開催予定
        </button>
        <button
          onClick={() => setStatusFilter('ongoing')}
          className={statusFilter === 'ongoing' ? 'bg-blue-500 text-white' : 'bg-gray-200'}
        >
          開催中
        </button>
        <button
          onClick={() => setStatusFilter('completed')}
          className={statusFilter === 'completed' ? 'bg-blue-500 text-white' : 'bg-gray-200'}
        >
          終了
        </button>
      </div>

      {filteredCompetitions.map((competition) => (
        <div key={competition.id}>{/* 大会カード */}</div>
      ))}
    </div>
  );
}
```

## 確認ダイアログ

削除などの重要な操作には確認ダイアログを表示しましょう。

```tsx
function ConfirmDialog({
  isOpen,
  title,
  message,
  onConfirm,
  onCancel,
}: {
  isOpen: boolean;
  title: string;
  message: string;
  onConfirm: () => void;
  onCancel: () => void;
}) {
  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 bg-black/50 flex items-center justify-center">
      <div className="bg-white p-6 rounded-lg max-w-md">
        <h2 className="text-lg font-bold mb-2">{title}</h2>
        <p className="mb-4">{message}</p>
        <div className="flex justify-end gap-2">
          <button onClick={onCancel} className="px-4 py-2 bg-gray-200 rounded">
            キャンセル
          </button>
          <button onClick={onConfirm} className="px-4 py-2 bg-red-500 text-white rounded">
            削除
          </button>
        </div>
      </div>
    </div>
  );
}
```

### 使用例

```tsx
const [showDeleteDialog, setShowDeleteDialog] = useState(false);
const [deleteTargetId, setDeleteTargetId] = useState<string | null>(null);

const handleDeleteClick = (id: string) => {
  setDeleteTargetId(id);
  setShowDeleteDialog(true);
};

const handleConfirmDelete = () => {
  if (deleteTargetId) {
    deleteCompetition(deleteTargetId);
  }
  setShowDeleteDialog(false);
  setDeleteTargetId(null);
};

return (
  <>
    {/* 削除ボタン */}
    <button onClick={() => handleDeleteClick(competition.id)}>削除</button>

    {/* 確認ダイアログ */}
    <ConfirmDialog
      isOpen={showDeleteDialog}
      title="大会を削除"
      message="この大会を削除しますか？この操作は取り消せません。"
      onConfirm={handleConfirmDelete}
      onCancel={() => setShowDeleteDialog(false)}
    />
  </>
);
```

## ローディング表示

データを読み込んでいる間はローディング表示をしましょう。

```tsx
function LoadingSpinner() {
  return (
    <div className="flex justify-center items-center p-4">
      <div className="animate-spin h-8 w-8 border-4 border-blue-500 rounded-full border-t-transparent"></div>
    </div>
  );
}
```

## 空の状態

データがない場合は、わかりやすいメッセージを表示しましょう。

```tsx
function EmptyState({ message, action }: { message: string; action?: React.ReactNode }) {
  return (
    <div className="text-center py-12">
      <p className="text-gray-500 mb-4">{message}</p>
      {action}
    </div>
  );
}

// 使用例
{competitions.length === 0 ? (
  <EmptyState
    message="まだ大会がありません"
    action={<Link href="/competitions/new">大会を作成</Link>}
  />
) : (
  // 大会一覧
)}
```

## この章のまとめ

- フィルタ機能で大会を探しやすくした
- 確認ダイアログで誤操作を防止
- ローディング表示とエラー表示でUX向上
- 空の状態を適切に表示

## おめでとうございます！

これで「Reactで作る！フロントエンド完結アプリ開発」は完了です！

**フロントエンドだけで動く完全なアプリが完成しました！**

あなたは以下のことができるようになりました。

- ✅ 開発環境を整える（Node.js, VSCode, Git）
- ✅ React の基本概念を理解する（コンポーネント、props、state）
- ✅ useEffect やカスタムフックを使える
- ✅ TypeScript でデータモデルを定義する
- ✅ Tailwind CSS でスタイリングする
- ✅ CRUD操作（作成・読み取り・更新・削除）を実装する
- ✅ フォーム処理とバリデーション
- ✅ 計算ロジック（順位表）を実装する
- ✅ localStorageでデータを永続化する

**この本で、動くアプリが完成しました！**

データベースやサーバーは不要で、ブラウザだけで完全に動作するアプリです。すべての機能が動作し、データはlocalStorageに保存されます。

### 次のステップ（任意）

さらに発展させたい場合は、**「Next.jsでバックエンド連携＆サービス公開」**に進みましょう！

「Next.jsでバックエンド連携＆サービス公開」では：
- バックエンドの概念（API、データベース、認証）を学ぶ
- Prisma + PostgreSQLでデータベース連携
- JWT認証でログイン機能を実装
- Vercelにデプロイして公開

することで、本格的なWebサービスとして公開できるようになります。

:::message
「Next.jsでバックエンド連携＆サービス公開」は**任意**です。この本だけで完成したアプリが作れます。さらに発展させたい場合のみ、「Next.jsでバックエンド連携＆サービス公開」に進んでください。
:::
