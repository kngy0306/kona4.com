---
title: "M1 ProチップのMacにRubyをインストールする"
subtitle: ""
date: 2022-07-23T09:20:44+09:00
draft: false
tags: ["ruby"]
# bigimg: [{src: "/img/2022/", desc: ""}]
# image: "/img/2022/"
description: "M1 Proチップ, シェルがfish shellの環境でRubyをインストールする"
---

インストールを行った環境

- macOS Monterey 12.4
- Apple M1 Pro
- fish shell

<!--more-->

## 流れ

1. rbenvをインストールする
1. パスを通す
1. Rubyをインストールする

## 前提

brewが使用できる状態

```sh

$ brew -v
Homebrew 3.5.6

```

## rbenvをインストールする

```sh

$ brew install ruby-build rbenv

```

`rbenv`は`ruby-build`に依存しているため一緒にインストールを行う。

## パスを通す

config.fishファイルを編集する。
著者の場合は以下にconfig.fishが存在する。

```sh

~/.config/fish/config.fish

```

config.fishに以下の2行を追記する。

```sh

set -x PATH $HOME/.rbenv/bin/ $PATH
eval (rbenv init - | source)

```

config.fishファイルを反映させる。

```sh

$ source ~/.config/fish/config.fish

```

rbenvコマンドが使用できるか確認する。記事作成時は1.2.0。

```sh

$ rbenv -v
rbenv 1.2.0

```

## Rubyをインストールする

インストール可能なRubyのバージョンを確認する。

```sh

$ rbenv install -l

```

記事作成時は以下のように表示された。

```sh

$ rbenv install -l
2.6.10
2.7.6
3.0.4
3.1.2
jruby-9.3.6.0
mruby-3.1.0
picoruby-3.0.0
rbx-5.0
truffleruby-22.1.0
truffleruby+graalvm-22.1.0

Only latest stable releases for each Ruby implementation are shown.
Use 'rbenv install --list-all / -L' to show all local versions.

```

Rubyをインストールする。（記事作成時の最新安定版は3.1.2）

```sh

$ rbenv install 3.1.2

```

PC全体で使用するRubyのバージョンを、上記でインストールしたバージョンに切り替える。

```sh

$ rbenv global 3.1.2

```

shimのrehash。このコマンドによって、`~/.rbenv/shims`に設定をコピーしている。

```sh

$ rbenv rehash

```

Rubyのバージョン確認。

```sh

$ ruby -v
ruby 3.1.2p20 (2022-04-12 revision 4491bb740a) [arm64-darwin21]

```

## 参考

- [M1 MacにRubyとRailsの環境構築してみた](https://zenn.dev/osuzuki/articles/a535b2840bbea3)  
- [rbenv rehashをちゃんと理解する](https://mogulla3.tech/articles/2020-12-29-01)  
