---
title: "putsメソッドはなぜどこでも使用することができるのか"
subtitle: ""
date: 2022-09-20T13:25:40+09:00
draft: false
tags: ["ruby"]
# bigimg: [{src: "", desc: ""}]
# image: ""
description: "putsというメソッドはなぜどこでも使用することができるのか"
---

Rubyにおいて、プログラムのどこでも使用できる`puts`メソッド、`pp`メソッドなどは、どこで定義されており、なぜ使用することができるのか。

<!--more-->

# 結論

`puts`メソッド、`pp`メソッドなどは、**Kernelモジュール**で定義されている。
**Kernelモジュール**は、**Objectクラス**でincludeされている。
Rubyのクラスはすべて、**Objectクラス**を継承しているので、すべてのクラスで`puts`メソッドが使用できる。

# Karnelモジュール

↓Kernelモジュールの、リファレンスマニュアル
https://docs.ruby-lang.org/ja/latest/class/Kernel.html

# Objectクラス

**Objectクラス**は、**Kernelモジュール**をincludeしている。

```rb
puts Object.include?(Kernel) # true
```

すべてのクラスは、**Objectクラス**を継承している。
詳細に言うと、**Stringクラス**や**Integerクラス**は、**Moduleクラス**を継承しており、**Moduleクラス**は**Objectクラス**を継承しています。

```rb
"== Stringクラス =="
puts String.class # Class
puts String.class.superclass # Module
puts String.class.superclass.superclass # Object

"== Integerクラス =="
puts Integer.class # Class
puts Integer.class.superclass # Module
puts Integer.class.superclass.superclass # Object
```

またRubyにおいて、クラス構文に囲まれていない一番外側の部分をトップレベルといい、**Objectクラス**のインスタンスになっています。

```rb
puts self.class # Object
```

以上のことから、Rubyのプログラム内では`puts`メソッドが使用できます。

# 参考文献

プロを目指す人のためのRuby入門［改訂2版］
https://gihyo.jp/book/2021/978-4-297-12437-3
