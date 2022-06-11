---
title: "PHPで理解するDI"
subtitle: ""
date: 2022-06-11T08:23:17+09:00
draft: false
tags: ["php"]
bigimg: [{src: "/img/2022/06-11-thum.jpg", desc: ""}]
image: "/img/2022/06-11-thum.jpg"
description: "DIについて学習したことのアウトプット記事です。"
---

DIについて学習したことを記事にしました。例で用いるコードにはPHPを使用しています。

<!--more-->

## DI(Dependency Injection)について

クラス間の依存関係をなくすためのデザインパターンの1つです。
クラスの内部で他クラスのインスタンスを作成するのではなく、クラスの外部で他クラスのインスタンスを作成し、コンストラクタなどでインスタンスを渡す手法です。

## 依存が起こる例

例として、PHPを用いて**通知を送信するクラス**を作成しました。  
通知を行うにはさまざまな方法がありますが、以下の例ではメールを用いて通知を送信することを想定しています。

Main.php
```php

<?php

// 通知クラス
class Notification {
  private $platform;

  public function __construct()
  {
    // Emailクラスのインスタンスを作成
    $email = new Email();
    $this->platform = $email;
  }

  // 通知を送信するメソッド
  public function pushNotification()
  {
    $this->platform->notify();
  }
}

// E-mailクラス
class Email {
  public function notify()
  {
    // Email特有の処理 〜
    
    echo "Emailで通知を送信しました。";
  }
}

// メイン処理
$noti = new Notification();
$noti->pushNotification();

```

実行

```sh

$ php Main.php

Emailで通知を送信しました。

```

上の例において、NotificationクラスはEmailクラスに依存しています。依存というのは、Emailクラスがないと動作できないという意味です。  
依存していることで、Emailクラス以外を作りづらいという弊害が発生します。

Emailクラスに加え、Slackクラスを追加したくなった場合、以下のような実装になりえます。


Main.php
```php

<?php

// 通知クラス
class Notification {
  private $platform;

  public function __construct($platformType)
  {
    // 引数によって分岐させる
    if ($platformType === "Email") {
      $this->platform = new Email();
    } elseif ($platformType === "Slack") {
      $this->platform = new Slack();
    }
  }

  public function pushNotification()
  {
    $this->platform->notify();
  }
}

// E-mailクラス
class Email {
  public function notify()
  {
    // Email特有の処理 〜

    echo "Emailで通知を送信しました。";
  }
}

// Slackクラス
class Slack {
  public function notify()
  {
    // Slack特有の処理 〜

    echo "Slackで通知を送信しました。";
  }
}

// メイン処理
$noti = new Notification("Slack");
$noti->pushNotification();

```

実行

```sh

$ php Main.php

Slackで通知を送信しました。

```

このままでは、Slackクラスの他にもさまざまなクラスを追加する場合、分岐処理がどんどん増えていってしまいます。

## 依存性の注入

これまでの例では、Notificationクラスの中で他のクラスのインスタンスを作成することで依存が発生していました。  
Notificationクラスの中ではなく、メイン処理でインスタンスを作成し、インスタンスをNotificationクラスへ渡すことで依存をなくすことができます。  
上記の手法が**依存性の注入**と呼ばれています。

Main.php

```php

<?php

// 通知クラス
class Notification {
  private $platform;

  public function __construct(Notifiable $notificationPlatform)
  {
    $this->platform = $notificationPlatform;
  }

  public function pushNotification()
  {
    $this->platform->notify();
  }
}

// 通知可能インタフェース
interface Notifiable {
  public function notify();
}

// E-mailクラス
class Email implements Notifiable {
  public function notify()
  {
    // Email特有の処理 〜

    echo "Emailで通知を送信しました。";
  }
}

// Slackクラス
class Slack implements Notifiable {
  public function notify()
  {
    // Slack特有の処理 〜

    echo "Slackで通知を送信しました。";
  }
}

// メイン処理
// メイン処理でインスタンス作成
$platformType = "Slack";
if ($platformType === "Email") {
  $platform = new Email();
} elseif ($platformType === "Slack") {
  $platform = new Slack();
}

// インスタンスを渡す
$noti = new Notification($platform);
$noti->pushNotification();

```

実行

```sh

$ php Main.php

Slackで通知を送信しました。

```

新たにNotifiableというインタフェースを作成しました。  
上記の実装で、Notificationクラスの依存先はNotifiableインタフェースへ変更されました。これにより、NotificationクラスはNotifiableインタフェースを満たしているインスタンスであれば何でも引数で受け取ることができます。

今後新しいクラスがいくら増えても、Notificationクラスは修正する必要がありません。

## 参考

[PHP本格入門[上]](https://www.amazon.co.jp/PHP%E6%9C%AC%E6%A0%BC%E5%85%A5%E9%96%80-%E4%B8%8A-%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E3%81%A8%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E6%8C%87%E5%90%91%E3%81%AE%E5%9F%BA%E7%A4%8E%E3%81%8B%E3%82%89%E3%83%87%E3%83%BC%E3%82%BF%E3%83%99%E3%83%BC%E3%82%B9%E9%80%A3%E6%90%BA%E3%81%BE%E3%81%A7-%E5%A4%A7%E5%AE%B6-%E6%AD%A3%E7%99%BB/dp/4297114682)  
[PHP本格入門[下]](https://www.amazon.co.jp/PHP%E6%9C%AC%E6%A0%BC%E5%85%A5%E9%96%80-%E4%B8%8B-%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E6%8C%87%E5%90%91%E8%A8%AD%E8%A8%88%E3%80%81%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E3%80%81%E7%8F%BE%E5%A0%B4%E3%81%A7%E4%BD%BF%E3%81%88%E3%82%8B%E5%AE%9F%E8%B7%B5%E3%83%8E%E3%82%A6%E3%83%8F%E3%82%A6%E3%81%BE%E3%81%A7-%E5%A4%A7%E5%AE%B6-%E6%AD%A3%E7%99%BB/dp/4297114704/ref=pd_lpo_1?pd_rd_i=4297114704&psc=1)