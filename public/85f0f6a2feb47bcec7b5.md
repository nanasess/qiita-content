---
title: phpbrew で EC-CUBE の開発環境を構築する
tags:
  - PHP
  - EC-CUBE
  - phpbrew
private: false
updated_at: '2020-04-27T16:30:31+09:00'
id: 85f0f6a2feb47bcec7b5
organization_url_name: ec-cube
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---
EC-CUBE を Mac で開発をする際に、いちばんおすすめの方法です。
Docker を使うとか、 MAMP や XAMPP を使うとかいろいろあると思いますが、、

- 頻繁に PHP のバージョンを切り替えたり、
- EC-CUBE のバージョンごとにいくつも環境を作ったり、

する必要があるので、最終的にこれに落ちついてます。
*受託開発などで、案件ごとに PHP バージョンが異なったりする場合は phpenv の方が使いやすいかもしれません。*

## 下準備

```shell
brew install libpng bzip2 icu4c openssl readline jpeg composer freetype libxml2 pcre texinfo mcrypt xz gettext mhash zlib libzip libcurl libiconv
```

## インストール

https://github.com/phpbrew/phpbrew/blob/master/README.ja.md#phpbrewのインストール

## EC-CUBE 向けの環境構築

*PHP バージョンはお好きなものを*

### 基本のものをインストール

```shell
phpbrew install -j $(sysctl -n hw.activecpu) 7.3 +default +dbs +fpm +opcache +openssl="$(brew --prefix openssl@1.1)" +bz2="$(brew --prefix bzip2)" +zlib="$(brew --prefix zlib)"
```

### Extension でインストールしないとエラーになるものをインストール

```shell
phpbrew ext install intl
phpbrew ext install opcache # 古いバージョンは zendopcache とする必要がある
phpbrew ext install apcu
phpbrew ext install gd -- --with-jpeg-dir=/usr/local/opt/jpeg --with-png-dir=/usr/local/opt/libpng --with-freetype-dir=/usr/local/opt/freetype --with-zlib-dir=/usr/local/opt/zlib --enable-gd-native-ttf --enable-gd-jis-conv
phpbrew ext install iconv -- --with-iconv=/usr/local/opt/libiconv
```

## 使用方法

```shell
PHP7.3.10 を使用する
phpbrew use 7.3.10

## あとは通常通りビルトインウェブサーバーで開発

# 4系
bin/console server:run

# 2系, 3系
php -S 0.0.0.0:8000 -t html
```

## php.ini の変更

```shell
## 環境変数 EDITOR のエディタで編集可能
phpbrew config
```
