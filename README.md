# docker でnextjs prisma postgresqlの環境を作る

prisma→drizzle にしたいが、まずは最初の走りとしてネットによく上がっているこの組み合わせでやります

- next.js
  - pnpm
- typescript
- prisma→drizzle
  - postgres
- docker

#### 参照
[参考文献](https://zenn.dev/uenishi_com/articles/4607db7c33e7b7)

## 手順
1. Next.jsアプリケーションを作成
1. PostgreSQLのコンテナ作成
1. Prismaのインストール
1. dbコンテナ立ち上げ時にcreate文を実行するようにする
1. makeコマンドでdocker compose 〇〇が立ち上がるようにする

## 導入手順

### pnpmでインストール
> pnpm dlx create-next-app@latest next-prisma-postgre-pnpm-docker

nextjsをlocalhost3000で立ち上げする
> pnpm run dev

### PostgreSQLのコンテナを定義
docker-compose.ymlをプロジェクトルートに作成して記載。
参照元の記事だと **volumes: db_data:**の記載がなくdocker compose upでエラーになる。
```yml
volumes:
  db_data:

services:
 db:
   image: postgres:15
   hostname: db
   env_file:
     - ./.postgre.env
   networks:
     - app_network
   environment:
     - TZ=Asia/Tokyo
     - POSTGRES_DB=$DB_NAME
     - POSTGRES_USER=$DB_USER
     - POSTGRES_PASSWORD=$DB_PASS
     - PGDATA=/var/lib/postgresql/data/pgdata
   ports:
     - 5432:5432
   volumes:
     - db_data:/var/lib/postgresql/data
     - ./sql:/docker-entrypoint-initdb.d

networks:
 app_network:
   driver: bridge
```

設定ファイルもプロジェクト直下に作成する。
```env
DB_HOST=db
DB_NAME=mydb
DB_USER=admin
DB_PASS=password
```

プロジェクト直下でdocker compose upを実行してdbのコンテナを立ち上げる。
> docker compose up

> pnpm add drizzle-orm pg

> pnpm add -D @types/pg drizzle-kit typescript ts-node

tsconfig.jsonを作成

### コーディング
