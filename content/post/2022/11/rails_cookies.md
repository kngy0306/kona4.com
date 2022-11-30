---
title: "Railsでクッキーを扱う"
subtitle: ""
date: 2022-11-29T22:50:33+09:00
draft: false
tags: ["rails"]
# bigimg: [{src: "/img/2022/", desc: ""}]
# image: "/img/2022/"
description: "Railsでクッキーの保存、削除、オプションを指定する方法についての記事です。"
---

先日 [セキュリティ上重要なクッキー属性を読んでみる](https://zenn.dev/powder/articles/session_rails_20221124) という記事を投稿しました。そこで取り上げたクッキーについて、Railsでの保存、削除、オプションの付与などを試してみました。

<!--more-->

本記事はすべて[Railsドキュメント](https://railsdoc.com/cookie_cache)を参考にしています。

## クッキーに保存

以下のように記載することでブラウザのクッキーに値を保存できます。

```rb

cookies[:sample_cookies] = user.id # user.id => 1

```

{{< rawhtml >}}<img src="/img/2022/2022-11-29-cookie1.jpg" alt="cookieに値の代入" width=100%>{{< /rawhtml >}}

## 値を暗号化して保存

```rb

cookies.signed[:signed_value] = 10

```

{{< rawhtml >}}<img src="/img/2022/2022-11-29-cookie3.jpg" alt="cookieに値の代入" width=100%>{{< /rawhtml >}}

## オプションを付与する

ハッシュを用いてオプションを指定します。指定可能なオプションの一覧は[こちら](https://railsdoc.com/cookie_cache#:~:text=key%3A%20%E3%82%AF%E3%83%83%E3%82%AD%E3%83%BC%E6%83%85%E5%A0%B1%20%7D-,%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3,-%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3)です。

```rb

cookies[:sample_cookies_with_option] = {
  value: user.id,
  path: '/',
  domain: '127.0.0.1',
  expires: 24.hour.from_now,
  secure: false,
  httponly: true
}

```

以下のスクリーンショットのように、オプションで指定した値が反映されています。

{{< rawhtml >}}<img src="/img/2022/2022-11-29-cookie2.jpg" alt="cookieに値の代入" width=100%>{{< /rawhtml >}}

## クッキーの削除

```rb

cookies.delete(:user_id)

```

