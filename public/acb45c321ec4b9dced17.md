---
title: Mac の sed で改行コードを変換
tags:
  - ShellScript
  - Mac
  - sed
  - BSD
private: false
updated_at: '2019-11-21T09:15:21+09:00'
id: acb45c321ec4b9dced17
organization_url_name: ec-cube
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---
Mac は BSD sed なので、情報が少ないんですよね。

### Windows(CRLF) → UNIX(LF) へ置換する

```shell
sed -i.bak 's/'$'\r//' filename
```

拡張子を指定して、まとめて置換したい場合はこちら

```shell
export LANG=C # Mac の BSD sed を使う場合のおまじない

find . -name '*.html' -or -name '*.css' -or -name '*.js' -print0 | xargs -0 sed -i.bak 's/'$'\r//'
```

### GNU sed を使う

Linux と挙動が違ってて面倒！っていう人は、 GNU sed を入れてください

```shell
brew install gnu-sed
```

でも、さくらのレンタルサーバーなんかは FreeBSD なので、 BSD sed も覚えておいて損は無いと思いますよ
