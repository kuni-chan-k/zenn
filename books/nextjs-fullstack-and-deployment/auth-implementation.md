---
title: "認証機能の実装"
---

## この章でやること

- ユーザーテーブルの追加
- JWT 認証の実装
- ログイン・サインアップ機能

## なぜ認証が必要？

大会管理アプリでは、以下のような制御が必要です：

- **誰でも見れる**: 大会一覧、順位表
- **ログインユーザーのみ**: 自分の大会の編集
- **管理者のみ**: すべての大会の管理

これを実現するために「認証（Authentication）」が必要です。

## 認証の方式

いくつかの方式がありますが、この本では **JWT（JSON Web Token）** を使います。

| 方式 | 特徴 |
|------|------|
| セッション | サーバー側で状態を保持。シンプルだが、スケールしにくい |
| JWT | トークンベース。サーバーレスでも使える |
| OAuth | 外部サービス（Google等）で認証。実装が簡単 |

## スキーマの更新

`prisma/schema.prisma` にユーザーモデルを追加：

```prisma
// ユーザー
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  password  String
  name      String
  role      String   @default("user") // user, organizer, admin
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("users")
}
```

マイグレーションを実行：

```bash
npx prisma migrate dev --name add-user
```

## 必要なパッケージのインストール

```bash
npm install bcryptjs jsonwebtoken
npm install --save-dev @types/bcryptjs @types/jsonwebtoken
```

- **bcryptjs**: パスワードのハッシュ化
- **jsonwebtoken**: JWT の生成・検証

## 環境変数の追加

`.env` に追加：

```env
JWT_SECRET="your-super-secret-key-change-in-production"
```

:::message alert
`JWT_SECRET` は本番環境では必ず変更してください。推測されにくい長いランダムな文字列を使います。
:::

## 認証ユーティリティ

`lib/auth.ts`:

```typescript
import jwt from "jsonwebtoken";
import bcrypt from "bcryptjs";
import { cookies } from "next/headers";
import { prisma } from "./prisma";

const JWT_SECRET = process.env.JWT_SECRET!;
const COOKIE_NAME = "auth-token";

// パスワードのハッシュ化
export async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, 10);
}

// パスワードの検証
export async function verifyPassword(
  password: string,
  hashedPassword: string
): Promise<boolean> {
  return bcrypt.compare(password, hashedPassword);
}

// JWT トークンの生成
export function generateToken(userId: string): string {
  return jwt.sign({ userId }, JWT_SECRET, { expiresIn: "7d" });
}

// JWT トークンの検証
export function verifyToken(token: string): { userId: string } | null {
  try {
    return jwt.verify(token, JWT_SECRET) as { userId: string };
  } catch {
    return null;
  }
}

// 現在のユーザーを取得
export async function getCurrentUser() {
  const cookieStore = await cookies();
  const token = cookieStore.get(COOKIE_NAME)?.value;

  if (!token) return null;

  const payload = verifyToken(token);
  if (!payload) return null;

  const user = await prisma.user.findUnique({
    where: { id: payload.userId },
    select: {
      id: true,
      email: true,
      name: true,
      role: true,
    },
  });

  return user;
}

// ログイン状態をチェック
export async function requireAuth() {
  const user = await getCurrentUser();
  if (!user) {
    throw new Error("Unauthorized");
  }
  return user;
}
```

## サインアップ API

`app/api/auth/signup/route.ts`:

```typescript
import { NextRequest, NextResponse } from "next/server";
import { cookies } from "next/headers";
import { prisma } from "@/lib/prisma";
import { hashPassword, generateToken } from "@/lib/auth";

export async function POST(request: NextRequest) {
  try {
    const { email, password, name } = await request.json();

    // バリデーション
    if (!email || !password || !name) {
      return NextResponse.json(
        { error: "必須項目を入力してください" },
        { status: 400 }
      );
    }

    // 既存ユーザーチェック
    const existingUser = await prisma.user.findUnique({
      where: { email },
    });

    if (existingUser) {
      return NextResponse.json(
        { error: "このメールアドレスは既に登録されています" },
        { status: 400 }
      );
    }

    // ユーザー作成
    const hashedPassword = await hashPassword(password);
    const user = await prisma.user.create({
      data: {
        email,
        password: hashedPassword,
        name,
      },
    });

    // トークン生成 & Cookie設定
    const token = generateToken(user.id);
    const cookieStore = await cookies();
    cookieStore.set("auth-token", token, {
      httpOnly: true,
      secure: process.env.NODE_ENV === "production",
      sameSite: "lax",
      maxAge: 60 * 60 * 24 * 7, // 7日
    });

    return NextResponse.json({
      user: {
        id: user.id,
        email: user.email,
        name: user.name,
        role: user.role,
      },
    });
  } catch (error) {
    console.error("Signup error:", error);
    return NextResponse.json(
      { error: "サーバーエラーが発生しました" },
      { status: 500 }
    );
  }
}
```

## ログイン API

`app/api/auth/login/route.ts`:

```typescript
import { NextRequest, NextResponse } from "next/server";
import { cookies } from "next/headers";
import { prisma } from "@/lib/prisma";
import { verifyPassword, generateToken } from "@/lib/auth";

export async function POST(request: NextRequest) {
  try {
    const { email, password } = await request.json();

    // ユーザー検索
    const user = await prisma.user.findUnique({
      where: { email },
    });

    if (!user) {
      return NextResponse.json(
        { error: "メールアドレスまたはパスワードが正しくありません" },
        { status: 401 }
      );
    }

    // パスワード検証
    const isValid = await verifyPassword(password, user.password);
    if (!isValid) {
      return NextResponse.json(
        { error: "メールアドレスまたはパスワードが正しくありません" },
        { status: 401 }
      );
    }

    // トークン生成 & Cookie設定
    const token = generateToken(user.id);
    const cookieStore = await cookies();
    cookieStore.set("auth-token", token, {
      httpOnly: true,
      secure: process.env.NODE_ENV === "production",
      sameSite: "lax",
      maxAge: 60 * 60 * 24 * 7,
    });

    return NextResponse.json({
      user: {
        id: user.id,
        email: user.email,
        name: user.name,
        role: user.role,
      },
    });
  } catch (error) {
    console.error("Login error:", error);
    return NextResponse.json(
      { error: "サーバーエラーが発生しました" },
      { status: 500 }
    );
  }
}
```

## ログアウト API

`app/api/auth/logout/route.ts`:

```typescript
import { NextResponse } from "next/server";
import { cookies } from "next/headers";

export async function POST() {
  const cookieStore = await cookies();
  cookieStore.delete("auth-token");

  return NextResponse.json({ success: true });
}
```

## 現在のユーザー取得 API

`app/api/auth/me/route.ts`:

```typescript
import { NextResponse } from "next/server";
import { getCurrentUser } from "@/lib/auth";

export async function GET() {
  const user = await getCurrentUser();

  if (!user) {
    return NextResponse.json({ user: null });
  }

  return NextResponse.json({ user });
}
```

## ログインページ

`app/login/page.tsx`:

```tsx
"use client";

import { useState } from "react";
import { useRouter } from "next/navigation";
import Link from "next/link";
import { Button, Input, Card } from "@/components/ui";

export default function LoginPage() {
  const router = useRouter();
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");
    setLoading(true);

    try {
      const res = await fetch("/api/auth/login", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email, password }),
      });

      const data = await res.json();

      if (!res.ok) {
        setError(data.error);
        return;
      }

      router.push("/");
      router.refresh();
    } catch {
      setError("エラーが発生しました");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-100">
      <Card title="ログイン">
        <form onSubmit={handleSubmit} className="w-80">
          {error && (
            <div className="mb-4 p-3 bg-red-100 text-red-700 rounded">
              {error}
            </div>
          )}

          <Input
            label="メールアドレス"
            name="email"
            type="text"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            required
          />

          <Input
            label="パスワード"
            name="password"
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            required
          />

          <Button type="submit" disabled={loading}>
            {loading ? "ログイン中..." : "ログイン"}
          </Button>

          <p className="mt-4 text-sm text-gray-600">
            アカウントをお持ちでない方は{" "}
            <Link href="/signup" className="text-blue-600 hover:underline">
              新規登録
            </Link>
          </p>
        </form>
      </Card>
    </div>
  );
}
```

## 権限チェック

API やページで権限をチェックする例：

```typescript
import { getCurrentUser } from "@/lib/auth";

// API Route での例
export async function POST(request: NextRequest) {
  const user = await getCurrentUser();

  if (!user) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  if (user.role !== "admin" && user.role !== "organizer") {
    return NextResponse.json({ error: "Forbidden" }, { status: 403 });
  }

  // 処理を続行...
}
```

## Git でコミット

```bash
git add .
git commit -m "JWT認証機能を実装"
```

## この章のまとめ

- ユーザーモデルを追加した
- JWT 認証を実装した
- ログイン・サインアップ・ログアウト機能を作成した
- 権限チェックの方法を学んだ

## 確認してみよう

- [ ] サインアップで新規ユーザーが作成できる
- [ ] ログインできる
- [ ] ログアウトできる
- [ ] ログイン状態が維持される（ページ再読み込み後も）
