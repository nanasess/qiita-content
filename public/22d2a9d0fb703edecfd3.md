---
title: MediaWiki + Short URL + IIS で日本語の URL が文字化けしてしまう
tags:
  - PHP
  - mediawiki
  - IIS
private: false
updated_at: '2015-07-29T15:03:33+09:00'
id: 22d2a9d0fb703edecfd3
organization_url_name: ec-cube
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---
環境
------

* MediaWiki 1.25.1
* IIS 8.0(Microsoft Azure WebApps[旧WebSites])
* PHP5.4.42
* URL Rewrite

現象
-----

<https://www.mediawiki.org/wiki/Manual:Short_URL/IIS7> を参考に、 Short URLの設定をしたところ、一部の日本語URLが文字化けしてしまう事象が発生

例)
http://example.com/wiki/あ
↓ 特定の文字が以下のように文字化けしてしまう
http://example.com/wiki/Ã ‚

原因
-----

MediaWiki の `WebRequest::getPathInfo()` で、 `$_SERVER['REQUEST_URI']` からタイトルを抽出しようとした際、 `parse_url()` 関数に URLエンコードされていないマルチバイト文字列が渡ってしまい、 `parse_url()` 関数の結果が文字化けしてしまう。

対応方法
---------

`parse_url()` 関数に、 URLエンコードされた文字列を渡すようにする。
根本的には、 IIS + URLRewrite が生成する `$_SERVER['REQUEST_URI']` に、 URLエンコードされていない文字列が出力されてしまうのが問題だが、これを回避するために、 `$_SERVER['UNENCODED_URL']` が取得できる場合は、こちらを利用する。

### 修正パッチ

```diff
--- a/includes/WebRequest.php
+++ b/includes/WebRequest.php
@@ -94,7 +94,12 @@ class WebRequest {
 		$matches = array();
 		if ( !empty( $_SERVER['REQUEST_URI'] ) ) {
 			// Slurp out the path portion to examine...
-			$url = $_SERVER['REQUEST_URI'];
+			if (!empty($_SERVER["UNENCODED_URL"])) {
+				$url = $_SERVER["UNENCODED_URL"]; // for Microsoft Azure
+			} else {
+				$url = $_SERVER['REQUEST_URI'];	
+			}
+
 			if ( !preg_match( '!^https?://!', $url ) ) {
 				$url = 'http://unused' . $url;
```


TODO
--------

* MediaWiki にバグレポートを出そうと思うが、 [Gerret](https://www.mediawiki.org/wiki/Gerrit/ja) の使い方を勉強しなくてはならない...
