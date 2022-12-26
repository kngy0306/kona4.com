---
title: "CORSについて"
subtitle: ""
date: 2022-12-26T15:08:09+09:00
draft: false
tags: [""]
# bigimg: [{src: "/img/2022/", desc: ""}]
# image: "/img/2022/"
description: "CORSの概要を背景を含めてまとめた記事です"
---

ウェブアプリケーションが発展し、同一オリジンポリシーの制限を超えてデータのやり取りを行いたいというニーズが増えてきました。
その際サイトを超えたデータのやり取りに関する仕様として策定されたのが**CORS**です。

<!--more-->

## 同一オリジンポリシー

同一オリジンではない、他のサイトへのアクセスを制限するもので、同一オリジンである条件は以下を満たすもの。

- ホストが一致している
- スキームが一致している
- ポート番号が一致している

## 異なるオリジンからの読み出しを許可する

異なるオリジンからの読み出しを許可するためには、`Access-Control-Allow-Origin`というヘッダーを**情報の提供元**がHTTPレスポンスヘッダに付与する必要があります。

## プリフライトリクエスト

読み出し（GETメソッド）などの**シンプルなリクエスト**の場合には上記のように`Access-Control-Allow-Origin`を付与すれば問題ありませんでした。
しかし**シンプルなリクエスト**ではない場合には、ブラウザは異なるオリジンに対し、プリフライトリクエストを送信します。

**シンプルなリクエスト**である条件は以下を満たすもの

**メソッド**
- GET
- HEAD
- POST

**リクエストヘッダ**
- Accept
- Accept-Language
- Content-Language
- Content-Type

**Content-Type**
- application/x-www-form-urlencoded
- multipart/form-data
- text/plain

上記のリクエスト以外は**プリフライトリクエスト**というHTTPリクエストを送ります。

## 認証を含むリクエスト

デフォルトでは異なるオリジンに対するリクエスト内に、HTTP認証やクッキーなどの認証に用いられるリクエストヘッダは送信されません。認証を含むリクエストを行うには以下を満たす必要があります。

- リクエスト側は、XMLHttpRequestの`withCredentials`プロパティをtrueにする。
- レスポンス側は、`Access-Control-Allow-Credentials`レスポンスヘッダーをtrueにする。

**参考ページ**

- [XMLHttpRequest.withCredentials](https://developer.mozilla.org/ja/docs/Web/API/XMLHttpRequest/withCredentials)
- [Access-Control-Allow-Credentials](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials)

## 参考文献

[体系的に学ぶ 安全なWebアプリケーションの作り方 第2版](https://www.amazon.co.jp/%E4%BD%93%E7%B3%BB%E7%9A%84%E3%81%AB%E5%AD%A6%E3%81%B6-%E5%AE%89%E5%85%A8%E3%81%AAWeb%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%AE%E4%BD%9C%E3%82%8A%E6%96%B9-%E7%AC%AC2%E7%89%88-%E8%84%86%E5%BC%B1%E6%80%A7%E3%81%8C%E7%94%9F%E3%81%BE%E3%82%8C%E3%82%8B%E5%8E%9F%E7%90%86%E3%81%A8%E5%AF%BE%E7%AD%96%E3%81%AE%E5%AE%9F%E8%B7%B5-%E5%BE%B3%E4%B8%B8/dp/4797393165)