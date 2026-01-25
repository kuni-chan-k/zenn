---
title: "状態管理の基礎（useState）"
---

## この章でやること

- state（状態）の概念を理解する
- useState フックの使い方を学ぶ
- イベント処理を学ぶ

## state（状態）とは

state は、**コンポーネントが持つ「変化するデータ」**です。

たとえば、以下のようなものが state になります。

- カウンターの数値
- 入力フォームのテキスト
- チェックボックスのオン/オフ
- モーダルの開閉状態

state が変わると、React は自動的に画面を更新します。

## なぜ普通の変数ではダメなのか

普通の変数では、値を変えても画面は更新されません。

```tsx
// ❌ これでは動かない
function Counter() {
  let count = 0;

  const handleClick = () => {
    count = count + 1;  // 値は変わるけど...
    console.log(count); // ログには出る
  };

  return (
    <div>
      <p>カウント: {count}</p>  {/* 画面は更新されない！ */}
      <button onClick={handleClick}>+1</button>
    </div>
  );
}
```

React に「値が変わったので画面を更新して」と伝えるために、state を使います。

## useState の使い方

`useState` は、state を使うための関数（フック）です。

```tsx
import { useState } from 'react';

function Counter() {
  // useState の使い方
  const [count, setCount] = useState(0);
  //     ↑       ↑              ↑
  //   現在の値  更新関数    初期値

  const handleClick = () => {
    setCount(count + 1);  // これで画面が更新される
  };

  return (
    <div>
      <p>カウント: {count}</p>
      <button onClick={handleClick}>+1</button>
    </div>
  );
}
```

### useState の構文

```tsx
const [値, 更新関数] = useState(初期値);
```

- **値**: 現在の state の値
- **更新関数**: state を更新するための関数（`set` + 名前 が慣例）
- **初期値**: 最初の値

### よくある使い方（TypeScript）

TypeScript では、初期値から型が自動で推論されます。複雑な型は明示的に指定します。

```tsx
// 数値（型は自動で number に推論される）
const [count, setCount] = useState(0);

// 文字列（型は自動で string に推論される）
const [name, setName] = useState("");

// 真偽値（型は自動で boolean に推論される）
const [isOpen, setIsOpen] = useState(false);

// 配列（初期値が空の場合、型を指定する）
const [items, setItems] = useState<string[]>([]);

// オブジェクト（初期値から推論、または型を指定）
const [user, setUser] = useState({ name: "", age: 0 });

// 型を明示的に指定する例
type User = {
  name: string;
  age: number;
};
const [user, setUser] = useState<User>({ name: "", age: 0 });
```

:::message
空の配列 `[]` や `null` が初期値の場合、TypeScript に型を教えてあげる必要があります。
`useState<型>(初期値)` の形で指定します。
:::

## イベント処理

ボタンのクリックや入力など、ユーザーの操作を「イベント」と呼びます。

### クリックイベント

```tsx
function Button() {
  const handleClick = () => {
    alert('クリックされました！');
  };

  return <button onClick={handleClick}>クリック</button>;
}
```

`onClick` に関数を渡します。`()` をつけないように注意！

```tsx
// ❌ ダメな例（即座に実行されてしまう）
<button onClick={handleClick()}>クリック</button>

// ✅ 正しい例（関数を渡す）
<button onClick={handleClick}>クリック</button>
```

### 入力イベント

```tsx
import { useState } from 'react';

function TextInput() {
  const [text, setText] = useState("");

  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    setText(event.target.value);
  };

  return (
    <div>
      <input type="text" value={text} onChange={handleChange} />
      <p>入力した文字: {text}</p>
    </div>
  );
}
```

`event.target.value` で入力された値を取得できます。

:::message
`React.ChangeEvent<HTMLInputElement>` は、input要素の変更イベントを表す型です。
VSCode では自動補完が効くので、すべてを暗記する必要はありません。
:::

### インラインで書く場合

短い処理は、直接書くこともできます。

```tsx
<input
  type="text"
  value={text}
  onChange={(e) => setText(e.target.value)}
/>
```

インラインで書く場合、型は自動で推論されるので省略できます。

## 実践: カウンターアプリ

学んだことを使って、カウンターアプリを作ってみましょう。

```tsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);
  const reset = () => setCount(0);

  return (
    <div style={{ textAlign: 'center', padding: '20px' }}>
      <h1>カウンター</h1>
      <p style={{ fontSize: '48px' }}>{count}</p>
      <button onClick={decrement}>-1</button>
      <button onClick={reset}>リセット</button>
      <button onClick={increment}>+1</button>
    </div>
  );
}
```

## 実践: Todo リスト（簡易版）

もう少し複雑な例として、簡単な Todo リストを作ってみましょう。

```tsx
import { useState } from 'react';

function TodoApp() {
  const [todos, setTodos] = useState<string[]>([]);
  const [inputText, setInputText] = useState("");

  const addTodo = () => {
    if (inputText.trim() === "") return;  // 空文字は追加しない

    setTodos([...todos, inputText]);  // 配列に追加
    setInputText("");  // 入力欄をクリア
  };

  return (
    <div style={{ padding: '20px' }}>
      <h1>Todoリスト</h1>

      <div>
        <input
          type="text"
          value={inputText}
          onChange={(e) => setInputText(e.target.value)}
          placeholder="やることを入力"
        />
        <button onClick={addTodo}>追加</button>
      </div>

      <ul>
        {todos.map((todo, index) => (
          <li key={index}>{todo}</li>
        ))}
      </ul>
    </div>
  );
}
```

### ポイント解説

1. **`useState<string[]>([])`** - 文字列の配列であることを TypeScript に伝える
2. **`[...todos, inputText]`** - スプレッド構文で、既存の配列に新しい要素を追加
3. **`todos.map(...)`** - 配列の各要素をリスト表示
4. **`key={index}`** - React がリストの各要素を識別するために必要

:::message
配列の state を更新するときは、元の配列を直接変更せず、新しい配列を作ります。これは React の重要なルールです。
:::

## フックのルール

`useState` などのフック（`use` で始まる関数）には、守るべきルールがあります。

1. **コンポーネントのトップレベルでのみ呼び出す**
   - if 文やループの中で呼び出さない
2. **React の関数コンポーネント内でのみ使う**
   - 通常の JavaScript 関数では使えない

```tsx
// ❌ ダメな例
function Counter() {
  if (true) {
    const [count, setCount] = useState(0);  // if文の中はNG
  }
}

// ✅ OKな例
function Counter() {
  const [count, setCount] = useState(0);  // トップレベルでOK

  if (count > 10) {
    // state を使う処理はOK
  }
}
```

## この章のまとめ

- state は変化するデータを管理する仕組み
- `useState` で state を作成・更新する
- state が変わると、React が自動で画面を更新する
- TypeScript では型が自動推論される（複雑な型は明示的に指定）
- イベント処理で state を更新することで、インタラクティブな UI を作れる

## 確認してみよう

- [ ] state と普通の変数の違いを説明できる
- [ ] useState の使い方を理解した
- [ ] クリックイベントで state を更新できる
- [ ] 入力フォームの値を state で管理できる
- [ ] 配列の state に型を指定できる
