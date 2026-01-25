---
title: "TypeScriptの基礎"
---

## この章でやること

- TypeScript とは何かを理解する
- 基本的な型を学ぶ
- React で TypeScript を使う方法を学ぶ

## TypeScript とは

TypeScript は、**JavaScript に「型」を追加した言語**です。Microsoft が開発し、今では React 開発の標準になっています。

### 「型」って何？

型とは、データの種類を明示することです。

```typescript
// JavaScript - 何を入れてもOK
let name = "田中";
name = 123;      // エラーにならない（でもバグの原因に）

// TypeScript - 型を指定
let name: string = "田中";
name = 123;      // エラー！ string に number は入れられない
```

### TypeScript のメリット

1. **間違いを事前に防げる** - 実行前にエラーがわかる
2. **補完が効く** - VSCode が候補を出してくれる
3. **読みやすい** - 何が入るか型を見ればわかる

### なぜ TypeScript を使うのか？

JavaScript でもアプリは作れますが、TypeScript を使うと以下のメリットがあります。

**1. バグを減らせる**

```typescript
// JavaScript - 実行して初めてエラーがわかる
function greet(name) {
  return `こんにちは、${name}さん！`;
}

greet(123);  // 実行すると "こんにちは、123さん！" と表示（バグ）

// TypeScript - 書いているときにエラーがわかる
function greet(name: string) {
  return `こんにちは、${name}さん！`;
}

greet(123);  // エラー！ 書いているときに気づける
```

**2. コードが読みやすくなる**

型を見れば、その関数が何を受け取って何を返すかが一目でわかります。

```typescript
// 型を見れば、この関数の使い方がわかる
function calculateTotal(price: number, quantity: number): number {
  return price * quantity;
}
```

**3. チーム開発で役立つ**

他の人が書いたコードでも、型を見れば使い方がわかります。ドキュメントを読む時間が減ります。

**4. リファクタリング（コードの改善）が安全**

コードを変更するとき、型が合わないとエラーが出るので、間違った変更に気づけます。

## 基本的な型

### プリミティブ型（基本の型）

```typescript
// 文字列
const name: string = "田中";

// 数値
const age: number = 25;

// 真偽値（true か false）
const isStudent: boolean = true;
```

### 配列

```typescript
// 文字列の配列
const fruits: string[] = ["りんご", "みかん", "バナナ"];

// 数値の配列
const scores: number[] = [80, 90, 75];
```

### オブジェクト

```typescript
// オブジェクトの型を定義
type User = {
  name: string;
  age: number;
};

// 型に従ったオブジェクトを作成
const user: User = {
  name: "田中",
  age: 25,
};
```

## 型推論

TypeScript は賢いので、型を書かなくても推測してくれます。

```typescript
// 型を書かなくても...
let count = 0;

// TypeScript が number だと推測してくれる
count = "hello";  // エラー！ string は入れられない
```

最初のうちは、**型を明示的に書く**ことをおすすめします。慣れてきたら省略できるところは省略しましょう。

## 関数の型

関数には、引数と戻り値に型をつけます。

```typescript
// 引数と戻り値に型をつける
function greet(name: string): string {
  return `こんにちは、${name}さん！`;
}

// アロー関数の場合
const add = (a: number, b: number): number => {
  return a + b;
};
```

### 戻り値がない関数

```typescript
function sayHello(name: string): void {
  console.log(`Hello, ${name}!`);
  // return がない場合は void
}
```

## React での TypeScript

### コンポーネントの props に型をつける

```tsx
// props の型を定義
type GreetingProps = {
  name: string;
  age?: number;  // ? をつけると省略可能
};

// コンポーネントで使う
function Greeting({ name, age }: GreetingProps) {
  return (
    <div>
      <p>こんにちは、{name}さん！</p>
      {age && <p>年齢: {age}歳</p>}
    </div>
  );
}

// 使い方
<Greeting name="田中" />           // OK
<Greeting name="田中" age={25} />  // OK
<Greeting name={123} />            // エラー！
```

### useState の型

```tsx
// 型を指定
const [count, setCount] = useState<number>(0);

// 型推論に任せる（初期値から推測）
const [name, setName] = useState("田中");  // string と推測される

// 配列やオブジェクトの場合は明示的に
type Todo = {
  id: number;
  text: string;
  done: boolean;
};

const [todos, setTodos] = useState<Todo[]>([]);
```

### イベントの型

```tsx
// input の onChange イベント
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  console.log(e.target.value);
};

// button の onClick イベント
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
  console.log("クリックされた！");
};

// フォームの onSubmit イベント
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault();
  // フォーム送信処理
};
```

:::message
イベントの型は覚える必要はありません。VSCode が教えてくれます。
:::

## よく使う型

### ユニオン型（複数の型のいずれか）

```typescript
// 文字列または数値
type ID = string | number;

// 特定の文字列のいずれか
type Status = "pending" | "completed" | "cancelled";

let status: Status = "pending";
status = "completed";  // OK
status = "done";       // エラー！ "done" は Status に含まれない
```

### オプショナル（省略可能）

```typescript
type User = {
  name: string;
  age?: number;  // あってもなくてもOK
};

const user1: User = { name: "田中" };           // OK
const user2: User = { name: "田中", age: 25 };  // OK
```

## 実践: 型付きカウンター

前の章で作ったカウンターに型をつけてみましょう。

```tsx
"use client";

import { useState } from "react";

// props の型
type CounterProps = {
  initialCount?: number;  // 省略可能、デフォルトは 0
};

export default function Counter({ initialCount = 0 }: CounterProps) {
  // useState に number 型を指定
  const [count, setCount] = useState<number>(initialCount);

  // 関数の型
  const increment = (): void => {
    setCount(count + 1);
  };

  const decrement = (): void => {
    setCount(count - 1);
  };

  const reset = (): void => {
    setCount(initialCount);
  };

  return (
    <div>
      <p>カウント: {count}</p>
      <button onClick={decrement}>-1</button>
      <button onClick={reset}>リセット</button>
      <button onClick={increment}>+1</button>
    </div>
  );
}
```

## TypeScript のエラーの読み方

TypeScript のエラーは最初は難しく感じますが、パターンを覚えれば大丈夫です。

### よくあるエラー

**1. 型が合わない**
```
Type 'number' is not assignable to type 'string'.
```
→ string のところに number を入れようとしている

**2. プロパティがない**
```
Property 'age' does not exist on type 'User'.
```
→ User 型に age プロパティが定義されていない

**3. 必須のプロパティがない**
```
Property 'name' is missing in type '{}' but required in type 'User'.
```
→ User 型には name が必須なのに指定していない

## この章のまとめ

- TypeScript は JavaScript に型を追加した言語
- 基本の型: string, number, boolean, 配列, オブジェクト
- `type` でオブジェクトの型を定義できる
- React の props や state にも型をつけられる
- VSCode が補完やエラーを教えてくれる

## 確認してみよう

- [ ] TypeScript が何かを説明できる
- [ ] 基本的な型（string, number, boolean）を理解した
- [ ] `type` でオブジェクトの型を定義できる
- [ ] コンポーネントの props に型をつけられる
- [ ] useState に型を指定できる

:::message
TypeScript は使っていくうちに慣れます。最初から完璧に理解しようとせず、エラーが出たら調べる、という姿勢で大丈夫です！
:::
