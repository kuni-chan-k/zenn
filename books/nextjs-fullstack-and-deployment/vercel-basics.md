---
title: "Vercelの基礎"
---

## この章でやること

- Vercelとは何かを理解する
- Vercelアカウントを作成する
- GitHubとVercelを連携する

## Vercelとは

**Vercel**は、Next.js を開発している会社が提供するホスティングサービスです。

- Next.js との相性が抜群
- GitHubと連携して自動デプロイ
- 無料枠が充実

## Vercelアカウントの作成

1. [vercel.com](https://vercel.com) にアクセス
2. 「Sign Up」をクリック
3. GitHubアカウントでサインアップ

## GitHubリポジトリの準備

デプロイするには、GitHubにコードをアップロードする必要があります。

```bash
# GitHubリポジトリを作成後
git remote add origin https://github.com/あなたのユーザー名/リポジトリ名.git
git push -u origin main
```

## Vercelでプロジェクトを作成

1. Vercelダッシュボードで「Add New Project」
2. GitHubリポジトリを選択
3. 「Deploy」をクリック

これだけで、アプリが公開されます。

## 自動デプロイ

GitHubにpushするたびに、自動でデプロイされます。

```
GitHub push → Vercel がビルド → 本番環境に反映
```

## この章のまとめ

- VercelはNext.jsに最適なホスティングサービス
- GitHubと連携して自動デプロイ
- 無料枠で十分に使える

## 確認してみよう

- [ ] Vercelアカウントを作成した
- [ ] GitHubと連携した
- [ ] プロジェクトをデプロイできた
