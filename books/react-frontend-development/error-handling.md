---
title: "エラーハンドリングとローディング状態"
---

## この章でやること

- エラーハンドリングの基本パターンを学ぶ
- ローディング状態の管理方法
- エラーとローディングを組み合わせた実践的なパターン
- ユーザーフレンドリーなエラー表示

## エラーハンドリングの基本

### try-catch を使う

`async/await` を使う場合、`try-catch` でエラーをキャッチします。

```tsx
"use client";

import { useState, useEffect } from 'react';

type User = {
  id: string;
  name: string;
};

export default function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function fetchUsers() {
      try {
        const response = await fetch('/api/users');
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        setUsers(data);
        setError(null); // 成功したらエラーをクリア
      } catch (err) {
        setError(err instanceof Error ? err.message : 'エラーが発生しました');
      }
    }

    fetchUsers();
  }, []);

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

### エラーの種類に応じた処理

HTTPステータスコードに応じて、異なるエラーメッセージを表示します。

```tsx
"use client";

import { useState, useEffect } from 'react';

type User = {
  id: string;
  name: string;
};

export default function UserDetail({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [error, setError] = useState<string | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchUser() {
      try {
        const response = await fetch(`/api/users/${userId}`);
        
        if (response.status === 404) {
          throw new Error('ユーザーが見つかりません');
        }
        
        if (response.status === 403) {
          throw new Error('このユーザーにアクセスする権限がありません');
        }
        
        if (!response.ok) {
          throw new Error('データの取得に失敗しました');
        }
        
        const data = await response.json();
        setUser(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'エラーが発生しました');
      } finally {
        setLoading(false);
      }
    }

    fetchUser();
  }, [userId]);

  if (loading) {
    return <p>読み込み中...</p>;
  }

  if (error) {
    return (
      <div>
        <p style={{ color: 'red' }}>エラー: {error}</p>
        <button onClick={() => window.location.reload()}>再読み込み</button>
      </div>
    );
  }

  if (!user) {
    return <p>ユーザー情報がありません</p>;
  }

  return <div>{user.name}</div>;
}
```

## ローディング状態の管理

### 基本的なパターン

```tsx
"use client";

import { useState, useEffect } from 'react';

type Data = {
  id: string;
  name: string;
};

export default function DataList() {
  const [data, setData] = useState<Data[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchData() {
      setLoading(true);
      try {
        const response = await fetch('/api/data');
        const result = await response.json();
        setData(result);
      } catch (error) {
        console.error('エラー:', error);
      } finally {
        setLoading(false);
      }
    }

    fetchData();
  }, []);

  if (loading) {
    return <p>読み込み中...</p>;
  }

  return (
    <ul>
      {data.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

### ローディング中のUI

ローディング中は、スピナーやスケルトンUIを表示します。

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
    return (
      <div>
        <p>読み込み中...</p>
        {/* スケルトンUI */}
        <div style={{ opacity: 0.5 }}>
          <div>名前1</div>
          <div>名前2</div>
          <div>名前3</div>
        </div>
      </div>
    );
  }

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

## エラーとローディングを組み合わせる

### 実践的なパターン

```tsx
"use client";

import { useState, useEffect } from 'react';

type User = {
  id: string;
  name: string;
};

type State = {
  data: User[] | null;
  loading: boolean;
  error: string | null;
};

export default function UserList() {
  const [state, setState] = useState<State>({
    data: null,
    loading: true,
    error: null,
  });

  useEffect(() => {
    async function fetchUsers() {
      setState(prev => ({ ...prev, loading: true, error: null }));

      try {
        const response = await fetch('/api/users');
        
        if (!response.ok) {
          throw new Error('データの取得に失敗しました');
        }
        
        const data = await response.json();
        setState({ data, loading: false, error: null });
      } catch (err) {
        setState({
          data: null,
          loading: false,
          error: err instanceof Error ? err.message : 'エラーが発生しました',
        });
      }
    }

    fetchUsers();
  }, []);

  if (state.loading) {
    return <p>読み込み中...</p>;
  }

  if (state.error) {
    return (
      <div>
        <p style={{ color: 'red' }}>エラー: {state.error}</p>
        <button onClick={() => window.location.reload()}>再読み込み</button>
      </div>
    );
  }

  if (!state.data || state.data.length === 0) {
    return <p>ユーザーがありません</p>;
  }

  return (
    <ul>
      {state.data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## 再試行機能

エラーが発生した場合、再試行できるようにします。

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
  const [error, setError] = useState<string | null>(null);

  const fetchUsers = useCallback(async () => {
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
  }, []);

  useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);

  if (loading) {
    return <p>読み込み中...</p>;
  }

  if (error) {
    return (
      <div>
        <p style={{ color: 'red' }}>エラー: {error}</p>
        <button onClick={fetchUsers}>再試行</button>
      </div>
    );
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

## フォーム送信時のエラーハンドリング

```tsx
"use client";

import { useState } from 'react';

type FormData = {
  name: string;
  email: string;
};

export default function ContactForm() {
  const [formData, setFormData] = useState<FormData>({
    name: '',
    email: '',
  });
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [success, setSuccess] = useState(false);

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setLoading(true);
    setError(null);
    setSuccess(false);

    try {
      const response = await fetch('/api/contact', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData),
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.message || '送信に失敗しました');
      }

      setSuccess(true);
      setFormData({ name: '', email: '' });
    } catch (err) {
      setError(err instanceof Error ? err.message : 'エラーが発生しました');
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
        disabled={loading}
      />
      <input
        type="email"
        value={formData.email}
        onChange={(e) => setFormData(prev => ({ ...prev, email: e.target.value }))}
        placeholder="メールアドレス"
        disabled={loading}
      />
      
      {error && (
        <p style={{ color: 'red' }}>エラー: {error}</p>
      )}
      
      {success && (
        <p style={{ color: 'green' }}>送信しました！</p>
      )}
      
      <button type="submit" disabled={loading}>
        {loading ? '送信中...' : '送信'}
      </button>
    </form>
  );
}
```

## 実践的なエラーハンドリングの例

### データ不整合のエラー

アプリでデータが不整合になった場合の処理例です。

```tsx
"use client";

import { useState, useEffect } from 'react';

type Competition = {
  id: string;
  name: string;
  status: 'upcoming' | 'ongoing' | 'completed';
};

export default function CompetitionList() {
  const [competitions, setCompetitions] = useState<Competition[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function fetchCompetitions() {
      try {
        // localStorageからデータを取得
        const data = localStorage.getItem('competitions');
        
        if (!data) {
          setCompetitions([]);
          return;
        }

        const parsed = JSON.parse(data);
        
        // データの検証
        if (!Array.isArray(parsed)) {
          throw new Error('データの形式が正しくありません');
        }

        // 各項目の検証
        const validated = parsed.filter((comp: any) => {
          return comp && 
                 typeof comp.id === 'string' &&
                 typeof comp.name === 'string' &&
                 ['upcoming', 'ongoing', 'completed'].includes(comp.status);
        });

        if (validated.length !== parsed.length) {
          console.warn('一部のデータが無効でした');
        }

        setCompetitions(validated);
      } catch (err) {
        if (err instanceof SyntaxError) {
          setError('データの読み込みに失敗しました。データが壊れている可能性があります。');
          // 壊れたデータを削除
          localStorage.removeItem('competitions');
        } else {
          setError(err instanceof Error ? err.message : 'エラーが発生しました');
        }
      } finally {
        setLoading(false);
      }
    }

    fetchCompetitions();
  }, []);

  if (loading) {
    return <p>読み込み中...</p>;
  }

  if (error) {
    return (
      <div>
        <p style={{ color: 'red' }}>エラー: {error}</p>
        <button onClick={() => window.location.reload()}>
          ページを再読み込み
        </button>
      </div>
    );
  }

  return (
    <ul>
      {competitions.map(comp => (
        <li key={comp.id}>{comp.name} ({comp.status})</li>
      ))}
    </ul>
  );
}
```

### ネットワークエラーの処理

ネットワークが不安定な場合の処理例です。

```tsx
"use client";

import { useState } from 'react';

export default function DataForm() {
  const [name, setName] = useState('');
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    setError(null);

    try {
      const response = await fetch('/api/data', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name }),
      });

      // ネットワークエラーのチェック
      if (!response.ok) {
        if (response.status === 0) {
          throw new Error('ネットワークエラー: サーバーに接続できません');
        }
        throw new Error(`エラー: ${response.status}`);
      }

      const data = await response.json();
      console.log('成功:', data);
      setName('');
    } catch (err) {
      if (err instanceof TypeError && err.message.includes('fetch')) {
        setError('ネットワークエラー: インターネット接続を確認してください');
      } else {
        setError(err instanceof Error ? err.message : 'エラーが発生しました');
      }
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        disabled={loading}
      />
      {error && (
        <p style={{ color: 'red' }}>{error}</p>
      )}
      <button type="submit" disabled={loading}>
        {loading ? '送信中...' : '送信'}
      </button>
    </form>
  );
}
```

### タイムアウトの処理

リクエストが長時間かかる場合のタイムアウト処理です。

```tsx
"use client";

import { useState, useEffect } from 'react';

export default function SlowDataLoader() {
  const [data, setData] = useState<string | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), 10000); // 10秒でタイムアウト

    async function fetchData() {
      try {
        const response = await fetch('/api/slow-data', {
          signal: controller.signal,
        });

        if (!response.ok) {
          throw new Error('データの取得に失敗しました');
        }

        const result = await response.json();
        setData(result);
      } catch (err) {
        if (err instanceof Error && err.name === 'AbortError') {
          setError('タイムアウト: リクエストが時間内に完了しませんでした');
        } else {
          setError(err instanceof Error ? err.message : 'エラーが発生しました');
        }
      } finally {
        clearTimeout(timeoutId);
        setLoading(false);
      }
    }

    fetchData();

    return () => {
      controller.abort();
      clearTimeout(timeoutId);
    };
  }, []);

  if (loading) {
    return <p>読み込み中...（最大10秒）</p>;
  }

  if (error) {
    return (
      <div>
        <p style={{ color: 'red' }}>エラー: {error}</p>
        <button onClick={() => window.location.reload()}>
          再試行
        </button>
      </div>
    );
  }

  return <div>{data}</div>;
}
```

## よくあるパターン

### パターン1: エラーコンポーネント

エラー表示をコンポーネント化します。

```tsx
// components/ErrorDisplay.tsx
type ErrorDisplayProps = {
  error: string;
  onRetry?: () => void;
};

export default function ErrorDisplay({ error, onRetry }: ErrorDisplayProps) {
  return (
    <div style={{ padding: '20px', border: '1px solid red', borderRadius: '4px' }}>
      <p style={{ color: 'red', margin: 0 }}>エラー: {error}</p>
      {onRetry && (
        <button onClick={onRetry} style={{ marginTop: '10px' }}>
          再試行
        </button>
      )}
    </div>
  );
}

// 使う側
"use client";

import { useState, useEffect } from 'react';
import ErrorDisplay from '@/components/ErrorDisplay';

export default function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchUsers = async () => {
    setLoading(true);
    setError(null);
    try {
      const response = await fetch('/api/users');
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

  if (loading) return <p>読み込み中...</p>;
  if (error) return <ErrorDisplay error={error} onRetry={fetchUsers} />;

  return (
    <ul>
      {users.map((user: any) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### パターン2: ローディングコンポーネント

ローディング表示もコンポーネント化します。

```tsx
// components/LoadingSpinner.tsx
export default function LoadingSpinner() {
  return (
    <div style={{ textAlign: 'center', padding: '20px' }}>
      <p>読み込み中...</p>
    </div>
  );
}
```

## この章のまとめ

- `try-catch` でエラーをキャッチする
- HTTPステータスコードに応じて、適切なエラーメッセージを表示する
- ローディング状態を管理して、ユーザーに進行状況を伝える
- エラーとローディングを組み合わせて、適切なUIを表示する
- 再試行機能を実装して、ユーザビリティを向上させる
- エラー表示やローディング表示をコンポーネント化して再利用する
- データ不整合、ネットワークエラー、タイムアウトなど、実践的なエラーケースに対応できる

## 確認してみよう

- [ ] `try-catch` でエラーハンドリングができる
- [ ] HTTPステータスコードに応じた処理ができる
- [ ] ローディング状態を管理できる
- [ ] エラーとローディングを組み合わせたUIが作れる
- [ ] 再試行機能が実装できる
- [ ] データ不整合のエラーに対応できる
- [ ] ネットワークエラーやタイムアウトを処理できる

:::message
エラーハンドリングとローディング状態の管理は、ユーザー体験に大きく影響します。適切に実装することで、使いやすいアプリになります。
:::
