# MultiBook's docker compose

## コンテナ構成図

```mermaid
graph TD
    nginx["nginx<br>(React + Laravel)"]
    php["php"]
    mysql["mysql"]
    phpmyadmin["phpmyadmin"]
    swagger["swagger"]

    nginx -->|uses| php
    php -->|uses| mysql
    phpmyadmin -->|uses| mysql
```

## サービス関連図

```mermaid
graph TD
  %% --- Host ------------------------------------------------------------------
  subgraph クライアント
    Host[ポートフォワード]
  end

  %% --- Containers & Services -------------------------------------------------
  subgraph nginxコンテナ
    style nginxコンテナ fill:#f5f5f5,stroke:#999,stroke-width:1px
    Nginx[Nginx<br/>80]
    Laravel[Laravel<br/>3000]
  end

  subgraph phpコンテナ
    style phpコンテナ fill:#f5f5f5,stroke:#999,stroke-width:1px
    PHPFPM[php-fpm<br/>9000]
  end

  subgraph mysqlコンテナ
    style mysqlコンテナ fill:#f5f5f5,stroke:#999,stroke-width:1px
    MySQL[MySQL Server<br/>3306]
  end

  subgraph swaggerコンテナ
    style swaggerコンテナ fill:#f5f5f5,stroke:#999,stroke-width:1px
    Swagger[Swagger<br/>8080]
  end

  subgraph phpmyadminコンテナ
    style phpmyadminコンテナ fill:#f5f5f5,stroke:#999,stroke-width:1px
    PMA[phpMyAdmin<br/>80]
  end

  %% --- Port Forwarding -------------------------------------------------------
  Host -- 8000 --> Nginx
  Host -- 8001 --> PMA
  Host -- 8002 --> Swagger

  %% --- Internal Routing ------------------------------------------------------
  Nginx -- "/api/* (reverse proxy)" --> Laravel
  Nginx -- "*.php$ (fastcgi_pass)" --> PHPFPM

  %% --- Typical DB Access (optional) -----------------------------------------
  PHPFPM
   -- "DB 接続" --> MySQL
  PMA -- "DB 接続" --> MySQL
```

## コマンド
### 起動
```
docker compose up -d
```
### 停止
```
docker compose down
```
### 再起動（ビルドとか色々リセット込み）
```
docker compose down && docker compose up -d --build --remove-orphans
```
### コンテナにログイン
(例)
```
docker compose exec nginx /bin/bash
```
### コンテナでコマンド実行
(例)
```
docker compose exec nginx ls -l /var/www/html/backend/public
```
### コンテナ起動時のログ調査
```
docker compose down && docker compose up -d --build --remove-orphans && docker compose logs -f
```
### Laravel単体テスト実行
```
docker compose exec -w=/var/www/html/backend php php artisan test --testdox
```
### Laravel単体テスト実行(ファイル指定)
```
docker compose exec -w=/var/www/html/backend php php artisan test tests/Feature/ImageTest.php --testdox
```
### Laravel単体テスト実行(ファイル、メソッド指定)
```
docker compose exec -w=/var/www/html/backend php php artisan test tests/Feature/ImageTest.php --filter update --testdox
```

### Swagger yamlファイル作成
```
cd $(git rev-parse --show-toplevel)/src_backend && ./vendor/bin/openapi app/Http/ -o ../api-docs/openapi.yml
```

## URL
### React
http://localhost:8000/
### phpMyAdmin
http://localhost:8001/
### Swagger
http://localhost:8002/