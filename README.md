# emdash-playground

[EmDash](https://emdashcms.com/) を Cloudflare Workers の無料プラン（カード登録なし）で試すためのリポジトリ。

- サイト: https://emdash-playground.soe-soe-soeda.workers.dev
- 管理画面: https://emdash-playground.soe-soe-soeda.workers.dev/_emdash/admin/

## 構成

`site/` は `pnpm create emdash@1.4.0 site --template cloudflare:blog` で作成したもの。

- Workers + D1（DB）+ KV（セッション）だけを使う
- R2 は無料でもカード登録が必要なため外している。そのためメディアのアップロードは使えない
- ログインはパスキー。他のユーザーは管理画面の Users → Invite User で招待リンクを発行して渡す

## ローカル開発

Node.js 22.23.4 以上が必要。

```sh
cd site
pnpm install
pnpm dev   # http://localhost:4321/_emdash/admin/
```

## デプロイ

```sh
cd site
pnpm wrangler login
pnpm build && pnpm wrangler deploy
```

初回は `EMDASH_ENCRYPTION_KEY` を secret として登録する（値は `site/.env`）。

```sh
pnpm wrangler secret put EMDASH_ENCRYPTION_KEY
```

サイトが 503（`Database migrations are required`）を返す場合は、手元からマイグレーションを適用する。
無料プランの CPU 制限で Worker 内の自動マイグレーションが途中で止まることがある。

```sh
export CLOUDFLARE_API_TOKEN=...   # wrangler login の OAuth トークンでも可
pnpm emdash migrate --from-config --account-id <ACCOUNT_ID> --d1 emdash-playground --status
pnpm emdash migrate --from-config --account-id <ACCOUNT_ID> --d1 emdash-playground \
  --expected-target-fingerprint=<status に出る値> [--take-over-lock=<止まったロックの id>]
```
