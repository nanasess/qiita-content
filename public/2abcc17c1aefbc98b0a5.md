---
title: PHP で "false" は true になるから注意
tags:
  - PHP
private: false
updated_at: '2017-02-23T16:40:49+09:00'
id: 2abcc17c1aefbc98b0a5
organization_url_name: ec-cube
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---
PHP で、文字列の `"false"` は true になるので注意しましょう。

```php
$test = "false";
var_dump((bool) $test);

// => bool(true)
```

環境変数に格納した値を評価したりする場合は要注意です。

```php
// デバッグモードを環境変数に保存
putenv("debug_mode=false");

if (getenv("debug_mode")) {
    echo "debug_mode = ON";
} else {
    echo "debug_mode = OFF";
}

// => 予想に反して debug_mode = ON が出力されてしまう
```

`===` を使用して評価するか、 `0 or 1` を使用すると良いでしょう。

```php
// デバッグモードを環境変数に保存
putenv("debug_mode=0");

if (getenv("debug_mode")) {
    echo "debug_mode = ON";
} else {
    echo "debug_mode = OFF";
}

// => 想定通り debug_mode = OFF が出力される
```
