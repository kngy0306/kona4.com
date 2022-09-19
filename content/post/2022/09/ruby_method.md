---
title: "Rubyのさまざまな変数についてまとめました"
subtitle: ""
date: 2022-09-16T11:50:28+09:00
draft: false
tags: ["ruby"]
bigimg: [{src: "/img/2022/ruby_method_thum.jpg", desc: ""}]
image: "/img/2022/ruby_method_thum.jpg"
description: "Rubyではインスタンス変数、クラスインスタンス変数などがあります。それぞれの特徴をまとめました"
---

Rubyには以下のようにさまざまな**変数**が存在します。それぞれの記述方法、特徴をまとめました。

- ローカル変数
- インスタンス変数
- クラスインスタンス変数
- クラス変数
- グローバル変数
- 組み込み変数

<!--more-->

# ローカル変数

Rubyにおいて、`@`などのプレフィックスがつかない変数のことを**ローカル変数**といいます。

```rb

user_name = "kona"
number = 123

puts user_name # "kona"
puts number # 123

```

ローカル変数は宣言と代入を同時に行う必要があります。

```rb

# 変数名だけだとエラー
user_name
# undefined local variable or method `user_name' for main:Object (NameError)

```

# インスタンス変数

クラスの内部で、`@`がついた変数を**インスタンス変数**といいます。  
同じインスタンス（オブジェクト）の中で共有されます。

```rb

class Book
  def initialize(name)
    @name = name
  end

  def description
    puts "This is #{@name}"
  end
end

book1 = Book.new("book1")
book1.description # This is book1

book2 = Book.new("book2")
book2.description # This is book2

```

インスタンス変数は宣言前でも呼び出すことが可能です。（ただしnilが返る）

```rb

class Book
  def description
    puts "This is #{@name}"
  end
end

book = Book.new
book.description # This is

```

インスタンス変数は、デフォルトではクラスの外部からアクセスすることはできません。

```rb

class Book
  def initialize(name)
    @name = name
  end

  def description
    puts "This is #{@name}"
  end
end

book1 = Book.new("book1")
puts book1.name # undefined method `name' for #<Book:0x000000010073aec8 @name="book1"> (NoMethodError)

```

以下のメソッドを使用することで外部からもアクセスが可能になります。

- `attr_reader`
- `attr_writer`
- `attr_accessor`

`attr_reader`：読み込み可能に  
`attr_writer`：書き込み可能に  
`attr_accessor`：読み書き可能に

```rb

class Book
  attr_accessor :name # 追加

  def initialize(name)
    @name = name
  end

  def description
    puts "This is #{@name}"
  end
end

book1 = Book.new("book1")
puts book1.name # book1 ← エラーがなくなる

```

# クラスインスタンス変数

クラスインスタンス変数は、クラス自信が保持している変数です。  
クラス直下、またはクラスメソッドの中で`@`を付けて宣言された変数がクラスインスタンス変数になります。

```rb

class Book
  # クラスインスタンス変数
  @name = "book_class"

  def self.name
    # クラスインスタンス変数
    @name
  end

  attr_accessor :name

  def initialize(name)
    # インスタンス変数
    @name = name
  end

  def description
    # インスタンス変数 ↓の@name
    puts "This is #{@name}"
  end
end

class Comic < Book
  # クラスインスタンス変数
  @name = "comic_class"

  def self.name
    # クラスインスタンス変数
    @name
  end
end

book = Book.new("book")
puts book.name # book
puts Book.name # book_class

puts Comic.name # comic_class

```

# クラス変数

クラスインスタンス変数は、スーパークラスとサブクラスで異なる変数として参照されていましたが、クラス変数はサブクラスでも同じ変数を参照できるようになります。  
クラス変数は`@@`を付けて宣言します。

```rb

class Book
  @@name = "book_class"

  def initialize(name)
    @@name = name
  end

  def self.name
    @@name
  end

  def name
    @@name
  end
end

class Comic < Book
  @@name = "comic_class"

  def self.name
    @@name
  end

  def name
    @@name
  end
end

puts Book.name # comic_class → Comicクラスの定義時に上書きされる
puts Comic.name # comic_class

book = Book.new("book")
puts book.name # book

puts Comic.name # book → bookインスタンス作成時に上書きされる

```

# グローバル変数

グローバル変数は、プログラムのどこからでも参照することができる変数です。  
`$`をを付けて宣言します。

```rb

$name = "book_global"

class Book
  def initialize(name)
    $name = name
  end

  def name
    $name
  end
end

puts $name # book_global
book = Book.new("book1")
puts book.name # book1
puts $name # book1

```

# 組み込み変数

組み込み変数は、グローバル変数の中で、Rubyによってあらかじめ用途を定めている変数です。  
例として、正規表現においてマッチした部分の値が代入される値として使用されています。

```rb

text = "住所は、123-4567です"
text =~ /(\d{3})-(\d{4})/

puts $& # 123-4567
puts $1 # 123
puts $2 # 4567

```

その他組み込み変数（特殊変数）は以下のページから確認できます。  
https://docs.ruby-lang.org/ja/latest/class/Kernel.html

# 参考文献

変数と定数  
https://docs.ruby-lang.org/ja/latest/doc/spec=2fvariables.html

プロを目指す人のためのRuby入門［改訂2版］  
https://gihyo.jp/book/2021/978-4-297-12437-3
