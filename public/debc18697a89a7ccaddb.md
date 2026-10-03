---
title: 1行のコマンドでローカル環境にEC-CUBEを最速でインストールする
tags:
  - PHP
  - SQLite3
  - EC-CUBE3
private: false
updated_at: '2018-05-30T09:21:56+09:00'
id: debc18697a89a7ccaddb
organization_url_name: ec-cube
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---
先日、 [ぼくのかんがえたさいきょうのEC-CUBE3開発環境](http://qiita.com/kiy0taka/items/1bd6dae16c8b5c99151a) というエントリがありましたが、もっと手軽に最速で試したい方向けです。

1行のコマンドで、簡単に EC-CUBE3 がローカル環境にインストールできます。
**たぶん最速。手元の環境では10秒程度でインストールできました。** *(もっと速いやり方をご存知の方はつっこみください)*

下準備
---------

### Mac の方

Max OS X 10.9 以降の方は、特に何もしなくて大丈夫です。

### Windows の方

1. [Web Platform Installer](https://www.microsoft.com/web/downloads/platform.aspx) などで **PHP5.4 以降**をインストールします
2. [Git for Windows](https://git-for-windows.github.io) から **Git BASH** をインストールします

以降、 Git BASH でコマンドを実行します。


EC-CUBE3 のインストール
--------------------------

以下のコマンドを Terminal や Git BASH へコピー&ペーストし、実行してください。EC-CUBE のダウンロードからインストール、 Webサーバーの起動まで一発で可能です。
データベースには SQLite3 を使用します。

```shell
## for Mac
export ECCUBE_VERSION=3.0.16 && curl "http://downloads.ec-cube.net/src/eccube-${ECCUBE_VERSION}.tar.gz" | tar xvzf - && cd "eccube-${ECCUBE_VERSION}" && php eccube_install.php sqlite3 none && php -S localhost:8080 -t html index.php

## for Windows
export ECCUBE_VERSION=3.0.16 && curl "http://downloads.ec-cube.net/src/eccube-${ECCUBE_VERSION}.tar.gz" | tar xvzf - && cd "eccube-${ECCUBE_VERSION}" && php eccube_install.php sqlite3 none && php -S localhost:8080 -t html html/index.php
```

このコマンドは、 EC-CUBE3.0.12 以降で使用可能です。
EC-CUBE3.0.12 以降のバージョンを使用したい場合は、 `ECCUBE_VERSION=3.0.16` 等にバージョンを変更してください。

以下のようなメッセージが表示されたら、 ブラウザで http://localhost:8080 へアクセスしてください。

```
EC-CUBE3 install finished successfully!
PHP built-in web server to run applications, `php -S localhost:8080 -t html`
Open your browser and access the http://localhost:8080/
PHP 5.6.15 Development Server started at Mon Dec  5 17:21:50 2016
Listening on http://localhost:8080
Document root is /path/to/eccube-3.0.16/html
Press Ctrl-C to quit.
```

EC-CUBE3 のトップページが表示されると思います！
管理画面には http://localhost:8080/admin にて ID: admin PASS: password でログインできます。

XAMPP や MAMP、 Virtual Box や Vagrant、 Docker などを構築するより断然手軽です。


*ただし、あくまでも開発用ですので、本番環境では絶対に使用しないでください。*

EC-CUBE のインストールで挫折された方も多いと思います。ぜひお試しください。
