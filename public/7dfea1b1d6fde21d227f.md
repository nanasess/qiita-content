---
title: PHP8 で Call to undefined function each() になる場合の対処法
tags:
  - PHP
  - PHP8
private: false
updated_at: '2020-12-08T15:18:36+09:00'
id: 7dfea1b1d6fde21d227f
organization_url_name: ec-cube
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---
PHP8 で [each](https://www.php.net/each) 関数が削除されたため、 `Call to undefined function each()` というエラーになります。

例えば、以下のような処理がある場合は、 `current()` と `next()` を使用した処理に書き換えます。

```php
// PHP7 までは each() 関数が使える
while (list($k, $v) = each($array)) {
    echo $k;
    echo $v;
}
```

```php
// PHP8 以降は each() 関数は使えない
while ($current = current($array)) {
    $k = key($array);
    $v = $current;
    echo $k;
    echo $v;
    next($array);
}
```

## Composer のパッケージで each() が使われている場合

Composer のパッケージで `each()` 関数が使われている場合は、直接書き換えるわけにはいきませんので、解決するためのパッケージを作成しました。

以下のコマンドで PHP8 でも `each()` 関数を使えるようになります。

```
composer require nanasess/php8-compat
```

本来は、こういった互換プログラムを使用するのはあまりよろしくないので、
1. 対応されるまで待つ
2. `each()` 関数を使わないよう修正し、 Pull Request する
などの対応をするのが良いと思います。

