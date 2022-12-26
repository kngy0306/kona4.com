---
title: "CAPTCHAの概要とreCAPTCHAについて"
subtitle: ""
date: 2022-12-26T08:02:56+09:00
draft: false
tags: [""]
# bigimg: [{src: "/img/2022/", desc: ""}]
# image: "/img/2022/"
description: "CAPTCHAの概要とreCAPTCHAについて"
---

**reCAPTCHA**は、アクセスがコンピュータによる自動応答ではないことを確認する、チャレンジ/レスポンス認証と呼ばれるセキュリティ方式の一種であり、CAPTCHAシステムの1つです。

<!--more-->

## チャレンジ/レスポンス認証

パスワードなどの情報を直接やり取りすることなく、ユーザーの認証を行う。

クライアントが認証を行う際、サーバから毎回ランダムな乱数（チャレンジ）が送信されれる。クライアントはパスワード（知っている情報）を入力し、**チャレンジ**と組み合わせてハッシュ化を行う。これを**レスポンス**としてサーバに送信する。

サーバは手元の認証情報とクライアントから送られてきたレスポンスを照合し、一致するかを確認する。

## CAPTCHAを使用する目的

ユーザ登録を行えるウェブサイトの場合、自動操作によって大量に新規ユーザを作成されてしまうことがあります。CAPTCHAはそのようなコンピュータによる自動操作を防ぐために使用できます。

## reCAPTCHAの種類

reCAPTCHAは2022年12月現在、**reCAPTCHA v2**と**reCAPTCHA v3**が利用できます。**reCAPTCHA v1**は2018年3月に停止されています。

reCAPTCHAのガイドページには以下の種類が記載されています。**reCAPTCHA v1**が停止されている旨も同じページに記載されています。

https://developers.google.com/recaptcha/docs/versions

- reCAPTCHA v2 ("I'm not a robot" Checkbox)
- reCAPTCHA v2 (Invisible reCAPTCHA badge)
- reCAPTCHA v2 (Android)
- reCAPTCHA v3

### reCAPTCHA v2 ("I'm not a robot" Checkbox)

「私はロボットではありません」（もしくは「I'm not a robot.」）というチェックボックスが表示され、クリックすることでユーザが人間かどうかを検証します。

### reCAPTCHA v2 (Invisible reCAPTCHA badge)

チェックボックスをクリックする操作が不要になり、ページ上での振る舞いをもとにユーザが人間かどうかを検証します。デフォルトでは、振る舞いが怪しい場合に画像を選択させ、認証を行います。「信号機の画像をすべて選択してください。」みたいなやつですね。

### reCAPTCHA v2 (Android)

Google PlayサービスのSafetyNet APIsの一部です。**reCAPTCHA v2 ("I'm not a robot" Checkbox)** のようなチェックボックスを表示することはできません。デバイスが多様であることなどの理由があるそうです。[参考ページ](https://cloud.google.com/recaptcha-enterprise/docs/instrument-android-apps?hl=ja)

### reCAPTCHA v3

ユーザの入力などなしに、振る舞いからスコアを算出します。スコアは0.0〜1.0の間で、人間らしければ1.0に近づきます。スコアをもとに何をするのかを実装する必要があります。実装の中にはスコアが〇〇以下ならすべてのアクセスを拒否する、という足切りも行うことができます。その場合たとえアクセスが人間でもアクセスできなくなります。

## 参考文献

[reCAPTCHA v2 と v3 の違い](https://qiita.com/koseki/items/a2436b831afcdf73b946)

[[e-Words] reCAPTCHA](https://e-words.jp/w/reCAPTCHA.html)