---
title: "Laravelで使用するMySQLをDockerで用意する【M1 Mac】"
subtitle: ""
date: 2022-02-12T09:26:16+09:00
draft: false
tags: ["docker"]
bigimg: [{src: "/img/2022/02-12-thum.jpg", desc: "laravel-docker"}]
image: "/img/2022/02-12-thum.jpg"
description: "ローカルでLaravelを使用し、データベースだけをDockerで用意する方法です。"
---

ローカルでLaravelのプロジェクトを作成し、データベースだけをDockerで用意する方法をまとめました。

<!--more-->

## 環境

PC：Apple M1 Pro

作成するMySQLバージョン：8.0

## Laravelのプロジェクトを作成する

例: SampleApp というプロジェクト名でバージョン8.xを使用する場合

```sh

composer create-project laravel/laravel --prefer-dist SampleApp "8.x"

cd SampleApp

```

この先実行するコマンドは基本的にSampleAppで実行します。

## データベースのデータ用フォルダ、設定ファイルを作成する

Laravelプロジェクト内に`docker`というフォルダを作成し、その中にDockerコンテナ内で作成する、MySQLのデータを保存する用のフォルダと、MySQLに追記する設定ファイルを記載します。

```sh

mkdir docker

cd docker

mkdir db 

touch mysql.cnf

```

mysql.cnfを以下のように編集します。文字設定、照合順序を指定しています。

```sh

[mysqld]
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci

[mysql]
default-character-set = utf8mb4
 
[client]
default-character-set = utf8mb4

```

文字設定をutf8にしない場合、日本語が以下のように文字化けしてしまします。

```sql

mysql> select * from customers;
+----+---------+---------------------+---------------------+
| id | name    | created_at          | updated_at          |
+----+---------+---------------------+---------------------+
|  1 | ???? ?? | 2022-02-12 08:22:22 | 2022-02-12 08:22:22 |
|  2 | ???? ?? | 2022-02-12 08:22:22 | 2022-02-12 08:22:22 |
+----+---------+---------------------+---------------------+
2 rows in set (0.01 sec)

```

dbフォルダは、空のままにしておきます。

## docker-compose.ymlを作成

docker-composeを用いてMySQLコンテナを作成します。  
Laravelプロジェクト内で、docker-compose.ymlを作成し、編集します。

```sh

touch docker-compose.yml

```

```yml

version: '3.8'

services:
  db:
    platform: linux/x86_64 # M1チップはこの記載が必要
    image: mysql:8.0
    environment:
        MYSQL_ROOT_PASSWORD: root
        MYSQL_DATABASE: sample
        MYSQL_USER: user
        MYSQL_PASSWORD: pass
        TZ: 'Asia/Tokyo'
    volumes:
        - ./docker/db:/var/lib/mysql
        - ./docker/mysql.cnf:/etc/mysql/conf.d/mysql.cnf
    ports:
        - 3306:3306

```

## コンテナを立ち上げ、MySQLにアクセスする

```sh

# コンテナ立ち上げ
docker-compose up -d

# コンテナへ入る
docker-compose exec db bash


# テーブルを指定してMySQLへアクセス
mysql -u user -ppass sample

mysql>

```

MySQLへアクセスできたら、以下コマンドで先程設定した文字設定が確認できます。

```mysql

mysql> SHOW VARIABLES LIKE "chara%";
+--------------------------+--------------------------------+
| Variable_name            | Value                          |
+--------------------------+--------------------------------+
| character_set_client     | utf8mb4                        |
| character_set_connection | utf8mb4                        |
| character_set_database   | utf8mb4                        |
| character_set_filesystem | binary                         |
| character_set_results    | utf8mb4                        |
| character_set_server     | utf8mb4                        |
| character_set_system     | utf8mb3                        |
| character_sets_dir       | /usr/share/mysql-8.0/charsets/ |
+--------------------------+--------------------------------+
8 rows in set (0.06 sec)

```

MySQL、コンテナから抜けるにはexitを使用します。

```sh

mysql> exit
Bye
root@e40224abf02:/# exit
exit

```

## Laravelからデータベースへアクセスする

MySQLのコンテナが用意できたので、Laravelからコンテナへアクセスするように設定していきます。

データベースへ接続するために、`.env`ファイルの以下の部分を編集します。

```env

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=sample
DB_USERNAME=user
DB_PASSWORD=pass

```

.envファイルを変更したら、以下コマンドを実行して反映させます。

```sh

php artisan cache:clear

```

マイグレーションを実行します。

```sh

php artisan migrate

Migration table not found.
Migration table created successfully.
Migrating: 2019_12_14_000001_create_personal_access_tokens_table
Migrated:  2019_12_14_000001_create_personal_access_tokens_table (188.85ms)

```

問題なく実行できていたら完了です！

## フォルダ構成

最終的なフォルダ構成は以下のようになっています。

```sh

├── app

〜 # Laravelのフォルダ郡

├── webpack.mix.js
├── docker # ここから下が、今回独自に作成したフォルダ・ファイル
│   ├── db
│   └── mysql.cnf
└── docker-compose.yml

```