---
title: "TypeScriptにおける、部分型について"
subtitle: ""
date: 2022-05-21T10:37:47+09:00
draft: false
tags: ["typescript"]
description: "TypeScriptにおける、オブジェクトと関数型の部分型について自分なりのアウトプット記事です。"
---

TypeScriptのオブジェクトにおいて、オブジェクトと関数型の部分型についてのアウトプット記事です。

<!--more-->

## 部分型とは

2つの型の互換性を表す概念。
型Sが、型Tの部分型であるとき、型Sの値が型Tの値でもあることを表す。

# オブジェクトの部分型について

## 具体例

```ts

type Bar = {
  name: string;
  gender: string;
  age: number;
}

type Foo = {
  name: string;
  gender: string;
}

const bar: Bar = {
  name: "kona",
  gender: "male",
  age: 23,
};

const foo: Foo = bar;
console.log(foo);

```

上記の例では、Bar型は、Foo型の部分型であるといえる。なのでFoo型の変数fooに、Bar型の変数barを代入することができている。  
BarはFooが持っていないageプロパティを持っているが、Fooではname,genderプロパティ以外には言及されていないため、余分なプロパティがあっても問題ない。

ただし、持っていないプロパティにアクセスするのはコンパイルエラーになる。


## プロパティの包含関係

オブジェクトの場合、プロパティの包含関係によって部分型関係を説明できる。  
型S, 型Tがオブジェクト型であるとし、以下の2つの条件が満たされていればSはTの部分型である。

- Tが持つプロパティはすべてSにも存在する
- 条件1のプロパティについて、Sにおけるプロパティの型はTの型の部分型（または同じ型）

## 余剰プロパティに対するエラー

```ts

const foo: Foo = {
  name: "kona",
  gender: "male",
  age: 23,
};

// 型 '{ name: string; gender: string; age: number; }' を型 'Foo' に割り当てることはできません。
// オブジェクト リテラルは既知のプロパティのみ指定できます。'age' は型 'Foo' に存在しません

```

上記の例は、オブジェクトリテラルでFoo型の変数にBar型と同じ型を代入しているが、エラーになる。  
オブジェクトリテラルでの代入は、TypeScriptが補助的にエラーを出力してくれる。（余分なプロパティはいらないので）

# 関数型の部分型について

## 返り値の型による部分型関係

SがTの部分型なら、(arg: U) => Sという関数型は、(arg: U) => Tという関数型の部分型になる。

```ts

type HasName = {
  name: string;
};

type HasNameAndAge = {
  name: string;
  age: number;
};

const fromAge = (age: number): HasNameAndAge => ({
  name: "kona",
  age
});

const f: (age: number) => HasName = fromAge;
const obj: HasName = f(100);
console.log(obj); // { name: 'kona', age: 100 }

```

上記の例では、HasNameAndAge型はHasName型の部分型である。  
変数fは、 `(引数: number) => HasName` という関数型であり、これに `(引数: number) => HasNameAndAge` を代入している。

出力結果から、objはnameプロパティだけしかないオブジェクトにはならず、ageプロパティも所持している。  
TypeScriptでは型情報によってランタイムの挙動に影響を与えない。

また、どんな型を返す関数型も、voidを返す関数型の部分型として扱われる。以下の書き方も可能。（余分な返り値は無視すれば良いので）

```ts

const v: (age: number) => void = fromAge;

```

## 引数の型による部分型関係 (部分型の向きが逆転する)

```ts

type HasName = {
  name: string;
};

type HasNameAndAge = {
  name: string;
  age: number;
};

const showName = (obj: HasName) => {
  console.log(obj.name);
};

const g: (obj: HasNameAndAge) => void = showName;

g({
  name: "kona",
  age: 23,
});

```

上の例では、  
HasNameという型を引数に持つshowName関数を、  
HasNameAndAgeという型を引数に持つg関数へ代入している。

これは`HasName型を受け取る関数`の型が、`HasNameAndAge型を受け取る関数`の部分型であるといえる。  
オブジェクトの部分型では、`HasNameAndAge`型が、`HasName`型の部分型であった。  
引数の部分型関係は逆転する。

オブジェクトの場合は、`HasName`型に定義したプロパティ以外のプロパティがあっても関係ないので、`HasNameAndAge`型を代入することができる。  
→ `HasNameAndAge`型は`HasName`型である。

関数の引数の場合、関数（`HasName型を受け取る`）のメソッドで使用するプロパティは、引数で定義されているプロパティなので、そのプロパティが定義されている引数の関数（`HasNameAndAge型を受け取る`）ならば、代入することができる。（余分なプロパティは無視すれば良いので）  
→ `HasName型を受け取る関数`の型は`HasNameAndAge型を受け取る関数`の型でもある。

## 引数の数による部分型関係

```ts

type Double = (arg: number) => number;
type Plus = (left: number, right: number) => number;

const d: Double = arg => arg * 2;
const p: Plus = (left, right) => left + right;

const bin: Plus = d;
console.log(bin(50, 1)); // 100

```

上の例では、Plus型の関数にDouble型の関数を代入している。  
DoubleはPlusの部分型。

Plusより引数が少ないため、余分なものは無視される。

# 参考

[プロを目指す人のためのTypeScript入門 安全なコードの書き方から高度な型の使い方まで](https://gihyo.jp/book/2022/978-4-297-12747-3)