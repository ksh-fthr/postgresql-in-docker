# このプロジェクトについて

Docker + PostgreSQL の環境構築を試したものです。 今後知見が増えたりアイディアが出たら更新することもあります。

# 環境について

以下の環境で実行・確認しています。

| 環境                                                         | バージョン              | 備考                         |
| ------------------------------------------------------------ | ----------------------- | ---------------------------- |
| macOS Catalina                                               | v10.15.7                |                              |
| [Docker Desktop for Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac/) | v3.0.3                  |                              |
| Docker                                                       | v20.10.0, build 7287ab3 | `$ docker --version`         |
| Docker Compose                                               | v1.27.4, build 40524192 | `$ docker-compose --version` |
| [Docker PostgreSQL](https://hub.docker.com/_/postgres)       | v13.1                   | Dockerfile で指定            |

# 構成

```bash
./
├── README.md
├── docker-compose.yml
└── postgresql
    ├── Dockerfile
    └── init
        └── initialize.sql
```

# ブランチについて
環境構築のためのプロジェクトなので、ブランチを切って修正することは考えていません。 `master` ブランチのみで更新を行う予定です。

# コンテナに対する操作
## 起動
次のコマンドを実行することで Docker 上に PostgreSQL が起動します。

```bash
$ docker-compose up -d --build
```

`-d` はバックグラウンドで動かすオプションです。 フォアグラウンドで動かしたい場合は `-d` をつけずに実行してください。

## 停止
### サービスの停止
サービスは停止するが Docker コンテナは削除したくない場合は下記コマンドを実行してください。

```bash
$ docker-compose stop
```

### コンテナの停止
サービスの停止とサービスを提供するコンテナの削除、それからネットワークも削除したい場合は下記コマンドを実行してください。

```bash
$ docker-compose down
```

# 作成される DB
`docker-compose up -d --build` を実行することで `./postgresql/init/initialize.sql` が実行され、`company` DB が作成されます。
DB の情報は以下のとおりです。

| DB名 | テーブル | ユーザ名 | パスワード |
| ---- | -------- | -------- | ---------- |
| company | employee | postgres | pgadmin |

# DB の確認
Docker コンテナに入って DB 操作するときは以下の手順で行えます。

## ホスト側での操作
### コンテナの確認
次のコマンドを実行して `CONTAINER ID` を確認します。

```bash
$ docker ps
CONTAINER ID   IMAGE                     COMMAND                  CREATED         STATUS         PORTS                    NAMES
d37e0fc92ac3   postgresql-in-docker_db   "docker-entrypoint.s…"   7 seconds ago   Up 6 seconds   0.0.0.0:5432->5432/tcp   docker-postgresql-work
```

( ここで取得した `CONTAINER ID` はあくまで上記コマンド実行時に確認できたものです。実際は実行する毎に異なる値になります )

### コンテナに入る
次のコマンドで `CONTAINER ID` を指定してコンテナに入ります。

```bash
$ docker exec -it d37e0fc92ac3 /bin/bash
```

以降は コンテナ上での作業になります。

## コンテナ上での操作
### postgres ユーザにスイッチする
次のコマンドで `postgres` ユーザにスイッチします。

```bash
root@postgres:/# su - postgres
```

### psql に入る
次のコマンドで `psql` に入ります。

```bash
postgres@postgres:~$ psql
psql (13.1 (Debian 13.1-1.pgdg100+1))
Type "help" for help.
```

以降は psql 上での作業になります。

## psql 上での操作
### 作成済みのデータベース一覧
```psql
postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 company   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(4 rows)
```

### company DB へ接続

```psql
postgres=# \c company
You are now connected to database "company" as user "postgres".
```

### テーブル一覧
```psql
company=# \dt;
          List of relations
 Schema |   Name   | Type  |  Owner   
--------+----------+-------+----------
 public | employee | table | postgres
(1 row)
```

### データ検索
```psql
company=# select * from employee;
 id |   name   |      tel      
----+----------+---------------
  1 | hogehoge | 000-0000-0000
  2 | piyopiyo | 111-1111-1111
  3 | fugafuga | 222-2222-2222
(3 rows)
```

# GUI クライアントでの確認
次の GUI クライアントで接続を確認しております。

## アプリ
- [PSequel](http://www.psequel.com/)

## 接続情報
設定する接続情報は以下のとおりです。

| HOST      | User     | Password | Database |
| --------- | -------- | -------- | -------- |
| 127.0.0.1 | postgres | pgadmin  | company  |

