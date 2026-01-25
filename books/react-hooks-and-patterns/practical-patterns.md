---
title: "実践的なパターン集"
---

## この章でやること

- 実践的な開発でよく使うパターンを学ぶ
- 複数の概念を組み合わせた実装
- ベストプラクティス
- よくある問題とその解決方法

## パターン1: データ取得とCRUD操作

データの一覧表示、作成、更新、削除を実装するパターンです。

```tsx
"use client";

import { useState, useEffect, useCallback } from 'react';

type Item = {
  id: string;
  name: string;
  description: string;
};

export default function ItemList() {
  const [items, setItems] = useState<Item[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  // データ取得
  const fetchItems = useCallback(async () => {
    setLoading(true);
    setError(null);
    try {
      const response = await fetch('/api/items');
      if (!response.ok) {
        throw new Error('データの取得に失敗しました');
      }
      const data = await response.json();
      setItems(data);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'エラーが発生しました');
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchItems();
  }, [fetchItems]);

  // 作成
  const handleCreate = async (name: string, description: string) => {
    try {
      const response = await fetch('/api/items', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name, description }),
      });
      if (!response.ok) {
        throw new Error('作成に失敗しました');
      }
      fetchItems(); // 再取得
    } catch (err) {
      console.error('エラー:', err);
    }
  };

  // 更新
  const handleUpdate = async (id: string, name: string, description: string) => {
    try {
      const response = await fetch(`/api/items/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name, description }),
      });
      if (!response.ok) {
        throw new Error('更新に失敗しました');
      }
      fetchItems(); // 再取得
    } catch (err) {
      console.error('エラー:', err);
    }
  };

  // 削除
  const handleDelete = async (id: string) => {
    if (!confirm('削除しますか？')) return;
    
    try {
      const response = await fetch(`/api/items/${id}`, {
        method: 'DELETE',
      });
      if (!response.ok) {
        throw new Error('削除に失敗しました');
      }
      fetchItems(); // 再取得
    } catch (err) {
      console.error('エラー:', err);
    }
  };

  if (loading) return <p>読み込み中...</p>;
  if (error) return <p style={{ color: 'red' }}>エラー: {error}</p>;

  return (
    <div>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.name} - {item.description}
            <button onClick={() => handleDelete(item.id)}>削除</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## パターン2: モーダルとフォームの組み合わせ

モーダルでフォームを表示して、データを作成・編集するパターンです。

```tsx
"use client";

import { useState } from 'react';

type Item = {
  id: string;
  name: string;
};

type ModalProps = {
  isOpen: boolean;
  onClose: () => void;
  onSubmit: (name: string) => void;
  initialValue?: string;
};

function ItemModal({ isOpen, onClose, onSubmit, initialValue = '' }: ModalProps) {
  const [name, setName] = useState(initialValue);

  if (!isOpen) return null;

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    onSubmit(name);
    setName('');
    onClose();
  };

  return (
    <div style={{
      position: 'fixed',
      top: 0,
      left: 0,
      right: 0,
      bottom: 0,
      backgroundColor: 'rgba(0, 0, 0, 0.5)',
      display: 'flex',
      alignItems: 'center',
      justifyContent: 'center',
    }}>
      <div style={{
        backgroundColor: 'white',
        padding: '20px',
        borderRadius: '8px',
        width: '400px',
      }}>
        <h2>アイテム{initialValue ? '編集' : '作成'}</h2>
        <form onSubmit={handleSubmit}>
          <input
            type="text"
            value={name}
            onChange={(e) => setName(e.target.value)}
            placeholder="名前"
            required
          />
          <div style={{ marginTop: '10px' }}>
            <button type="submit">保存</button>
            <button type="button" onClick={onClose}>キャンセル</button>
          </div>
        </form>
      </div>
    </div>
  );
}

export default function ItemManagement() {
  const [items, setItems] = useState<Item[]>([]);
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [editingItem, setEditingItem] = useState<Item | null>(null);

  const handleCreate = (name: string) => {
    // 作成処理
    setItems(prev => [...prev, { id: Date.now().toString(), name }]);
  };

  const handleEdit = (item: Item) => {
    setEditingItem(item);
    setIsModalOpen(true);
  };

  const handleUpdate = (name: string) => {
    if (!editingItem) return;
    setItems(prev =>
      prev.map(item =>
        item.id === editingItem.id ? { ...item, name } : item
      )
    );
    setEditingItem(null);
  };

  return (
    <div>
      <button onClick={() => setIsModalOpen(true)}>新規作成</button>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.name}
            <button onClick={() => handleEdit(item)}>編集</button>
          </li>
        ))}
      </ul>
      <ItemModal
        isOpen={isModalOpen || editingItem !== null}
        onClose={() => {
          setIsModalOpen(false);
          setEditingItem(null);
        }}
        onSubmit={editingItem ? handleUpdate : handleCreate}
        initialValue={editingItem?.name}
      />
    </div>
  );
}
```

## パターン3: 検索とフィルタリング

検索機能とフィルタリング機能を実装するパターンです。

```tsx
"use client";

import { useState, useMemo } from 'react';

type User = {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
};

export default function UserList({ users }: { users: User[] }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [roleFilter, setRoleFilter] = useState<'all' | 'admin' | 'user'>('all');

  // フィルタリングされたユーザーリスト
  const filteredUsers = useMemo(() => {
    return users.filter(user => {
      const matchesSearch = user.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
                           user.email.toLowerCase().includes(searchTerm.toLowerCase());
      const matchesRole = roleFilter === 'all' || user.role === roleFilter;
      return matchesSearch && matchesRole;
    });
  }, [users, searchTerm, roleFilter]);

  return (
    <div>
      <div>
        <input
          type="text"
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          placeholder="検索..."
        />
        <select
          value={roleFilter}
          onChange={(e) => setRoleFilter(e.target.value as 'all' | 'admin' | 'user')}
        >
          <option value="all">すべて</option>
          <option value="admin">管理者</option>
          <option value="user">一般ユーザー</option>
        </select>
      </div>
      <ul>
        {filteredUsers.map(user => (
          <li key={user.id}>
            {user.name} ({user.email}) - {user.role}
          </li>
        ))}
      </ul>
      {filteredUsers.length === 0 && (
        <p>該当するユーザーがありません</p>
      )}
    </div>
  );
}
```

## パターン4: ページネーション

大量のデータをページに分けて表示するパターンです。

```tsx
"use client";

import { useState, useMemo } from 'react';

type Item = {
  id: string;
  name: string;
};

export default function PaginatedList({ items }: { items: Item[] }) {
  const [currentPage, setCurrentPage] = useState(1);
  const itemsPerPage = 10;

  // 現在のページのアイテム
  const currentItems = useMemo(() => {
    const startIndex = (currentPage - 1) * itemsPerPage;
    const endIndex = startIndex + itemsPerPage;
    return items.slice(startIndex, endIndex);
  }, [items, currentPage]);

  // 総ページ数
  const totalPages = Math.ceil(items.length / itemsPerPage);

  return (
    <div>
      <ul>
        {currentItems.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
      <div>
        <button
          onClick={() => setCurrentPage(prev => Math.max(1, prev - 1))}
          disabled={currentPage === 1}
        >
          前へ
        </button>
        <span>
          ページ {currentPage} / {totalPages}
        </span>
        <button
          onClick={() => setCurrentPage(prev => Math.min(totalPages, prev + 1))}
          disabled={currentPage === totalPages}
        >
          次へ
        </button>
      </div>
    </div>
  );
}
```

## パターン5: オプティミスティックアップデート

サーバーの応答を待たずに、UIを先に更新するパターンです。

```tsx
"use client";

import { useState } from 'react';

type Todo = {
  id: string;
  text: string;
  completed: boolean;
};

export default function TodoList({ initialTodos }: { initialTodos: Todo[] }) {
  const [todos, setTodos] = useState<Todo[]>(initialTodos);

  const toggleTodo = async (id: string) => {
    const todo = todos.find(t => t.id === id);
    if (!todo) return;

    // オプティミスティックアップデート: 先にUIを更新
    setTodos(prev =>
      prev.map(t =>
        t.id === id ? { ...t, completed: !t.completed } : t
      )
    );

    try {
      // サーバーに送信
      const response = await fetch(`/api/todos/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ completed: !todo.completed }),
      });

      if (!response.ok) {
        // エラーが発生したら元に戻す
        setTodos(prev =>
          prev.map(t =>
            t.id === id ? { ...t, completed: todo.completed } : t
          )
        );
        throw new Error('更新に失敗しました');
      }
    } catch (error) {
      console.error('エラー:', error);
      // エラーメッセージを表示
    }
  };

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => toggleTodo(todo.id)}
          />
          {todo.text}
        </li>
      ))}
    </ul>
  );
}
```

## よくある問題と解決方法

### 問題1: 無限ループ

依存配列を正しく指定しないと、無限ループが発生します。

```tsx
// ❌ 悪い例: 依存配列がない
useEffect(() => {
  setCount(count + 1); // 毎回実行される → 無限ループ
});

// ✅ 良い例: 依存配列を指定
useEffect(() => {
  // 必要な処理
}, []); // 最初の1回だけ実行
```

### 問題2: 古いstateを参照する

`setState` の関数形式を使うと、最新のstateを参照できます。

```tsx
// ❌ 悪い例: 古いstateを参照する可能性がある
const handleClick = () => {
  setCount(count + 1);
  setCount(count + 1); // count は古い値のまま
};

// ✅ 良い例: 関数形式で最新のstateを参照
const handleClick = () => {
  setCount(prev => prev + 1);
  setCount(prev => prev + 1); // 最新の値を使う
};
```

### 問題3: メモリリーク

クリーンアップ関数を忘れると、メモリリークが発生します。

```tsx
// ❌ 悪い例: クリーンアップがない
useEffect(() => {
  const interval = setInterval(() => {
    // 処理
  }, 1000);
  // クリーンアップがない → メモリリーク
}, []);

// ✅ 良い例: クリーンアップ関数を返す
useEffect(() => {
  const interval = setInterval(() => {
    // 処理
  }, 1000);
  return () => clearInterval(interval); // クリーンアップ
}, []);
```

## この章のまとめ

- CRUD操作のパターンを覚えると、様々な機能に応用できる
- モーダルとフォームを組み合わせると、UXが向上する
- 検索とフィルタリングは、`useMemo` でパフォーマンスを最適化できる
- ページネーションで、大量のデータを効率的に表示できる
- オプティミスティックアップデートで、レスポンシブなUIを実現できる
- よくある問題を理解して、適切に対処する

## 確認してみよう

- [ ] CRUD操作のパターンが実装できる
- [ ] モーダルとフォームを組み合わせられる
- [ ] 検索とフィルタリングが実装できる
- [ ] ページネーションが実装できる
- [ ] オプティミスティックアップデートが理解できる

:::message
これらのパターンは、実践的なアプリ開発で頻繁に使います。パターンを覚えておくと、開発がスムーズになります。
:::
