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

[参考](https://qiita.com/friedaji/items/c1894821a2c49395cfd7)

```yml
volumes:
  db_data:

services:
 db:
   image: postgres:15
   hostname: db
   env_file:
     - ./.env
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
設定ファイル名は**.env**の名前にする必要がある。.db.envとかにすると設定を読み込めない。
```env
DB_HOST=db
DB_NAME=mydb
DB_USER=admin
DB_PASS=password
```

プロジェクト直下でdocker compose upを実行してdbのコンテナを立ち上げる。
> docker compose up

#### tablePlusで接続確認
以下のパラメータを設定
- User : admin
- Password : password
- database : mydb

### コンテナにNode環境を作成

ルートディレクトにDockerfileを作成

pnpmをインストールすること
```Dockerfile
FROM node:18-alpine

RUN apk add g++ make py3-pip

WORKDIR /app/

COPY ./ /app/
RUN apk add --no-cache git
RUN npm install -g npm@9.7.2
RUN npm install -g node-gyp
RUN npm install -g pnpm
RUN npm upgrade --save --legacy-peer-deps
RUN npm install
```

docker起動
> docker compose up --build

appコンテナの中に入って、npm run devを実行する。
> docker exec -it next-prisma-postgre-pnpm-docker sh 
> pnpm run dev

### prismaのセットアップ
> pnpm add prisma ts-node --save-dev

prismaの初期化
> pnpm dlx prsima init

初期化コマンドを実行すると
prismaフォルダと、.envに情報が追加される。
.envのDATABASE_URLは.envの元の情報をもとに書き換えること。

> DATABASE_URL="postgresql://admin:password@db:5432/mydb?schema=public"

docker起動
> docker compose down
> docker compose up --build

DBにデータを書き込む
> npx prisma db push
> pnpm dlx prisma db push

→ schema.prisma をもとに dbを更新する。（migration ファイルを生成せずに)

5555にアクセスできない・・・