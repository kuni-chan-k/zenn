---
title: "デプロイと次のステップ"
---

## この章でやること

- Vercel へのデプロイ
- データベースの本番環境設定
- 次のステップの紹介

## 完成おめでとうございます！ 🎉

2部構成の本を完走しました！

**Part 1（基礎編）で学んだこと：**
- Next.js + TypeScript + Tailwind CSS
- コンポーネント設計
- CRUD 操作
- 順位表の計算ロジック

**Part 2（発展編）で学んだこと：**
- Prisma + PostgreSQL
- JWT 認証
- API Routes
- 権限管理

## Vercel へのデプロイ

### 1. GitHub にプッシュ

```bash
git remote add origin https://github.com/your-username/match-manager.git
git push -u origin main
```

### 2. Vercel にデプロイ

1. [Vercel](https://vercel.com) にアクセス
2. GitHub アカウントでログイン
3. 「New Project」をクリック
4. リポジトリを選択してインポート

### 3. 環境変数の設定

Vercel のプロジェクト設定で、以下の環境変数を追加：

```
DATABASE_URL=postgresql://...（本番用DB）
JWT_SECRET=your-production-secret
```

### 4. データベースの準備

本番用のデータベースには以下のサービスが使えます：

| サービス | 特徴 |
|---------|------|
| Vercel Postgres | Vercel との統合が簡単 |
| Supabase | 無料枠が充実 |
| Neon | サーバーレス対応 |
| PlanetScale | MySQL 互換 |

### Vercel Postgres の例

1. Vercel ダッシュボードで「Storage」→「Create Database」
2. 「Postgres」を選択
3. 自動的に `DATABASE_URL` が設定される

## 本番環境の注意点

### セキュリティ

- [ ] `JWT_SECRET` を強力なランダム文字列に変更
- [ ] HTTPS を有効化（Vercel なら自動）
- [ ] 環境変数を適切に管理

### パフォーマンス

- [ ] 画像の最適化（Next.js Image コンポーネント）
- [ ] 適切なキャッシュ設定
- [ ] データベースのインデックス

## 次のステップ

このアプリをさらに発展させるアイデアを紹介します。

### 機能追加

1. **リアルタイム更新**
   - WebSocket または Server-Sent Events
   - 試合結果のリアルタイム反映

2. **通知機能**
   - メール通知（Resend, SendGrid）
   - プッシュ通知（Web Push API）

3. **選手管理**
   - 選手の登録・プロフィール
   - 個人成績の追跡

4. **統計・分析**
   - チーム成績のグラフ表示
   - 対戦相手との相性分析

### 技術的な改善

1. **テスト**
   - Jest + React Testing Library
   - Playwright で E2E テスト

2. **CI/CD**
   - GitHub Actions でテスト自動実行
   - プレビューデプロイ

3. **監視**
   - エラートラッキング（Sentry）
   - パフォーマンス監視

## match-flow リポジトリ

この本の発展版として、実際に開発中のプロジェクトがあります：

```
/Users/kunimitsu/workspace/サービス開発/match-flow/
```

以下の機能が実装されています：

- ✅ トーナメント戦/総当り戦
- ✅ 試合マッピング編集
- ✅ 第2回戦以降の自動生成
- ✅ シードプレイヤー設定
- ✅ 主催者向けダッシュボード

コードを参考にして、さらに発展させてみてください。

## 最後に

この本を通じて、React/Next.js で実用的なアプリを作る力が身についたはずです。

学んだことを活かして、ぜひ自分のプロジェクトに挑戦してください：

- 卓球大会管理システム
- バスケットボールリーグ管理
- eスポーツ大会管理
- 社内イベント管理

**React は学べば学ぶほど楽しくなります。これからも開発を楽しんでください！**

---

## 参考リンク

- [Next.js 公式ドキュメント](https://nextjs.org/docs)
- [Prisma 公式ドキュメント](https://www.prisma.io/docs)
- [Tailwind CSS 公式ドキュメント](https://tailwindcss.com/docs)
- [Vercel 公式ドキュメント](https://vercel.com/docs)
