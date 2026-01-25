---
title: "カスタムフックの基礎"
---

## この章でやること

- カスタムフックとは何かを理解する
- カスタムフックの作り方を学ぶ
- ロジックを再利用する方法
- よく使うカスタムフックのパターン

## カスタムフックとは

カスタムフックは、**ロジックを再利用するための関数**です。

複数のコンポーネントで同じロジックを使う場合、カスタムフックにまとめることで、コードの重複を減らし、保守性を高められます。

### カスタムフックのルール

1. **`use` で始まる名前をつける**
   - 例: `useUsers`, `useForm`, `useLocalStorage`

2. **React のフック（useState、useEffect など）を使える**
   - 通常の関数では使えないが、カスタムフック内では使える

3. **コンポーネントのトップレベルで呼び出す**
   - if文やループの中で呼び出さない

## 基本的なカスタムフック

### 例1: localStorage を扱うフック

```tsx
// hooks/useLocalStorage.ts
import { useState, useEffect } from 'react';

export function useLocalStorage<T>(key: string, initialValue: T) {
  // localStorage から値を読み込む
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') {
      return initialValue;
    }
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  // localStorage に値を保存する関数
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue] as const;
}

// 使う側
"use client";

import { useLocalStorage } from '@/hooks/useLocalStorage';

export default function TodoApp() {
  const [todos, setTodos] = useLocalStorage<string[]>('todos', []);

  const addTodo = (text: string) => {
    setTodos(prev => [...prev, text]);
  };

  return (
    <div>
      {/* ... */}
    </div>
  );
}
```

### 例2: データ取得のフック

```tsx
// hooks/useUsers.ts
import { useState, useEffect } from 'react';

type User = {
  id: string;
  name: string;
  email: string;
};

type UseUsersReturn = {
  users: User[];
  loading: boolean;
  error: string | null;
  refetch: () => void;
};

export function useUsers(): UseUsersReturn {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchUsers = async () => {
    setLoading(true);
    setError(null);
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
  };

  useEffect(() => {
    fetchUsers();
  }, []);

  return { users, loading, error, refetch: fetchUsers };
}

// 使う側
"use client";

import { useUsers } from '@/hooks/useUsers';

export default function UserList() {
  const { users, loading, error, refetch } = useUsers();

  if (loading) return <p>読み込み中...</p>;
  if (error) return <p style={{ color: 'red' }}>エラー: {error}</p>;

  return (
    <div>
      <button onClick={refetch}>再読み込み</button>
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

## よく使うカスタムフックのパターン

### パターン1: フォーム管理フック

```tsx
// hooks/useForm.ts
import { useState } from 'react';

type UseFormReturn<T> = {
  formData: T;
  handleChange: (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => void;
  reset: () => void;
  setFormData: React.Dispatch<React.SetStateAction<T>>;
};

export function useForm<T extends Record<string, any>>(
  initialValues: T
): UseFormReturn<T> {
  const [formData, setFormData] = useState<T>(initialValues);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value,
    }));
  };

  const reset = () => {
    setFormData(initialValues);
  };

  return { formData, handleChange, reset, setFormData };
}

// 使う側
"use client";

import { useForm } from '@/hooks/useForm';

export default function ContactForm() {
  const { formData, handleChange, reset } = useForm({
    name: '',
    email: '',
    message: '',
  });

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    console.log(formData);
    reset();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        name="name"
        value={formData.name}
        onChange={handleChange}
      />
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
      />
      <textarea
        name="message"
        value={formData.message}
        onChange={handleChange}
      />
      <button type="submit">送信</button>
    </form>
  );
}
```

### パターン2: ウィンドウサイズの取得

```tsx
// hooks/useWindowSize.ts
import { useState, useEffect } from 'react';

type WindowSize = {
  width: number;
  height: number;
};

export function useWindowSize(): WindowSize {
  const [windowSize, setWindowSize] = useState<WindowSize>({
    width: 0,
    height: 0,
  });

  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    window.addEventListener('resize', handleResize);
    handleResize(); // 初期値を設定

    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return windowSize;
}

// 使う側
"use client";

import { useWindowSize } from '@/hooks/useWindowSize';

export default function ResponsiveComponent() {
  const { width } = useWindowSize();

  return (
    <div>
      {width < 768 ? (
        <p>モバイル表示</p>
      ) : (
        <p>デスクトップ表示</p>
      )}
    </div>
  );
}
```

### パターン3: デバウンスフック

検索入力などで、入力が止まってから処理を実行するフックです。

```tsx
// hooks/useDebounce.ts
import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);

  return debouncedValue;
}

// 使う側
"use client";

import { useState, useEffect } from 'react';
import { useDebounce } from '@/hooks/useDebounce';

export default function SearchBox() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  const [results, setResults] = useState<string[]>([]);

  useEffect(() => {
    if (!debouncedSearchTerm) {
      setResults([]);
      return;
    }

    async function search() {
      const response = await fetch(`/api/search?q=${debouncedSearchTerm}`);
      const data = await response.json();
      setResults(data);
    }

    search();
  }, [debouncedSearchTerm]);

  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="検索"
      />
      <ul>
        {results.map((result, index) => (
          <li key={index}>{result}</li>
        ))}
      </ul>
    </div>
  );
}
```

## カスタムフックのベストプラクティス

### 1. 単一責任の原則

1つのカスタムフックは、1つの責任だけを持たせます。

```tsx
// ✅ 良い例: データ取得だけに集中
export function useUsers() {
  // ユーザー取得のロジック
}

// ❌ 悪い例: データ取得とフォーム処理が混在
export function useUsersAndForm() {
  // ユーザー取得とフォーム処理が混在
}
```

### 2. 名前を明確にする

カスタムフックの名前から、何をするかがわかるようにします。

```tsx
// ✅ 良い例
useUsers()
useLocalStorage()
useDebounce()

// ❌ 悪い例
useData()
useHook()
useStuff()
```

### 3. 型を定義する

TypeScript を使う場合、戻り値の型を明確に定義します。

```tsx
type UseUsersReturn = {
  users: User[];
  loading: boolean;
  error: string | null;
};

export function useUsers(): UseUsersReturn {
  // ...
}
```

## この章のまとめ

- カスタムフックは、ロジックを再利用するための関数
- `use` で始まる名前をつける
- React のフック（useState、useEffect など）を使える
- 単一責任の原則に従って、1つのフックは1つの責任だけを持つ
- 型を定義して、使いやすくする

## 確認してみよう

- [ ] カスタムフックが何かを説明できる
- [ ] カスタムフックを作成できる
- [ ] カスタムフックを使ってロジックを再利用できる
- [ ] localStorage を扱うカスタムフックが作れる
- [ ] データ取得のカスタムフックが作れる

:::message
カスタムフックを使うと、コードの重複を減らし、保守性を高められます。実践的なアプリ開発では、よく使うパターンをカスタムフックにまとめることが多いです。
:::
