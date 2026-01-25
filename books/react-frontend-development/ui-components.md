---
title: "UIコンポーネント"
---

## この章でやること

- 再利用可能な UI コンポーネントを作成
- Button, Input, Card, Table の実装

## なぜ UI コンポーネントを作るのか

アプリ全体で統一されたデザインを保つために、基本的な UI パーツをコンポーネント化します。

- ボタンのスタイルが統一される
- 修正が1箇所で済む
- コードがシンプルになる

## Button コンポーネント

`components/ui/Button.tsx`:

```tsx
type ButtonProps = {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: "primary" | "secondary" | "danger";
  type?: "button" | "submit";
  disabled?: boolean;
};

export default function Button({
  children,
  onClick,
  variant = "primary",
  type = "button",
  disabled = false,
}: ButtonProps) {
  const baseStyles = "px-4 py-2 rounded font-medium transition-colors";

  const variantStyles = {
    primary: "bg-blue-600 text-white hover:bg-blue-700 disabled:bg-blue-300",
    secondary: "bg-gray-200 text-gray-800 hover:bg-gray-300 disabled:bg-gray-100",
    danger: "bg-red-600 text-white hover:bg-red-700 disabled:bg-red-300",
  };

  return (
    <button
      type={type}
      onClick={onClick}
      disabled={disabled}
      className={`${baseStyles} ${variantStyles[variant]}`}
    >
      {children}
    </button>
  );
}
```

### 使い方

```tsx
<Button>保存</Button>
<Button variant="secondary">キャンセル</Button>
<Button variant="danger">削除</Button>
<Button disabled>送信中...</Button>
```

## Input コンポーネント

`components/ui/Input.tsx`:

```tsx
type InputProps = {
  label: string;
  name: string;
  type?: "text" | "number" | "date" | "datetime-local";
  value: string | number;
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
  placeholder?: string;
  required?: boolean;
  error?: string;
};

export default function Input({
  label,
  name,
  type = "text",
  value,
  onChange,
  placeholder,
  required = false,
  error,
}: InputProps) {
  return (
    <div className="mb-4">
      <label htmlFor={name} className="block text-sm font-medium text-gray-700 mb-1">
        {label}
        {required && <span className="text-red-500 ml-1">*</span>}
      </label>
      <input
        id={name}
        name={name}
        type={type}
        value={value}
        onChange={onChange}
        placeholder={placeholder}
        required={required}
        className={`w-full px-3 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 ${
          error ? "border-red-500" : "border-gray-300"
        }`}
      />
      {error && <p className="mt-1 text-sm text-red-500">{error}</p>}
    </div>
  );
}
```

### 使い方

```tsx
<Input
  label="大会名"
  name="name"
  value={name}
  onChange={(e) => setName(e.target.value)}
  required
/>
```

## Select コンポーネント

`components/ui/Select.tsx`:

```tsx
type Option = {
  value: string;
  label: string;
};

type SelectProps = {
  label: string;
  name: string;
  value: string;
  onChange: (e: React.ChangeEvent<HTMLSelectElement>) => void;
  options: Option[];
  required?: boolean;
  placeholder?: string;
};

export default function Select({
  label,
  name,
  value,
  onChange,
  options,
  required = false,
  placeholder = "選択してください",
}: SelectProps) {
  return (
    <div className="mb-4">
      <label htmlFor={name} className="block text-sm font-medium text-gray-700 mb-1">
        {label}
        {required && <span className="text-red-500 ml-1">*</span>}
      </label>
      <select
        id={name}
        name={name}
        value={value}
        onChange={onChange}
        required={required}
        className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
      >
        <option value="">{placeholder}</option>
        {options.map((option) => (
          <option key={option.value} value={option.value}>
            {option.label}
          </option>
        ))}
      </select>
    </div>
  );
}
```

## Card コンポーネント

`components/ui/Card.tsx`:

```tsx
type CardProps = {
  children: React.ReactNode;
  title?: string;
};

export default function Card({ children, title }: CardProps) {
  return (
    <div className="bg-white rounded-lg shadow p-6">
      {title && (
        <h2 className="text-lg font-semibold mb-4">{title}</h2>
      )}
      {children}
    </div>
  );
}
```

### 使い方

```tsx
<Card title="大会情報">
  <p>春季フットサルリーグ 2024</p>
</Card>
```

## Table コンポーネント

`components/ui/Table.tsx`:

```tsx
type Column<T> = {
  key: keyof T | string;
  header: string;
  render?: (item: T) => React.ReactNode;
};

type TableProps<T> = {
  columns: Column<T>[];
  data: T[];
  emptyMessage?: string;
};

export default function Table<T extends { id: string }>({
  columns,
  data,
  emptyMessage = "データがありません",
}: TableProps<T>) {
  if (data.length === 0) {
    return (
      <div className="text-center py-8 text-gray-500">
        {emptyMessage}
      </div>
    );
  }

  return (
    <div className="overflow-x-auto">
      <table className="w-full border-collapse">
        <thead>
          <tr className="bg-gray-50">
            {columns.map((column) => (
              <th
                key={String(column.key)}
                className="px-4 py-3 text-left text-sm font-semibold text-gray-700 border-b"
              >
                {column.header}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {data.map((item) => (
            <tr key={item.id} className="hover:bg-gray-50">
              {columns.map((column) => (
                <td
                  key={`${item.id}-${String(column.key)}`}
                  className="px-4 py-3 text-sm text-gray-600 border-b"
                >
                  {column.render
                    ? column.render(item)
                    : String(item[column.key as keyof T] ?? "")}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

### 使い方

```tsx
const columns = [
  { key: "name", header: "大会名" },
  { key: "date", header: "開催日" },
  {
    key: "actions",
    header: "操作",
    render: (item) => <Button>編集</Button>,
  },
];

<Table columns={columns} data={competitions} />
```

## コンポーネント一覧のエクスポート

`components/ui/index.ts`:

```tsx
export { default as Button } from "./Button";
export { default as Input } from "./Input";
export { default as Select } from "./Select";
export { default as Card } from "./Card";
export { default as Table } from "./Table";
```

### インポートが簡単に

```tsx
// 個別にインポートする代わりに
import Button from "@/components/ui/Button";
import Input from "@/components/ui/Input";

// まとめてインポートできる
import { Button, Input, Card } from "@/components/ui";
```

## Git でコミット

```bash
git add .
git commit -m "UIコンポーネントを追加"
```

## この章のまとめ

- 再利用可能な UI コンポーネントを作成した
- TypeScript で props の型を定義した
- インデックスファイルでエクスポートを整理した

## 確認してみよう

- [ ] Button コンポーネントが 3 つのバリエーションで動作する
- [ ] Input コンポーネントでエラー表示ができる
- [ ] Table コンポーネントでデータが表示される
- [ ] 空の状態でメッセージが表示される
