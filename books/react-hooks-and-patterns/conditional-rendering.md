---
title: "条件付きレンダリング"
---

## この章でやること

- 条件に応じて画面を表示・非表示する方法を学ぶ
- if文を使った条件分岐
- 三項演算子（`? :`）の使い方
- 論理AND演算子（`&&`）の使い方
- 実践的なパターンを学ぶ

## 条件付きレンダリングとは

条件付きレンダリングは、**条件に応じて異なる内容を表示する**ことです。

例えば：
- ログインしているときは「ログアウト」ボタンを表示
- ログインしていないときは「ログイン」ボタンを表示
- データを読み込み中のときは「読み込み中...」を表示
- エラーが発生したときはエラーメッセージを表示

## if文を使った条件分岐

### 基本的な使い方

JSXの中で直接if文は使えませんが、コンポーネント内でif文を使って条件分岐できます。

```tsx
"use client";

import { useState } from 'react';

export default function LoginButton() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);

  if (isLoggedIn) {
    return (
      <div>
        <p>ようこそ！</p>
        <button onClick={() => setIsLoggedIn(false)}>ログアウト</button>
      </div>
    );
  }

  return (
    <div>
      <button onClick={() => setIsLoggedIn(true)}>ログイン</button>
    </div>
  );
}
```

### 変数にJSXを代入する

条件に応じて異なるJSXを変数に代入して、最後に返す方法もあります。

```tsx
"use client";

import { useState } from 'react';

export default function UserProfile({ userId }: { userId: string | null }) {
  let content;

  if (userId === null) {
    content = <p>ログインしてください</p>;
  } else {
    content = <p>ユーザーID: {userId}</p>;
  }

  return <div>{content}</div>;
}
```

## 三項演算子（`? :`）

三項演算子は、**条件に応じて2つの値のいずれかを返す**演算子です。

### 基本的な構文

```tsx
条件 ? 真の場合の値 : 偽の場合の値
```

### JSXの中で使う

```tsx
"use client";

import { useState } from 'react';

export default function LoginButton() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);

  return (
    <div>
      {isLoggedIn ? (
        <button onClick={() => setIsLoggedIn(false)}>ログアウト</button>
      ) : (
        <button onClick={() => setIsLoggedIn(true)}>ログイン</button>
      )}
    </div>
  );
}
```

### 実践例: ローディング状態の表示

```tsx
"use client";

import { useState, useEffect } from 'react';

type User = {
  name: string;
};

export default function UserProfile() {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/user')
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      });
  }, []);

  return (
    <div>
      {loading ? (
        <p>読み込み中...</p>
      ) : user ? (
        <p>ようこそ、{user.name}さん！</p>
      ) : (
        <p>ユーザー情報が取得できませんでした</p>
      )}
    </div>
  );
}
```

### 実践例: エラーメッセージの表示

```tsx
"use client";

import { useState } from 'react';

export default function ContactForm() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!email.includes('@')) {
      setError('メールアドレスが正しくありません');
    } else {
      setError('');
      // 送信処理
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      {error ? (
        <p style={{ color: 'red' }}>{error}</p>
      ) : null}
      <button type="submit">送信</button>
    </form>
  );
}
```

## 論理AND演算子（`&&`）

`&&` を使うと、**条件が真の場合だけ要素を表示**できます。

### 基本的な使い方

```tsx
条件 && <JSX要素>
```

条件が `true` のときだけ、右側の要素が表示されます。

```tsx
"use client";

import { useState } from 'react';

export default function Message() {
  const [showMessage, setShowMessage] = useState(false);

  return (
    <div>
      <button onClick={() => setShowMessage(!showMessage)}>
        {showMessage ? '非表示' : '表示'}
      </button>
      {showMessage && <p>メッセージが表示されています</p>}
    </div>
  );
}
```

### 実践例: ローディング中の表示

```tsx
"use client";

import { useState, useEffect } from 'react';

export default function DataList() {
  const [items, setItems] = useState<string[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/items')
      .then(res => res.json())
      .then(data => {
        setItems(data);
        setLoading(false);
      });
  }, []);

  return (
    <div>
      {loading && <p>読み込み中...</p>}
      {!loading && (
        <ul>
          {items.map((item, index) => (
            <li key={index}>{item}</li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

### 実践例: エラーメッセージの表示

```tsx
"use client";

import { useState } from 'react';

export default function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!email || !password) {
      setError('メールアドレスとパスワードを入力してください');
      return;
    }
    
    // ログイン処理
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      {error && <p style={{ color: 'red' }}>{error}</p>}
      <button type="submit">ログイン</button>
    </form>
  );
}
```

## 複数の条件を組み合わせる

### 複数の条件をチェック

```tsx
"use client";

import { useState, useEffect } from 'react';

type User = {
  name: string;
  email: string;
};

export default function UserProfile() {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState('');

  useEffect(() => {
    fetch('/api/user')
      .then(res => {
        if (!res.ok) {
          throw new Error('ユーザー情報の取得に失敗しました');
        }
        return res.json();
      })
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);

  if (loading) {
    return <p>読み込み中...</p>;
  }

  if (error) {
    return <p style={{ color: 'red' }}>{error}</p>;
  }

  if (!user) {
    return <p>ユーザー情報がありません</p>;
  }

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

## よくあるパターン

### パターン1: 早期リターン

条件が満たされない場合は、早期にreturnして処理を終了します。

```tsx
function Component({ userId }: { userId: string | null }) {
  if (!userId) {
    return <p>ユーザーIDが必要です</p>;
  }

  // userId がある場合の処理
  return <div>ユーザーID: {userId}</div>;
}
```

### パターン2: ローディング・エラー・データの3状態

```tsx
"use client";

import { useState, useEffect } from 'react';

type Data = {
  id: string;
  name: string;
};

export default function DataDisplay() {
  const [data, setData] = useState<Data | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState('');

  useEffect(() => {
    fetch('/api/data')
      .then(res => res.json())
      .then(result => {
        setData(result);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);

  if (loading) return <p>読み込み中...</p>;
  if (error) return <p style={{ color: 'red' }}>エラー: {error}</p>;
  if (!data) return <p>データがありません</p>;

  return <div>{data.name}</div>;
}
```

### パターン3: リストが空の場合の表示

```tsx
"use client";

import { useState } from 'react';

export default function TodoList() {
  const [todos, setTodos] = useState<string[]>([]);

  return (
    <div>
      {todos.length === 0 ? (
        <p>タスクがありません</p>
      ) : (
        <ul>
          {todos.map((todo, index) => (
            <li key={index}>{todo}</li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

## この章のまとめ

- if文で条件分岐して、異なるJSXを返せる
- 三項演算子（`? :`）で、条件に応じて2つの値のいずれかを表示できる
- 論理AND演算子（`&&`）で、条件が真の場合だけ要素を表示できる
- 複数の条件を組み合わせて、ローディング・エラー・データの状態を管理できる
- 早期リターンを使うと、コードが読みやすくなる

## 確認してみよう

- [ ] if文で条件分岐できる
- [ ] 三項演算子（`? :`）が使える
- [ ] 論理AND演算子（`&&`）が使える
- [ ] ローディング状態を条件付きレンダリングで表示できる
- [ ] エラーメッセージを条件付きレンダリングで表示できる

:::message
条件付きレンダリングは、実践的なアプリ開発で頻繁に使います。パターンを覚えておくと、開発がスムーズになります。
:::
