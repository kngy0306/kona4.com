---
title: "React + Rails プロジェクトをDockerで構築する"
subtitle: ""
date: 2022-10-07T16:15:14+09:00
draft: false
tags: ["react", "ruby"]
bigimg: [{src: "/img/2022/react-rails.jpg", desc: ""}]
image: "/img/2022/react-rails.jpg"
description: "React(Vite) + Rails プロジェクトをDockerで構築する際のメモ"
---

## 使用技術

フロントエンド

- React (18.x)
- TypeScript
- Vite

バックエンド

- Ruby on Rails (6.x)
- Ruby (2.7)
- Postgres

<!--more-->

# 最終的なフォルダ構成

```

├── api
│   ├── Dockerfile
│   └── Railsのファイル群
├── front
│    ├── Dockerfile
│    └── Reactのファイル群
└── docker-compose.yml

```

# docker-compose.yml

コンテナを複数管理するため、`Docker Compose`を利用します。`docker-compose.yml`ファイルを作成し、以下のように記載します。

`db`、`environment`に記載している`POSTGRES_USER`などは任意です。後ほどRailsでDB接続の設定を行う際に参照します。

```yml

version: "3.8"

services:
  front:
    container_name: sample_front
    build:
      context: ./front
      dockerfile: Dockerfile
    ports:
      - 5173:5173
    volumes:
      - ./front:/app:cached
    environment:
      - HOST=0.0.0.0
      - CHOKIDAR_USEPOLLING=true
    command: yarn dev --host 0.0.0.0
    stdin_open: true
    tty: true
  api:
    container_name: sample_api
    build:
      context: ./api
      dockerfile: Dockerfile
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - ./api:/app
    ports:
      - "3000:3000"
    tty: true
    depends_on:
      - db
  db:
    container_name: sample_db
    image: postgres:14-alpine
    volumes:
      - ./sample_db:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: sample
    ports:
      - "5432:5432"  

```

# apiフォルダの構成

apiフォルダを作成し、その後いくつかファイルを作成します。

## Dockerfile

apiフォルダ内に`Dockerfile`を作成し、以下のように記載します。

```sh

FROM ruby:2.7

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update -qq && apt-get install -y nodejs postgresql-client yarn
WORKDIR /app
COPY Gemfile /app/Gemfile
RUN bundle install

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 3000

CMD ["rails", "server", "-b", "0.0.0.0"]

```

## Gemfile

apiフォルダ内に`Gemfile`を作成し、以下のように記載します。

```rb

source "https://rubygems.org"
git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

gem 'rails', '~> 6.0'

```

## entrypoint.sh

apiフォルダ内に`entrypoint.sh`を作成し、以下のように記載します。

```sh

#!/bin/bash
set -e

# Remove a potentially pre-existing server.pid for Rails.
rm -f /myapp/tmp/pids/server.pid

# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"

```

# frontフォルダの構成

frontフォルダを作成し、1つファイルを作成します。

## Dockerfile

apiフォルダ内に`Dockerfile`を作成し、以下のように記載します。

```sh

FROM node:16.14.2-alpine3.14

RUN apk update

ENV TZ Asia/Tokyo
ENV PATH $HOME/.yarn/bin:$HOME/.config/yarn/global/node_modules/.bin:$PATH

WORKDIR /app

```

## 現状のフォルダ構成

```sh

├── api
│   ├── Dockerfile
│   ├── Gemfile
│   └── entrypoint.sh
├── front
│   └── Dockerfile
└── docker-compose.yml

```

# コンテナのビルド

```sh

docker compose build

```

# Railsプロジェクトの構築

RailsはAPI専用として構築します。

```sh

docker compose run --rm api bundle exec rails new . --api -d postgresql

```

Gemfileがすでにフォルダにあるため、以下のような文言が出ると思いますが、`Y`を入力して上書きします。

```sh

conflict  Gemfile
Overwrite /app/Gemfile? (enter "h" for help) [Ynaqdhm] ← Yを入力

```

パッケージインストール

```sh

docker compose run --rm api bundle install

```

# Reactプロジェクトの構築

Viteを使用してReactプロジェクトを構築します。一度`tmp`ファイル内に作成し、そのあとフォルダの中身を移動しています。  
そのような手順にしている理由は、Viteでプロジェクトを構築する際、フォルダが空でないとエラーになってしまうのを回避するためです。

```sh

docker compose run --rm front yarn create vite tmp --template react-ts

docker compose run --rm front sh -c "mv ./tmp/* ./ && rm -r ./tmp"

docker compose run --rm front yarn install

```

# DB設定~コンテナ立ち上げ

## DB設定

`api/config/database.yml`をdocker-compose.ymlに記載したように修正します。

```yml

# PostgreSQL. Versions 9.3 and up are supported.
#
# Install the pg driver:
#   gem install pg
# On macOS with Homebrew:
#   gem install pg -- --with-pg-config=/usr/local/bin/pg_config
# On macOS with MacPorts:
#   gem install pg -- --with-pg-config=/opt/local/lib/postgresql84/bin/pg_config
# On Windows:
#   gem install pg
#       Choose the win32 build.
#       Install PostgreSQL and put its /bin directory on your path.
#
# Configure Using Gemfile
# gem 'pg'
#
default: &default
  adapter: postgresql
  encoding: unicode
  # For details on connection pooling, see Rails configuration guide
  # https://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  port: 5432
  host: db
  username: user
  password: password

development:
  <<: *default
  database: sample_development

  # The specified database role being used to connect to postgres.
  # To create additional roles in postgres see `$ createuser --help`.
  # When left blank, postgres will use the default role. This is
  # the same name as the operating system user running Rails.
  #username: app

  # The password associated with the postgres role (username).
  #password:

  # Connect on a TCP socket. Omitted by default since the client uses a
  # domain socket that doesn't need configuration. Windows does not have
  # domain sockets, so uncomment these lines.
  #host: localhost

  # The TCP port the server listens on. Defaults to 5432.
  # If your server runs on a different port number, change accordingly.
  #port: 5432

  # Schema search path. The server defaults to $user,public
  #schema_search_path: myapp,sharedapp,public

  # Minimum log levels, in increasing order:
  #   debug5, debug4, debug3, debug2, debug1,
  #   log, notice, warning, error, fatal, and panic
  # Defaults to warning.
  #min_messages: notice

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: sample_test

# As with config/credentials.yml, you never want to store sensitive information,
# like your database password, in your source code. If your source code is
# ever seen by anyone, they now have access to your database.
#
# Instead, provide the password or a full connection URL as an environment
# variable when you boot the app. For example:
#
#   DATABASE_URL="postgres://myuser:mypass@localhost/somedatabase"
#
# If the connection URL is provided in the special DATABASE_URL environment
# variable, Rails will automatically merge its configuration values on top of
# the values provided in this file. Alternatively, you can specify a connection
# URL environment variable explicitly:
#
#   production:
#     url: <%= ENV['MY_APP_DATABASE_URL'] %>
#
# Read https://guides.rubyonrails.org/configuring.html#configuring-a-database
# for a full overview on how database connection configuration can be specified.
#
production:
  <<: *default
  database: sample_production
  username: <%= ENV["DATABASE_USERNAME"] %>
  password: <%= ENV["DATABASE_PASSWORD"] %>
  host: <%= ENV["DATABASE_HOST"] %>

```

## コンテナの再ビルド

```sh

docker compose build

```

## DB作成

```sh

docker compose run --rm api rails db:create

```

## コンテナ起動

```sh

docker compose up

```

以下のようにログが出力されたら、  

```sh

sample_front  |   VITE v3.1.6  ready in 2996 ms
sample_front  |
sample_front  |   ➜  Local:   http://localhost:5173/
sample_front  |   ➜  Network: http://172.28.0.3:5173/
sample_api    | => Booting Puma
sample_api    | => Rails 6.1.7 application starting in development
sample_api    | => Run `bin/rails server --help` for more startup options
sample_api    | Puma starting in single mode...
sample_api    | * Puma version: 5.6.5 (ruby 2.7.6-p219) ("Birdie's Version")
sample_api    | *  Min threads: 5
sample_api    | *  Max threads: 5
sample_api    | *  Environment: development
sample_api    | *          PID: 1
sample_api    | * Listening on http://0.0.0.0:3000
sample_api    | Use Ctrl-C to stop

```

[http://localhost:5173/](http://localhost:5173/) と [http://localhost:3000/](http://localhost:3000/)

へアクセスします。  
それぞれViteの画面、Railsの画面が表示されていれば完了です。

# 参考文献

Rails による API 専用アプリケーション  
https://railsguides.jp/api_app.html

Next.js + Ruby on Rails(APIモード) on Docker 構築手順  
https://zenn.dev/taku1115/articles/6c9fa97ab37e38

Vite + React x tailwindcss 高速セットアップ  
https://zenn.dev/ryuu/scraps/8cf05fab5a52bb

Docker+Viteでローカル環境を汚さないJSフレームワーク  
https://zenn.dev/ayuu/articles/84b482c37bea9a
