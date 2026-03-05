# CLAUDE.md

このファイルはリポジトリ内で作業する Claude Code (claude.ai/code) へのガイダンスを提供します。

## 概要

MultiBook は2つの git submodule を管理する Docker Compose プロジェクトです：
- `src_backend/` — Laravel (PHP) API バックエンド
- `src_frontend/` — React + TypeScript SPA フロントエンド

## アーキテクチャ

Nginx がポート 8000 のエントリーポイントとして機能します：
- 静的ファイルおよび React SPA は直接配信
- `/api/*` へのリクエストはポート 3000 の Laravel にリバースプロキシ
- `*.php` リクエストはポート 9000 の PHP-FPM に fastcgi 経由で転送

React フロントエンドはバックエンドとの通信をすべて `/api/` プレフィックス経由で行います。認証は Google OAuth (`/api/auth/google`) で処理されます。

## コマンド

### セットアップ（初回のみ）
```bash
git submodule update --init --recursive

# Backend
cd src_backend && composer install && chmod -R 777 storage && chmod -R 777 bootstrap/cache

# Frontend
cd src_frontend && yarn install && npm run build
```

### Docker
```bash
docker compose up -d                                                          # 起動
docker compose down                                                           # 停止
docker compose down && docker compose up -d --build --remove-orphans         # フルリビルド
```

### テスト
```bash
# Laravel テスト全件実行
docker compose exec -w=/var/www/html/backend php php artisan test --testdox

# テストファイル指定
docker compose exec -w=/var/www/html/backend php php artisan test tests/Feature/ImageTest.php --testdox

# テストメソッド指定
docker compose exec -w=/var/www/html/backend php php artisan test tests/Feature/ImageTest.php --filter update --testdox
```

### ログ
```bash
docker compose logs -f nginx   # nginx アクセスログ
docker compose logs -f php     # PHP/Laravel エラーログ（500エラー等）
```

### Swagger API ドキュメント
```bash
# src_backend/app/Http/ の OpenAPI PHP アトリビュートから openapi.yml を再生成
cd $(git rev-parse --show-toplevel)/src_backend && ./vendor/bin/openapi app/Http/ -o ../api-docs/openapi.yml
```

## サービス URL

| サービス      | URL                      |
|--------------|--------------------------|
| React アプリ  | http://localhost:8000/   |
| phpMyAdmin   | http://localhost:8001/   |
| Swagger UI   | http://localhost:8002/   |

## ブランチ・PR 規約

[CONTRIBUTING.md](./CONTRIBUTING.md) より：
- 作業開始前に必ず Issue を作成する
- ブランチ命名規則：`{type}/{issue番号}-{説明}` （例：`feature/101-login-page`）
- タイプ一覧：`feature`, `bugfix`, `hotfix`, `refactor`, `docs`
- PR タイトルは Issue タイトルと同じにし、本文に `Close #123` を含める
