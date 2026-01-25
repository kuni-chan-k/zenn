---
title: "HTTP通信の基礎"
---

## この章でやること

- HTTP通信の基本を理解する
- HTTPメソッド（GET, POST, PUT, DELETE）を知る
- ステータスコードを理解する

## HTTPとは

HTTP（HyperText Transfer Protocol）は、Webでデータをやりとりするためのルールです。

ブラウザとサーバーは、このルールに従って通信します。

## HTTPリクエスト

クライアントからサーバーへの「お願い」です。

```
GET /api/competitions HTTP/1.1
Host: example.com
Content-Type: application/json
```

リクエストには以下の情報が含まれます。

| 要素 | 説明 | 例 |
|-----|------|-----|
| メソッド | 何をしたいか | GET, POST, PUT, DELETE |
| URL | どこに | /api/competitions |
| ヘッダー | 追加情報 | Content-Type, Authorization |
| ボディ | 送るデータ | { "name": "春季大会" } |

## HTTPメソッド

「何をしたいか」を表します。CRUDと対応しています。

| メソッド | 意味 | CRUD | 使用例 |
|---------|------|------|--------|
| **GET** | 取得する | Read | 大会一覧を取得 |
| **POST** | 作成する | Create | 新しい大会を作成 |
| **PUT** | 更新する | Update | 大会情報を更新 |
| **DELETE** | 削除する | Delete | 大会を削除 |

### 使用例

```typescript
// GET: 大会一覧を取得
fetch('/api/competitions')

// POST: 新しい大会を作成
fetch('/api/competitions', {
  method: 'POST',
  body: JSON.stringify({ name: '春季大会' })
})

// PUT: 大会を更新
fetch('/api/competitions/1', {
  method: 'PUT',
  body: JSON.stringify({ name: '春季大会2024' })
})

// DELETE: 大会を削除
fetch('/api/competitions/1', {
  method: 'DELETE'
})
```

## HTTPレスポンス

サーバーからクライアントへの「返事」です。

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 1,
  "name": "春季大会"
}
```

## ステータスコード

レスポンスの結果を3桁の数字で表します。

| コード | 意味 | 説明 |
|-------|------|------|
| **200** | OK | 成功 |
| **201** | Created | 作成成功 |
| **400** | Bad Request | リクエストが不正 |
| **401** | Unauthorized | 認証が必要 |
| **403** | Forbidden | 権限がない |
| **404** | Not Found | 見つからない |
| **500** | Internal Server Error | サーバーエラー |

### 覚え方

- **2xx**: 成功
- **4xx**: クライアントのエラー（リクエストが悪い）
- **5xx**: サーバーのエラー（サーバーが悪い）

## JSON形式

データは**JSON**（JavaScript Object Notation）形式でやりとりします。

```json
{
  "id": 1,
  "name": "春季フットサルリーグ",
  "status": "ongoing",
  "teams": [
    { "id": 1, "name": "FCレッドスター" },
    { "id": 2, "name": "ブルーウィングス" }
  ]
}
```

JavaScriptのオブジェクトとほぼ同じ形式なので、扱いやすいのが特徴です。

## ブラウザで確認してみよう

ブラウザの開発者ツール（F12）で、HTTP通信を確認できます。

1. 開発者ツールを開く（F12 または右クリック→検証）
2. 「Network」タブを選択
3. ページを操作する
4. リクエストとレスポンスを確認

実際に見てみると理解が深まります。

## この章のまとめ

- HTTPはWebでデータをやりとりするルール
- HTTPメソッド（GET, POST, PUT, DELETE）で何をしたいかを表す
- ステータスコードでレスポンスの結果を表す
- データはJSON形式でやりとりする

## 確認してみよう

- [ ] HTTPメソッドの4種類を説明できる
- [ ] ステータスコードの意味がわかる
- [ ] JSON形式を理解した
