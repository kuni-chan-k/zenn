---
title: "環境変数の管理"
---

## この章でやること

- 環境変数の必要性を理解する
- Vercelで環境変数を設定する
- 開発環境と本番環境の切り分け

## 環境変数とは

**環境変数**は、環境ごとに異なる設定値を管理する仕組みです。

```
開発環境: DATABASE_URL = "postgresql://localhost/dev_db"
本番環境: DATABASE_URL = "postgresql://production-server/prod_db"
```

## なぜ環境変数が必要？

以下の情報はコードに直接書いてはいけません。

- データベースの接続情報
- APIキー
- JWT の秘密鍵

理由：
- GitHubにコードをアップロードすると誰でも見られる
- 環境ごとに値が異なる

## .env ファイル

開発環境では、`.env` ファイルに環境変数を書きます。

`.env.local`:
```
DATABASE_URL="postgresql://localhost:5432/myapp"
JWT_SECRET="your-secret-key"
```

:::message alert
`.env.local` は必ず `.gitignore` に追加してください。
GitHubにアップロードしないように！
:::

## Vercelでの環境変数設定

1. Vercelダッシュボードでプロジェクトを選択
2. 「Settings」→「Environment Variables」
3. 変数名と値を入力

## この章のまとめ

- 環境変数で機密情報を管理
- .env ファイルはGitHubにアップロードしない
- Vercelのダッシュボードで本番用の値を設定

## 確認してみよう

- [ ] 環境変数の必要性を理解した
- [ ] .env ファイルを .gitignore に追加した
- [ ] Vercelで環境変数を設定した
