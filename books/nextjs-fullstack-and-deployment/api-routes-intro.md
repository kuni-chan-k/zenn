---
title: "API Routes入門"
---

## この章でやること

- Next.js の API Routes を理解する
- 簡単な API を作成する
- リクエストの受け取り方を学ぶ

## API Routesとは

Next.js の **API Routes** は、バックエンドのAPIを作る機能です。

`app/api/` フォルダに `route.ts` ファイルを置くと、APIエンドポイントになります。

```
app/
├── page.tsx                    ← フロントエンド
└── api/
    └── hello/
        └── route.ts            ← /api/hello にアクセスできる
```

## 最初のAPI

簡単なAPIを作ってみましょう。

`app/api/hello/route.ts`:

```typescript
import { NextResponse } from 'next/server';

export async function GET() {
  return NextResponse.json({ message: 'Hello, World!' });
}
```

ブラウザで `http://localhost:3000/api/hello` にアクセスすると、以下が表示されます。

```json
{ "message": "Hello, World!" }
```

## HTTPメソッドの対応

関数名でHTTPメソッドを指定します。

```typescript
// GET リクエストを処理
export async function GET() {
  return NextResponse.json({ message: 'GETリクエスト' });
}

// POST リクエストを処理
export async function POST() {
  return NextResponse.json({ message: 'POSTリクエスト' });
}

// PUT リクエストを処理
export async function PUT() {
  return NextResponse.json({ message: 'PUTリクエスト' });
}

// DELETE リクエストを処理
export async function DELETE() {
  return NextResponse.json({ message: 'DELETEリクエスト' });
}
```

## リクエストの受け取り

### リクエストボディの取得

POSTやPUTでは、クライアントからデータを受け取ります。

```typescript
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  // リクエストボディを取得
  const body = await request.json();

  console.log(body); // { name: '春季大会' }

  return NextResponse.json({
    message: '作成しました',
    data: body
  });
}
```

### URLパラメータの取得

`/api/competitions/1` のような URL から ID を取得します。

`app/api/competitions/[id]/route.ts`:

```typescript
import { NextRequest, NextResponse } from 'next/server';

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const id = params.id; // "1"

  return NextResponse.json({
    message: `ID: ${id} の大会を取得しました`
  });
}
```

## レスポンスの返し方

### 成功レスポンス

```typescript
// 200 OK（デフォルト）
return NextResponse.json({ data: competitions });

// 201 Created
return NextResponse.json(
  { message: '作成しました' },
  { status: 201 }
);
```

### エラーレスポンス

```typescript
// 400 Bad Request
return NextResponse.json(
  { error: '名前は必須です' },
  { status: 400 }
);

// 404 Not Found
return NextResponse.json(
  { error: '見つかりませんでした' },
  { status: 404 }
);

// 500 Internal Server Error
return NextResponse.json(
  { error: 'サーバーエラー' },
  { status: 500 }
);
```

## 実践例：大会API

`app/api/competitions/route.ts`:

```typescript
import { NextRequest, NextResponse } from 'next/server';

// 仮のデータ（本来はデータベースから取得）
let competitions = [
  { id: '1', name: '春季大会', status: 'upcoming' },
  { id: '2', name: '秋季大会', status: 'ongoing' },
];

// GET: 大会一覧を取得
export async function GET() {
  return NextResponse.json(competitions);
}

// POST: 新しい大会を作成
export async function POST(request: NextRequest) {
  const body = await request.json();

  // バリデーション
  if (!body.name) {
    return NextResponse.json(
      { error: '名前は必須です' },
      { status: 400 }
    );
  }

  const newCompetition = {
    id: String(Date.now()),
    name: body.name,
    status: 'upcoming',
  };

  competitions.push(newCompetition);

  return NextResponse.json(newCompetition, { status: 201 });
}
```

## フロントエンドからの呼び出し

```typescript
// 大会一覧を取得
const response = await fetch('/api/competitions');
const competitions = await response.json();

// 新しい大会を作成
const response = await fetch('/api/competitions', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ name: '春季大会' }),
});
const newCompetition = await response.json();
```

## この章のまとめ

- API Routes は `app/api/` フォルダに作成する
- 関数名（GET, POST, PUT, DELETE）でHTTPメソッドを指定
- `request.json()` でリクエストボディを取得
- `NextResponse.json()` でレスポンスを返す

## 確認してみよう

- [ ] API Routes の作り方を理解した
- [ ] HTTPメソッドごとの関数を書ける
- [ ] リクエストボディの取得方法がわかる
- [ ] レスポンスの返し方がわかる
