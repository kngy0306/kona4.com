---
title: "EC2からPlanetScaleへ接続したときのエラー解消"
subtitle: ""
date: 2022-11-27T16:16:59+09:00
draft: false
tags: ["aws"]
# bigimg: [{src: "/img/2022/", desc: ""}]
# image: "/img/2022/"
description: "EC2で動作している、Laravelを使用したアプリケーションからPlanetScaleへ接続しようとした際に遭遇したエラーと、その解消について書きました。"
---

EC2上でLaravelを使用してアプリケーションを作成したさい、DBに[PlaceScale](https://planetscale.com/)を使用し、Laravelから接続を行いました。そのときに`SQLSTATE[HY000] [2002] No such file or directory`というエラーが出たため、その解消手順を記載しました。

<!--more-->

## 環境

EC2（Amazon Linux）  
Laravel 8.83.8

## DB接続

LaravelからPlanetScaleのDBへ接続するには、PlanetScaleで作成したDBの画面から、**Connect** → **Connect with**タブからLaravelを選択します。  
以下のように`.env`ファイルに記載する内容が表示されるので、これをもとに`.env`ファイルを作成もしくは更新することでDB接続の準備は完了します。

```env

DB_CONNECTION=mysql
DB_HOST=ap-aaaaaa.connect.bbbb.cloud
DB_PORT=3306
DB_DATABASE=sample_db
DB_USERNAME=xxxxxxxxxxx
DB_PASSWORD=************
MYSQL_ATTR_SSL_CA=/etc/ssl/cert.pem

```

## エラー内容

`.env`ファイルを編集後、`php artisan migrate`コマンドを使用してテーブル作成を行おうとしましたが、以下のようにエラーが出力されました。

```sh

SQLSTATE[HY000] [2002] No such file or directory (SQL: select exists(select * from `informations` where `number` = 000001) as `exists`)

```

## 解消手順

EC2でAmazon Linuxを使用している場合、表示されている内容では接続ができません。以下の箇所を変更する必要があります。

```sh

MYSQL_ATTR_SSL_CA=/etc/ssl/cert.pem

```

DB接続の際、PlanetScaleの画面から**Connect**ボタンで表示されたモーダルの下部に以下のように記載があります。

> View the [certificate authority root paths](https://planetscale.com/docs/concepts/secure-connections#ca-root-configuration) for the MYSQL_ATTR_SSL_CA details.

このページへ遷移すると、以下のように記述があります。

> RedHat / Fedora / CentOS / Mageia / Vercel / Netlify
> This path also applies to RedHat or Fedora derivatives like Amazon Linux and Oracle Linux. This is the path to use for applications deployed on Vercel and Netlify.
>
> `/etc/pki/tls/certs/ca-bundle.crt`

これをもとに、さきほどの`.env`ファイルに記載する内容を変更します。

```env

DB_CONNECTION=mysql
DB_HOST=ap-aaaaaa.connect.bbbb.cloud
DB_PORT=3306
DB_DATABASE=sample_db
DB_USERNAME=xxxxxxxxxxx
DB_PASSWORD=************
MYSQL_ATTR_SSL_CA=/etc/pki/tls/certs/ca-bundle.crt

```

以上の内容で`.env`ファイルを作成することでDB接続が行えるようになりました。

自分と同じエラーで進まなくなった方の助けになれば幸いです。
