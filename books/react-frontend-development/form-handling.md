---
title: "フォーム処理とバリデーション"
---

## この章でやること

- フォームの送信処理を学ぶ
- `onSubmit` イベントと `preventDefault` の使い方
- 複数入力フィールドの管理
- バリデーション（入力チェック）の実装
- エラーメッセージの表示

## フォーム処理の基礎

### 基本的なフォーム

```tsx
"use client";

import { useState } from 'react';

export default function SimpleForm() {
  const [name, setName] = useState('');

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault(); // ページのリロードを防ぐ
    console.log('送信された名前:', name);
    // ここで送信処理を行う
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="名前を入力"
      />
      <button type="submit">送信</button>
    </form>
  );
}
```

### preventDefault とは

`preventDefault()` は、**フォームのデフォルトの動作（ページのリロード）を防ぐ**メソッドです。

フォームを送信すると、通常はページがリロードされますが、React アプリではページをリロードせずに処理を行いたいので、`preventDefault()` を呼び出します。

```tsx
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault(); // ← これがないとページがリロードされる
  // 送信処理
};
```

## 複数入力フィールドの管理

### 方法1: 各フィールドにuseStateを使う

```tsx
"use client";

import { useState } from 'react';

export default function ContactForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [message, setMessage] = useState('');

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    console.log({ name, email, message });
    // 送信処理
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="名前"
      />
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="メールアドレス"
      />
      <textarea
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        placeholder="メッセージ"
      />
      <button type="submit">送信</button>
    </form>
  );
}
```

### 方法2: オブジェクトで管理する（推奨）

フィールドが増えると、オブジェクトで管理する方が便利です。

```tsx
"use client";

import { useState } from 'react';

type FormData = {
  name: string;
  email: string;
  message: string;
};

export default function ContactForm() {
  const [formData, setFormData] = useState<FormData>({
    name: '',
    email: '',
    message: '',
  });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value,
    }));
  };

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    console.log(formData);
    // 送信処理
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="名前"
      />
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="メールアドレス"
      />
      <textarea
        name="message"
        value={formData.message}
        onChange={handleChange}
        placeholder="メッセージ"
      />
      <button type="submit">送信</button>
    </form>
  );
}
```

:::message
オブジェクトで管理する場合、各入力フィールドに `name` 属性を設定する必要があります。`handleChange` で `e.target.name` を使って、どのフィールドが変更されたかを判断します。
:::

## バリデーション（入力チェック）

### 基本的なバリデーション

```tsx
"use client";

import { useState } from 'react';

type FormData = {
  email: string;
  password: string;
};

type Errors = {
  email?: string;
  password?: string;
};

export default function LoginForm() {
  const [formData, setFormData] = useState<FormData>({
    email: '',
    password: '',
  });
  const [errors, setErrors] = useState<Errors>({});

  const validate = (): boolean => {
    const newErrors: Errors = {};

    // メールアドレスのチェック
    if (!formData.email) {
      newErrors.email = 'メールアドレスを入力してください';
    } else if (!formData.email.includes('@')) {
      newErrors.email = 'メールアドレスの形式が正しくありません';
    }

    // パスワードのチェック
    if (!formData.password) {
      newErrors.password = 'パスワードを入力してください';
    } else if (formData.password.length < 8) {
      newErrors.password = 'パスワードは8文字以上で入力してください';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0; // エラーがない場合は true
  };

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value,
    }));
    // 入力中にエラーをクリア
    if (errors[name as keyof Errors]) {
      setErrors(prev => ({
        ...prev,
        [name]: undefined,
      }));
    }
  };

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    
    if (validate()) {
      console.log('送信:', formData);
      // 送信処理
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          placeholder="メールアドレス"
        />
        {errors.email && <p style={{ color: 'red' }}>{errors.email}</p>}
      </div>
      <div>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
          placeholder="パスワード"
        />
        {errors.password && <p style={{ color: 'red' }}>{errors.password}</p>}
      </div>
      <button type="submit">ログイン</button>
    </form>
  );
}
```

### 実践例: 登録フォーム

```tsx
"use client";

import { useState } from 'react';

type FormData = {
  name: string;
  email: string;
  password: string;
  confirmPassword: string;
};

type Errors = {
  name?: string;
  email?: string;
  password?: string;
  confirmPassword?: string;
};

export default function SignUpForm() {
  const [formData, setFormData] = useState<FormData>({
    name: '',
    email: '',
    password: '',
    confirmPassword: '',
  });
  const [errors, setErrors] = useState<Errors>({});

  const validate = (): boolean => {
    const newErrors: Errors = {};

    if (!formData.name.trim()) {
      newErrors.name = '名前を入力してください';
    }

    if (!formData.email) {
      newErrors.email = 'メールアドレスを入力してください';
    } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(formData.email)) {
      newErrors.email = 'メールアドレスの形式が正しくありません';
    }

    if (!formData.password) {
      newErrors.password = 'パスワードを入力してください';
    } else if (formData.password.length < 8) {
      newErrors.password = 'パスワードは8文字以上で入力してください';
    }

    if (formData.password !== formData.confirmPassword) {
      newErrors.confirmPassword = 'パスワードが一致しません';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value,
    }));
  };

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    
    if (validate()) {
      console.log('登録:', formData);
      // 登録処理
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>名前</label>
        <input
          type="text"
          name="name"
          value={formData.name}
          onChange={handleChange}
        />
        {errors.name && <p style={{ color: 'red' }}>{errors.name}</p>}
      </div>
      <div>
        <label>メールアドレス</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
        />
        {errors.email && <p style={{ color: 'red' }}>{errors.email}</p>}
      </div>
      <div>
        <label>パスワード</label>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
        />
        {errors.password && <p style={{ color: 'red' }}>{errors.password}</p>}
      </div>
      <div>
        <label>パスワード（確認）</label>
        <input
          type="password"
          name="confirmPassword"
          value={formData.confirmPassword}
          onChange={handleChange}
        />
        {errors.confirmPassword && <p style={{ color: 'red' }}>{errors.confirmPassword}</p>}
      </div>
      <button type="submit">登録</button>
    </form>
  );
}
```

## ローディング状態の管理

フォーム送信中は、ボタンを無効化してローディング状態を表示します。

```tsx
"use client";

import { useState } from 'react';

export default function ContactForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: '',
  });
  const [loading, setLoading] = useState(false);
  const [success, setSuccess] = useState(false);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value,
    }));
  };

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setLoading(true);
    setSuccess(false);

    try {
      // 送信処理（API呼び出しなど）
      await fetch('/api/contact', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData),
      });
      
      setSuccess(true);
      setFormData({ name: '', email: '', message: '' });
    } catch (error) {
      console.error('送信エラー:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="名前"
        disabled={loading}
      />
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="メールアドレス"
        disabled={loading}
      />
      <textarea
        name="message"
        value={formData.message}
        onChange={handleChange}
        placeholder="メッセージ"
        disabled={loading}
      />
      {success && <p style={{ color: 'green' }}>送信しました！</p>}
      <button type="submit" disabled={loading}>
        {loading ? '送信中...' : '送信'}
      </button>
    </form>
  );
}
```

## よくあるパターン

### パターン1: フォームのリセット

送信成功後にフォームをリセットします。

```tsx
const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault();
  
  // 送信処理
  await submitForm(formData);
  
  // フォームをリセット
  setFormData({
    name: '',
    email: '',
    message: '',
  });
};
```

### パターン2: 入力中のバリデーション

ユーザーが入力している間に、リアルタイムでバリデーションを行います。

```tsx
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  const { name, value } = e.target;
  setFormData(prev => ({
    ...prev,
    [name]: value,
  }));

  // リアルタイムバリデーション
  if (name === 'email' && value && !value.includes('@')) {
    setErrors(prev => ({
      ...prev,
      email: 'メールアドレスの形式が正しくありません',
    }));
  } else {
    setErrors(prev => ({
      ...prev,
      email: undefined,
    }));
  }
};
```

## この章のまとめ

- `onSubmit` イベントでフォーム送信を処理する
- `preventDefault()` でページのリロードを防ぐ
- 複数フィールドはオブジェクトで管理すると便利
- バリデーションで入力チェックを行う
- エラーメッセージを条件付きレンダリングで表示する
- ローディング状態を管理して、送信中の操作を防ぐ

## 確認してみよう

- [ ] `onSubmit` と `preventDefault` が使える
- [ ] 複数入力フィールドをオブジェクトで管理できる
- [ ] バリデーションを実装できる
- [ ] エラーメッセージを表示できる
- [ ] ローディング状態を管理できる

:::message
フォーム処理は実践的なアプリ開発で頻繁に使います。バリデーションのパターンを覚えておくと、開発がスムーズになります。
:::
