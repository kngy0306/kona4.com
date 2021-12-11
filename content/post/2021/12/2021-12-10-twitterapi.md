---
title: "TwitterAPIを使って乃木坂46公式アカウントの投稿を取得する【JavaScript】"
subtitle: ""
date: 2021-12-10T22:03:06+09:00
draft: false
tags: ["API"]
bigimg: [{src: "/img/2021/2021-12-10-thum.jpg", desc: "Twitter API v2"}]
image: "/img/2021/2021-12-10-thum.jpg"
description: "Node.jsでTwitterAPI v2を使用し、乃木坂46公式アカウントの投稿を取得する"
---

JavaScriptでTwitterAPIを使ってツイートを取得しました。  
取得する条件がたくさん指定できるのですが、自分が取得した例（**乃木坂46の公式アカウントの投稿を取得**）を紹介しようと思います！

<!--more-->

[Developer Portal](https://developer.twitter.com/ja/docs/basics/developer-portal/overview)にてBearer Tokenを取得済みであることを前提とします。

## 環境

```sh

❯ node -v
v16.5.0

❯ yarn -v
1.22.10

```

## 公式ドキュメント

https://developer.twitter.com/en

上記サイトの、[Search Tweets](https://developer.twitter.com/en/docs/twitter-api/tweets/search/api-reference/get-tweets-search-recent)を試しました。  

いくつかの言語で書かれた[サンプルコード](https://github.com/twitterdev/Twitter-API-v2-sample-code/tree/main/Recent-Search)もGitHubにあるので、そちらを参考にコードを編集しました。  
サンプルコード内の、`recent_search.js`を使用しました。  


## パッケージのインストール

サンプルコードで使用されていたパッケージのインストールを行います。

```sh

yarn add -D dotenv needle

```

## クエリパラメータを編集する

ツイートを取得する条件を指定するために、以下の箇所を編集していきます。

```js

// Edit query parameters below
// specify a search query, and any additional fields that are required
// by default, only the Tweet ID and text fields are returned
const params = {
    'query': 'from:twitterdev -is:retweet',
    'tweet.fields': 'author_id'
}

```

`params`に記載できるものは以下ページ内の**Query parameters**から参照できます。  
https://developer.twitter.com/en/docs/twitter-api/tweets/search/api-reference/get-tweets-search-recent

また、以下ページに**Query parameters**の、`'query'`に指定できる**Operators**が記載されています。  
https://developer.twitter.com/en/docs/twitter-api/tweets/search/integrate/build-a-query

### アカウントを指定する

`from:`のあとに、乃木坂46公式アカウントのusernameを指定します。

```js

const params = {
  'query': 'from:nogizaka46'
}

```

実行してみます。

```sh

❯ node index.js
{
  data: [
    {
      id: '1469316238299914247',
      text: '【ニュース更新】 「乃木坂配信中」にて【食わず嫌い】見破ることはできるか？ 掛橋VS筒井 初めて食べるものを当てるゲーム【対決】公開！ https://t.co/DzKeBqAnQm https://t.co/PPUPJN2Wwb'
    },

  〜〜省略〜〜

```

### ツイート内の画像・動画を取得する

アカウント指定に加え、投稿内の画像を取得しようと思います。  
**media.fields**と**expansions**を追加します。

[ドキュメント](https://developer.twitter.com/en/docs/twitter-api/tweets/search/api-reference/get-tweets-search-recent)内、`media.fields`の説明欄に書かれているように、**media.fields**には**expansions**も記載する必要があります。

> The Tweet will only return media fields if the Tweet contains media and if you've also included the expansions=attachments.media_keys query parameter in your request.

```js

const params = {
  'query': 'from:nogizaka46',
  'expansions': 'attachments.media_keys',
  'media.fields': 'url'
}

```

実行結果

```sh

❯ node index.js
{
  data: [
    {
      attachments: { media_keys: [ '3_1469316236475461634' ] },
      id: '1469316238299914247',
      text: '【ニュース更新】 「乃木坂配信中」にて【食わず嫌い】見破ることはできるか？ 掛橋VS筒井 初めて食べるものを当てるゲーム【対決】公開！ https://t.co/DzKeBqAnQm https://t.co/PPUPJN2Wwb'
    },
    
    〜〜省略〜〜

  ],
  includes: {
    media: [
      {
        media_key: '3_1469316236475461634',
        type: 'photo',
        url: 'https://pbs.twimg.com/media/FGQPVRHVcAIJKaX.jpg'
      },

      〜〜省略〜〜

    ]
}

```

**includes**が追加され、画像や動画のURLが追加で取得できます。

### ツイートの情報を取得する

さらに追加で、ツイートのいいね数などを取得できる`'tweet.fields': 'public_metrics'`を追加してみました。

```js

const params = {
  'query': 'from:nogizaka46',
  'tweet.fields': 'public_metrics',
  'expansions': 'attachments.media_keys',
  'media.fields': 'url'
}

```

実行結果

```sh

❯ node index.js
{
  data: [
    {
      text: '【ニュース更新】 「乃木坂配信中」にて【食わず嫌い】見破ることはできるか？ 掛橋VS筒井 初めて食べるものを当てるゲーム【対決】公開！ https://t.co/DzKeBqAnQm https://t.co/PPUPJN2Wwb',
      attachments: { media_keys: [ '3_1469316236475461634' ] },
      public_metrics: {
        retweet_count: 599,
        reply_count: 12,
        like_count: 4111,
        quote_count: 46
      },
      id: '1469316238299914247'
    },

    〜〜省略〜〜
}

```

**public_metrics**が追加されました。

実行した当時の最新ツイートを確認すると、時間差で多少の違いが出ていますが、たしかにツイートの情報が取得できていることを確認できました。

{{< rawhtml >}}<img src="/img/2021/2021-12-10-result.jpg" alt="result" width=95%>{{< /rawhtml >}}

## 最後に
2021年11月に**Twitter API v2**がTwitterの主要APIになったので、使ってみたい！と思い試してみました。  
今後TwitterAPIを使用したアプリケーションも作ってみたいなと思いました！
