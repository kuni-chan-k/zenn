---
title: "データフェッチング（API呼び出し）"
---

## この章でやること

- fetch API の使い方を学ぶ
- async/await の使い方
- useEffect と組み合わせたデータ取得
- ローディング状態の管理
- エラーハンドリング

## fetch API の基礎

### 基本的な使い方

`fetch` は、**HTTPリクエストを送信してデータを取得する**ためのAPIです。

```tsx
// GETリクエスト
fetch('/api/users')
  .then(response => response.json())
  .then(data => {
    console.log(data);
  });
```

### async/await を使う

`async/await` を使うと、コードが読みやすくなります。

```tsx
async function fetchUsers() {
  const response = await fetch('/api/users');
  const data = await response.json();
  console.log(data);
}
```

## useEffect と組み合わせる

### 基本的なパターン

コンポーネントが表示されたときにデータを取得します。

```tsx
"use client";

import { useState, useEffect } from 'react';

type User = {
  id: string;
  name: string;
  email: string;
};

export default function UserList() {
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    async function fetchUsers() {
      const response = await fetch('/api/users');
      const data = await response.json();
      setUsers(data);
    }

    fetchUsers();
  }, []);

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name} ({user.email})
        </li>
      ))}
    </ul>
  );
}
```

### ローディング状態の管理

データ取得中は、ローディング状態を表示します。

```tsx
"use client";

import { useState, useEffect } from 'react';

type User = {
  id: string;
  name: string;
};

export default function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchUsers() {
      const response = await fetch('/api/users');
      const data = await response.json();
      setUsers(data);
      setLoading(false);
    }

    fetchUsers();
  }, []);

  if (loading) {
    return <p>読み込み中...</p>;
  }

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### エラーハンドリング

エラーが発生した場合の処理を追加します。

```tsx
"use client";

import { useState, useEffect } from 'react';

type User = {
  id: string;
  name: string;
};

export default function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function fetchUsers() {
      try {
        const response = await fetch('/api/users');
        
        if (!response.ok) {
          throw new Error('データの取得に失敗しました');
        }
        
        const data = await response.json();
        setUsers(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'エラーが発生しました');
      } finally {
        setLoading(false);
      }
    }

    fetchUsers();
  }, []);

  if (loading) {
    return <p>読み込み中...</p>;
  }

  if (error) {
    return <p style={{ color: 'red' }}>エラー: {error}</p>;
  }

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## POSTリクエスト（データの送信）

### 基本的な使い方

```tsx
"use client";

import { useState } from 'react';

type FormData = {
  name: string;
  email: string;
};

export default function CreateUserForm() {
  const [formData, setFormData] = useState<FormData>({
    name: '',
    email: '',
  });
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setLoading(true);

    try {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(formData),
      });

      if (!response.ok) {
        throw new Error('ユーザーの作成に失敗しました');
      }

      const data = await response.json();
      console.log('作成されたユーザー:', data);
      
      // フォームをリセット
      setFormData({ name: '', email: '' });
    } catch (error) {
      console.error('エラー:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={formData.name}
        onChange={(e) => setFormData(prev => ({ ...prev, name: e.target.value }))}
        placeholder="名前"
      />
      <input
        type="email"
        value={formData.email}
        onChange={(e) => setFormData(prev => ({ ...prev, email: e.target.value }))}
        placeholder="メールアドレス"
      />
      <button type="submit" disabled={loading}>
        {loading ? '送信中...' : '作成'}
      </button>
    </form>
  );
}
```

## パラメータ付きのリクエスト

### URLパラメータ

```tsx
"use client";

import { useState, useEffect } from 'react';

type User = {
  id: string;
  name: string;
};

export default function UserDetail({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchUser() {
      try {
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        setUser(data);
      } catch (error) {
        console.error('エラー:', error);
      } finally {
        setLoading(false);
      }
    }

    fetchUser();
  }, [userId]); // userId が変わったときに再取得

  if (loading) return <p>読み込み中...</p>;
  if (!user) return <p>ユーザーが見つかりません</p>;

  return <div>{user.name}</div>;
}
```

### クエリパラメータ

```tsx
"use client";

import { useState, useEffect } from 'react';

type User = {
  id: string;
  name: string;
};

export default function UserSearch() {
  const [searchTerm, setSearchTerm] = useState('');
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (!searchTerm) {
      setUsers([]);
      return;
    }

    async function searchUsers() {
      setLoading(true);
      try {
        const response = await fetch(`/api/users?q=${encodeURIComponent(searchTerm)}`);
        const data = await response.json();
        setUsers(data);
      } catch (error) {
        console.error('エラー:', error);
      } finally {
        setLoading(false);
      }
    }

    // デバウンス（500ms待ってから検索）
    const timer = setTimeout(searchUsers, 500);
    return () => clearTimeout(timer);
  }, [searchTerm]);

  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="ユーザーを検索"
      />
      {loading && <p>検索中...</p>}
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

## よくあるパターン

### パターン1: データ取得のカスタムフック

データ取得のロジックを再利用可能なフックにします。

```tsx
// hooks/useUsers.ts
import { useState, useEffect } from 'react';

type User = {
  id: string;
  name: string;
};

export function useUsers() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function fetchUsers() {
      try {
        const response = await fetch('/api/users');
        if (!response.ok) {
          throw new Error('データの取得に失敗しました');
        }
        const data = await response.json();
        setUsers(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'エラーが発生しました');
      } finally {
        setLoading(false);
      }
    }

    fetchUsers();
  }, []);

  return { users, loading, error };
}

// コンポーネントで使う
"use client";

import { useUsers } from '@/hooks/useUsers';

export default function UserList() {
  const { users, loading, error } = useUsers();

  if (loading) return <p>読み込み中...</p>;
  if (error) return <p style={{ color: 'red' }}>エラー: {error}</p>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### パターン2: 再取得関数

データを更新した後、再取得できるようにします。

```tsx
"use client";

import { useState, useEffect, useCallback } from 'react';

type User = {
  id: string;
  name: string;
};

export default function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);

  const fetchUsers = useCallback(async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/users');
      const data = await response.json();
      setUsers(data);
    } catch (error) {
      console.error('エラー:', error);
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);

  const handleDelete = async (id: string) => {
    await fetch(`/api/users/${id}`, { method: 'DELETE' });
    fetchUsers(); // 削除後に再取得
  };

  if (loading) return <p>読み込み中...</p>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name}
          <button onClick={() => handleDelete(user.id)}>削除</button>
        </li>
      ))}
    </ul>
  );
}
```

## この章のまとめ

- `fetch` API でHTTPリクエストを送信できる
- `async/await` で非同期処理を読みやすく書ける
- `useEffect` と組み合わせて、コンポーネント表示時にデータを取得できる
- ローディング状態とエラーハンドリングを適切に管理する
- POSTリクエストでデータを送信できる
- カスタムフックにすることで、データ取得ロジックを再利用できる

## 確認してみよう

- [ ] `fetch` API が使える
- [ ] `async/await` が使える
- [ ] `useEffect` と組み合わせてデータを取得できる
- [ ] ローディング状態を管理できる
- [ ] エラーハンドリングができる
- [ ] POSTリクエストでデータを送信できる

:::message
データフェッチングは実践的なアプリ開発の核心部分です。パターンを覚えておくと、様々な場面で応用できます。
:::
