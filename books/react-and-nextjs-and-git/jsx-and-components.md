---
title: "JSXとコンポーネント"
---

## この章でやること

- JSX の書き方を学ぶ
- コンポーネントの作り方を学ぶ
- props（プロップス）でデータを渡す方法を学ぶ

## JSX とは

JSX は、**JavaScript の中に HTML のようなコードを書ける書き方**です。

```tsx
// これが JSX
const element = <h1>Hello, World!</h1>;
```

見た目は HTML ですが、実際は JavaScript（TypeScript）です。React でコンポーネントを作るときに使います。

## JSX のルール

JSX にはいくつかのルールがあります。

### 1. 必ず1つの要素で囲む

```tsx
// ❌ ダメな例（複数の要素がバラバラ）
return (
  <h1>タイトル</h1>
  <p>本文</p>
);

// ✅ OKな例（divで囲む）
return (
  <div>
    <h1>タイトル</h1>
    <p>本文</p>
  </div>
);

// ✅ OKな例（空のタグで囲む）
return (
  <>
    <h1>タイトル</h1>
    <p>本文</p>
  </>
);
```

`<>...</>` は「フラグメント」と呼ばれ、余計な div を増やさずに複数要素を囲めます。

### 2. class は className と書く

HTML の `class` は、JSX では `className` と書きます。

```tsx
// HTML
<div class="container">...</div>

// JSX
<div className="container">...</div>
```

### 3. JavaScript を埋め込むときは `{}` で囲む

```tsx
const name: string = "田中";
const age: number = 25;

return (
  <div>
    <p>名前: {name}</p>
    <p>年齢: {age}歳</p>
    <p>来年: {age + 1}歳</p>
  </div>
);
```

`{}` の中には、変数や計算式など、JavaScript の式を書けます。

### 4. タグは必ず閉じる

HTML では省略できる閉じタグも、JSX では必須です。

```tsx
// ❌ ダメな例
<img src="photo.jpg">
<input type="text">

// ✅ OKな例（自己閉じタグ）
<img src="photo.jpg" />
<input type="text" />
```

## コンポーネントの作り方

コンポーネントは、**JSX を返す関数**として作ります。

```tsx
// コンポーネントの定義
function Greeting() {
  return <h1>こんにちは！</h1>;
}

// コンポーネントの使い方
function App() {
  return (
    <div>
      <Greeting />
      <Greeting />
      <Greeting />
    </div>
  );
}
```

### コンポーネント名のルール

- **大文字で始める**（例: `Greeting`, `ArticleCard`）
- 小文字で始まると、HTML タグとして扱われます

```tsx
// ❌ ダメな例（小文字で始まっている）
function greeting() { ... }

// ✅ OKな例（大文字で始まっている）
function Greeting() { ... }
```

## props（プロップス）

props は、**親コンポーネントから子コンポーネントにデータを渡す仕組み**です。

### props の使い方（TypeScript）

TypeScript では、props の型を定義します。

```tsx
// 型を定義
type GreetingProps = {
  name: string;
};

// 子コンポーネント：props を受け取る
function Greeting(props: GreetingProps) {
  return <h1>こんにちは、{props.name}さん！</h1>;
}

// 親コンポーネント：props を渡す
function App() {
  return (
    <div>
      <Greeting name="田中" />
      <Greeting name="鈴木" />
      <Greeting name="佐藤" />
    </div>
  );
}
```

表示結果:
```
こんにちは、田中さん！
こんにちは、鈴木さん！
こんにちは、佐藤さん！
```

### 分割代入で書く

props は「分割代入」という書き方でスッキリ書けます。

```tsx
type GreetingProps = {
  name: string;
};

// 通常の書き方
function Greeting(props: GreetingProps) {
  return <h1>こんにちは、{props.name}さん！</h1>;
}

// 分割代入で書く（こちらが一般的）
function Greeting({ name }: GreetingProps) {
  return <h1>こんにちは、{name}さん！</h1>;
}
```

### 複数の props を渡す

```tsx
type UserCardProps = {
  name: string;
  age: number;
  job: string;
};

function UserCard({ name, age, job }: UserCardProps) {
  return (
    <div className="card">
      <h2>{name}</h2>
      <p>年齢: {age}歳</p>
      <p>職業: {job}</p>
    </div>
  );
}

function App() {
  return (
    <div>
      <UserCard name="田中" age={25} job="エンジニア" />
      <UserCard name="鈴木" age={30} job="デザイナー" />
    </div>
  );
}
```

:::message
文字列は `"..."` で、数値は `{...}` で渡します。
TypeScript を使うと、間違った型を渡すとエラーで教えてくれます。
:::

## children props

特別な props として `children` があります。タグで囲んだ中身を受け取れます。

```tsx
type CardProps = {
  children: React.ReactNode;
};

function Card({ children }: CardProps) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

function App() {
  return (
    <Card>
      <h2>タイトル</h2>
      <p>本文がここに入ります</p>
    </Card>
  );
}
```

:::message
`React.ReactNode` は、JSX で表示できるすべてのもの（テキスト、要素、配列など）を表す型です。
:::

## 実際に書いてみよう

以下のコードを写経して、動きを確認してみましょう。

```tsx
// ボタンコンポーネント
type ButtonProps = {
  text: string;
  color: string;
};

function Button({ text, color }: ButtonProps) {
  return (
    <button style={{ backgroundColor: color, color: 'white', padding: '10px 20px' }}>
      {text}
    </button>
  );
}

// メッセージコンポーネント
type MessageProps = {
  title: string;
  children: React.ReactNode;
};

function Message({ title, children }: MessageProps) {
  return (
    <div style={{ border: '1px solid #ccc', padding: '16px', margin: '8px' }}>
      <h3>{title}</h3>
      <div>{children}</div>
    </div>
  );
}

// メインのアプリ
function App() {
  return (
    <div>
      <h1>コンポーネントの練習</h1>

      <Button text="送信" color="blue" />
      <Button text="キャンセル" color="gray" />

      <Message title="お知らせ">
        <p>これはお知らせの内容です。</p>
      </Message>

      <Message title="警告">
        <p>注意してください！</p>
      </Message>
    </div>
  );
}
```

## この章のまとめ

- JSX は JavaScript の中に HTML 風のコードを書ける
- コンポーネントは大文字で始まる関数として作る
- props で親から子にデータを渡せる
- TypeScript では props の型を定義する
- children で囲んだ中身を受け取れる

## 確認してみよう

- [ ] JSX の基本ルールを理解した
- [ ] コンポーネントを作れる
- [ ] props でデータを渡せる
- [ ] props に型を付けられる
- [ ] children の使い方を理解した
