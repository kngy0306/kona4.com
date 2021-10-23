---
title: "package.jsonに記載されていないパッケージをアップグレードする方法メモ"
subtitle: ""
date: 2021-10-23T11:17:14+09:00
draft: false
tags: ["React"]
bigimg: [{src: "/img/2021/10-23-yarn.png", desc: ""}]
image: "/img/2021/10-23-yarn.png"
description: "package.jsonに記載されていないパッケージをアップグレードする方法についてまとめました。"
---

この記事は、package.jsonに記載されていないパッケージをアップグレードする方法について調べたときのメモになります。

<!--more-->

Reactのプロジェクトを作成し、GItHubに上げたところ、
{{<figure src="/img/2021/10-23-ss.jpg" alt="screenshot">}}
いくつか脆弱性があると指摘されました。

しかしReactプロジェクト内にあるpackage.jsonにはそれらのパッケージは記載されていません。（package.jsonに記載されているパッケージがまた別のパッケージに依存しているため。）

## package.jsonに記載されていないパッケージをアップグレードする方法

`yarn audit`を使用して脆弱性のあるパッケージをチェックすることができます。

```bash

yarn audit

```

実行結果

```bash

> yarn audit
yarn audit v1.22.10
┌───────────────┬──────────────────────────────────────────────────────────────┐
│ moderate      │  Inefficient Regular Expression Complexity in                │
│               │ chalk/ansi-regex                                             │
├───────────────┼──────────────────────────────────────────────────────────────┤

~~~

├───────────────┼──────────────────────────────────────────────────────────────┤
│ Dependency of │ react-scripts                                                │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ Path          │ react-scripts > react-dev-utils > browserslist               │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ More info     │ https://www.npmjs.com/advisories/1002655                     │
└───────────────┴──────────────────────────────────────────────────────────────┘
97 vulnerabilities found - Packages audited: 1689
Severity: 66 Moderate | 30 High | 1 Critical
Done in 2.12s.

>

```

### アップグレードする

```bash

yarn upgrade xxx

```

と指定してパッケージのアップグレードが可能です。

今回は以下のコマンドで一括で全てアップグレードを行いました。

```bash

yarn upgrade --latest

```

また、パッケージ1つづつ確認しながらアップグレードができるコマンド`yarn upgrade-interactive`もあります。

どちらのコマンドも、`--latest`をつけることでpackage.jsonを無視して最新にアップグレードされます。

## package.jsonに記載されているパッケージのアップグレード方法

package.jsonに記載されているパッケージであれば、

```bash

yarn outdated

```

を実行することでバージョンの情報を教えてくれます。

アップグレードしたいパッケージがあれば、

```bash

yarn add xxx@1.6

```

などバージョンを指定してインストールしなおす方法や、

```bash

yarn upgrade xxx --latest

```

で最新にアップグレードする方法があります。

その他、

```bash

npx npm-check-updates

```

などでもアップグレード可能なパッケージを確認することができます。（npxを使用しているのは、npm-check-updatesをインストールしていないためです。）

また、

```bash

npx -p npm-check-updates  -c "ncu -u"

```

でアップグレードも可能です。
