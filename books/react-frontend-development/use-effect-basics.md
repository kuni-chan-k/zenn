---
title: "useEffectの基礎"
---

## この章でやること

- `useEffect` とは何かを理解する
- 副作用処理の概念を学ぶ
- `useEffect` の基本的な使い方を覚える
- 依存配列の理解
- クリーンアップ関数の使い方

## useEffect とは

`useEffect` は、**コンポーネントの副作用（side effect）を処理するためのフック**です。

### 副作用（side effect）とは

副作用とは、コンポーネントのレンダリング以外で行う処理のことです。

- データの取得（API呼び出し）
- DOMの操作
- タイマーの設定
- イベントリスナーの登録
- localStorage への保存・読み込み

### なぜ useEffect が必要なのか

`useState` だけでは、コンポーネントが表示されたときに何か処理を実行することができません。

```tsx
// ❌ これは動かない（レンダリング中に副作用を実行できない）
function UserProfile() {
  const [user, setUser] = useState(null);
  
  // これはエラーになる
  fetch('/api/user').then(data => setUser(data));
  
  return <div>{user?.name}</div>;
}
```

`useEffect` を使うと、コンポーネントが表示された後や、特定の値が変わったときに処理を実行できます。

```tsx
// ✅ 正しい書き方
function UserProfile() {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch('/api/user').then(data => setUser(data));
  }, []);
  
  return <div>{user?.name}</div>;
}
```

## useEffect の基本的な使い方

### 基本的な構文

```tsx
import { useEffect } from 'react';

useEffect(() => {
  // ここに実行したい処理を書く
}, [依存配列]);
```

### コンポーネントが表示されたときに実行する

依存配列を空にすると、コンポーネントが最初に表示されたときに1回だけ実行されます。

```tsx
"use client";

import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('コンポーネントが表示されました');
  }, []); // 空の配列 = 最初の1回だけ実行

  return (
    <div>
      <p>カウント: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}
```

### 実践例: localStorage からデータを読み込む

```tsx
"use client";

import { useState, useEffect } from 'react';

export default function TodoApp() {
  const [todos, setTodos] = useState<string[]>([]);

  // コンポーネントが表示されたときに、localStorage からデータを読み込む
  useEffect(() => {
    const savedTodos = localStorage.getItem('todos');
    if (savedTodos) {
      setTodos(JSON.parse(savedTodos));
    }
  }, []);

  const addTodo = (text: string) => {
    const newTodos = [...todos, text];
    setTodos(newTodos);
    localStorage.setItem('todos', JSON.stringify(newTodos));
  };

  return (
    <div>
      {/* ... */}
    </div>
  );
}
```

## 依存配列の理解

依存配列は、`useEffect` を**いつ実行するか**を制御します。

### 依存配列が空の場合

```tsx
useEffect(() => {
  // コンポーネントが最初に表示されたときに1回だけ実行
}, []);
```

### 依存配列に値がある場合

依存配列に指定した値が変わったときに、`useEffect` が実行されます。

```tsx
"use client";

import { useState, useEffect } from 'react';

export default function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);

  // userId が変わったときに実行される
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, [userId]); // userId が変わったときに再実行

  return <div>{user?.name}</div>;
}
```

### 依存配列を省略した場合

依存配列を省略すると、**毎回のレンダリング後に実行**されます。通常は使いません。

```tsx
// ⚠️ 注意: これは毎回実行される（通常は使わない）
useEffect(() => {
  console.log('レンダリングされました');
});
```

:::message alert
依存配列を正しく指定しないと、無限ループが発生したり、期待通りに動かなかったりします。必ず依存配列を意識しましょう。
:::

## クリーンアップ関数

`useEffect` の中で、タイマーやイベントリスナーを設定した場合、コンポーネントが削除される前にそれらをクリーンアップする必要があります。

### クリーンアップ関数の書き方

`useEffect` の戻り値として関数を返すと、それがクリーンアップ関数になります。

```tsx
useEffect(() => {
  // 設定処理
  
  return () => {
    // クリーンアップ処理
  };
}, [依存配列]);
```

### 実践例: タイマーのクリーンアップ

```tsx
"use client";

import { useState, useEffect } from 'react';

export default function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);

    // クリーンアップ関数: コンポーネントが削除される前にタイマーを止める
    return () => {
      clearInterval(interval);
    };
  }, []); // 最初の1回だけ実行

  return <div>経過時間: {seconds}秒</div>;
}
```

### 実践例: イベントリスナーのクリーンアップ

```tsx
"use client";

import { useState, useEffect } from 'react';

export default function WindowSize() {
  const [width, setWidth] = useState(0);

  useEffect(() => {
    const handleResize = () => {
      setWidth(window.innerWidth);
    };

    window.addEventListener('resize', handleResize);
    handleResize(); // 最初のサイズを取得

    // クリーンアップ関数: イベントリスナーを削除
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);

  return <div>ウィンドウ幅: {width}px</div>;
}
```

## よくあるパターン

### パターン1: データの取得

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
    fetch('/api/users')
      .then(res => res.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>読み込み中...</div>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### パターン2: フォームの初期値設定

```tsx
"use client";

import { useState, useEffect } from 'react';

export default function EditForm({ userId }: { userId: string }) {
  const [name, setName] = useState('');

  useEffect(() => {
    // userId が変わったときに、ユーザー情報を取得してフォームに設定
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setName(data.name));
  }, [userId]);

  return (
    <form>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
    </form>
  );
}
```

## useEffect のルール

`useState` と同様に、`useEffect` にも守るべきルールがあります。

1. **コンポーネントのトップレベルでのみ呼び出す**
   - if 文やループの中で呼び出さない

2. **React の関数コンポーネント内でのみ使う**
   - 通常の JavaScript 関数では使えない

```tsx
// ❌ ダメな例
function Component() {
  if (true) {
    useEffect(() => {
      // if文の中はNG
    }, []);
  }
}

// ✅ OKな例
function Component() {
  useEffect(() => {
    // トップレベルでOK
  }, []);
}
```

## よくあるエラーと対処法

### 1. 無限ループが発生する

```tsx
// ❌ 問題: 依存配列に state を入れると無限ループになる
const [count, setCount] = useState(0);

useEffect(() => {
  setCount(count + 1);  // count が変わる → useEffect が実行 → count が変わる → ...
}, [count]);  // count を依存配列に入れている

// ✅ 正しい: 依存配列を空にする、または関数形式を使う
useEffect(() => {
  setCount((prev) => prev + 1);
}, []);  // 1回だけ実行
```

### 2. 依存配列の警告が出る

```tsx
// ⚠️ 警告: React Hook useEffect has a missing dependency
const [userId, setUserId] = useState(1);

useEffect(() => {
  fetch(`/api/users/${userId}`);
}, []);  // userId を使っているのに依存配列にない

// ✅ 正しい: 使っている変数は依存配列に入れる
useEffect(() => {
  fetch(`/api/users/${userId}`);
}, [userId]);  // userId を依存配列に追加
```

### 3. if 文の中で useEffect を呼んでいる

```tsx
// ❌ エラー: React Hook "useEffect" is called conditionally
function Component({ shouldFetch }: { shouldFetch: boolean }) {
  if (shouldFetch) {
    useEffect(() => {
      // if文の中はNG
    }, []);
  }
}

// ✅ 正しい: トップレベルで呼び、条件は中に入れる
function Component({ shouldFetch }: { shouldFetch: boolean }) {
  useEffect(() => {
    if (shouldFetch) {
      // 条件は中に入れる
    }
  }, [shouldFetch]);
}
```

### 4. クリーンアップを忘れている

```tsx
// ❌ 問題: タイマーが削除されずにメモリリークの原因になる
useEffect(() => {
  const timer = setInterval(() => {
    console.log('tick');
  }, 1000);
  // クリーンアップがない
}, []);

// ✅ 正しい: クリーンアップ関数で削除する
useEffect(() => {
  const timer = setInterval(() => {
    console.log('tick');
  }, 1000);
  
  return () => {
    clearInterval(timer);  // クリーンアップ
  };
}, []);
```

## この章のまとめ

- `useEffect` は副作用処理（データ取得、タイマーなど）を行うためのフック
- 依存配列が空なら、コンポーネントが表示されたときに1回だけ実行
- 依存配列に値を指定すると、その値が変わったときに実行
- タイマーやイベントリスナーは、クリーンアップ関数で削除する
- コンポーネントのトップレベルでのみ呼び出す
- 依存配列には、useEffect 内で使う変数を必ず含める

## 確認してみよう

- [ ] `useEffect` が何をするフックか説明できる
- [ ] 依存配列が空の場合と値がある場合の違いを理解した
- [ ] localStorage からデータを読み込むコードが書ける
- [ ] クリーンアップ関数の使い方を理解した
- [ ] タイマーやイベントリスナーのクリーンアップができる

:::message
`useEffect` は最初は難しく感じますが、使っていくうちに理解が深まります。依存配列を意識することが大切です。
:::
