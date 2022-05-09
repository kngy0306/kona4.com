---
title: "（自分用メモ）useContextの使い方"
subtitle: ""
date: 2022-05-09T08:41:10+09:00
draft: false
tags: ["typescript", "react"]
bigimg: [{src: "/img/2022/05-09-code.jpg", desc: ""}]
image: "/img/2022/05-09-useContext.jpg"
description: "TypeScript + Reactの環境でuseContextの使い方について自分用のメモを記事にしました。"
---

React16.8から加わったHooksの1つである、`useContext`の使い方を自分用のメモとして記事を書きました。

CodeSandboxでサンプルコードをかきました。下記URLからコードを確認していただくと、分かりやすいと思います。

https://codesandbox.io/embed/wispy-voice-1uwjo7?fontsize=14&hidenavigation=1&theme=dark

<!--more-->

## ソースコード

App.tsx

```ts

import "./styles.css";
import { Page1 } from "./Page1";
import { useState } from "react";
import React from "react";

type messageType = {
  message: string;
  setMessage: React.Dispatch<React.SetStateAction<string>>;
};

export const MessageContext = React.createContext({} as messageType);

export default function App() {
  const [message, setMessage] = useState("Appページのメッセージ");

  return (
    <MessageContext.Provider value={{ message, setMessage }}>
      <div className="App">
        <h1>App</h1>
        <p>{message}</p>
        <Page1 />
      </div>
    </MessageContext.Provider>
  );
}

```

Page1.tsx

```ts

import { Page2 } from "./Page2";

export const Page1 = () => {
  return (
    <div className="Page1">
      <h1>Page1</h1>
      <Page2 />
    </div>
  );
};

```

Page2.tsx

```ts

import React, { useContext } from "react";
import { MessageContext } from "./App";

export const Page2 = () => {
  const { message, setMessage } = useContext(MessageContext);

  const changeHandle = (event: React.ChangeEvent<HTMLInputElement>) => {
    setMessage(event.target.value);
  };

  return (
    <div className="Page2">
      <h1>Page2</h1>
      <input
        value={message}
        id="message"
        onChange={(e) => {
          changeHandle(e);
        }}
      />
    </div>
  );
};

```

## 解説

{{< rawhtml >}}<img src="/img/2022/05-09-com.jpg" alt="comp" width=100%>{{< /rawhtml >}}

**App.tsx**が上位に位置するコンポーネントであり、その中に**Page1.tsx**を配置し、さらに**Page1.tsx**の中に**Page2.tsx**が配置されています。

サンプルコードを操作していただくとわかりますが、**Page2.tsx**コンポーネントにあるinput要素へ文字を入力すると、**App.tsx**のテキストが同時に変化します。

### App.tsx

**App.tsx**では`export const MessageContext = React.createContext({} as messageType);`でコンテクストオブジェクトを作成&エクスポートしています。  
また`<MessageContext.Provider value={{ message, setMessage }}>`で下位のコンポーネントを囲っています。これにより下位のコンポーネントが**value**で渡した値を使用することができ、その操作によって値が変化します。  
サンプルコードでは**useState**で作成した値をオブジェクトとして、valueで渡しています。

### Page2.tsx

**Page2.tsx**では、`import { MessageContext } from "./App";`と`export const Page2 = () => { const { message, setMessage } = useContext(MessageContext);`によって**App.tsx**で定義したvalueの値を受け取っています。

上記のようにすることで複数コンポーネントをまたいだ値の受け渡しを行うことができます。

## 参考

https://ja.reactjs.org/docs/hooks-reference.html#usecontext