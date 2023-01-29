---
title: "【Rails】セキュリティに関わりが大きそうなバリデーションヘルパーを試す"
subtitle: ""
date: 2023-01-28T19:37:28+09:00
draft: false
tags: ["rails"]
# bigimg: [{src: "/img/2022/", desc: ""}]
# image: "/img/2022/"
description: "Ruby on Railsのモデルで使用できるバリデーションヘルパーのうち、セキュリティに大きく関連しそうなヘルパーを抜粋するした。"
---

Ruby on Railsのモデルで使用できるバリデーションヘルパーには多くの種類が存在します。その中でセキュリティに関わりが大きそうなヘルパーを抜粋し、実際の挙動を試してみました。

以下ページ内にある「バリデーションヘルパー」を参考にしています。

https://railsguides.jp/active_record_validations.html

<!--more-->

## 環境

- Ruby　3.1.2
- Rails 7.0.3.1

[scaffold](https://railsdoc.com/page/rails_scaffold)を使用して以下のようなEventモデルを例として作成しました。

```rb
create_table "events", force: :cascade do |t|
  t.string "title_ja"
  t.string "title_en"
  t.integer "telephone_number"
  t.string "day_of_week"
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
end
```

## inclusion

> このヘルパーは、指定の集合に属性の値が含まれているかどうかを検証します。集合には任意のenumerableオブジェクトが使えます。

formにはセレクトボックスを設置し、曜日を選択してもらうようにします。

```rb
<%= form.label :day_of_week, style: "display: block" %>
<%= form.select :day_of_week, options_for_select([['曜日を指定してください', ''], ['日曜日', 'sunday'], ['月曜日', 'monday'], ['火曜日', 'tuesday'], ['水曜日', 'wednesday'], ['木曜日', 'thursday'], ['金曜日', 'friday'], ['土曜日', 'saturday'],]) %>
```

バリデーション配下のようにモデルに記載します。

```rb
validates :day_of_week, inclusion: { in: %w[sunday monday tuesday wednesday thursday friday saturday] }
```

上記バリデーションを行うことでたとえばselectタグ、optionのvalueを変えられても不正な値を防ぐことができます。

{{< rawhtml >}}<img src="/img/2023/01-01.jpg" alt="valueの値を不正に変更している画像" width=100%>{{< /rawhtml >}}

`inclusion`は含まれていることを確認するバリデーションでしたが、その逆で含まれていないことを確認するバリデーションヘルパーである、`exclusion`も存在します。

## format

> このヘルパーは、withオプションで与えられた正規表現と属性の値がマッチするかどうかを検証します。

参考ページに記載されているものと同様に、以下のようにバリデーションを記述しました。この正規表現により英文字しか入力できないようになります。

```rb
validates :title_en, format: { with: /\A[a-zA-Z]+\z/ }
```

## numericality

> このヘルパーは、属性に数値のみが使われていることを検証します。デフォルトでは、整数値または浮動小数点数値にマッチします。これらの冒頭に符号がある場合もマッチします。
> :only_integerをtrueに設定すると、属性の値に対するバリデーションで以下の正規表現が使われます。
> `/\A[+-]?\d+\z/`

```rb
validates :telephone_number, numericality: { only_integer: true }
```

## バリデーションのメッセージ

すべてのバリデーションにおいて、バリデーションエラー時のメッセージを指定できます。すべてのバリデーションにエラーメッセージを指定した例は以下になります。

```rb
class Event < ApplicationRecord
  validates :title_en, format: { with: /\A[a-zA-Z]+\z/, message: '英文字のみ入力できます' }
  validates :telephone_number, numericality: { only_integer: true, message: '数値のみ入力できます' }
  validates :day_of_week, inclusion: { in: %w[sunday monday tuesday wednesday thursday friday saturday], message: '正しい曜日を選択してください' }
end
```
