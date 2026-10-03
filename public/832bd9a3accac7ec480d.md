---
title: 文字列を簡単に base64 エンコードする
tags:
  - base64
  - shell
private: false
updated_at: '2018-06-04T09:54:13+09:00'
id: 832bd9a3accac7ec480d
organization_url_name: ec-cube
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---
`openssl` コマンドを使用すれば、コマンドラインで簡単に base64 エンコード可能

```bash
echo -n "XXX" | openssl enc -e -base64
```

> 実行結果
> eHh4Cg==

**-d** でデコードも可能

```bash
echo 'eHh4Cg==' | openssl enc -d -base64
```

> 実行結果
> xxx
