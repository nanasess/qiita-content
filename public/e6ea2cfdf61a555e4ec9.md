---
title: Windows 版 Emacs で起動時にエラー
tags:
  - Emacs
  - Windows
private: false
updated_at: '2015-09-09T10:03:36+09:00'
id: e6ea2cfdf61a555e4ec9
organization_url_name: ec-cube
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---
環境
------

- GNU Emacs 24.5.1 (x86_64-w64-mingw32)
- Windows7

概要
-----

Emacs (runemacs.exe) のアイコンをダブルクリックし、起動しようとすると以下のような Warning が発生する。

```
Warning (initialization): An error occurred while loading `c:/Users/username/.emacs.d/init.elc':

Wrong type argument: stringp, nil

To ensure normal operation, you should investigate and remove the
cause of the error in your initialization file.  Start Emacs with
the `--debug-init' option to view a complete error backtrace.
```

よくよく調べてみると `(require 'tramp)` でエラーが発生していた。

```
Debugger entered--Lisp error: (wrong-type-argument stringp nil)
  string-match("cmd\\.exe" nil)
  (if (string-match "cmd\\.exe" tramp-encoding-shell) "/c" "-c")
  eval((if (string-match "cmd\\.exe" tramp-encoding-shell) "/c" "-c"))
  byte-code("\303\304N\211\203
```

原因
-----

環境変数 `COMSPEC` が設定されてないのが原因。
`COMSPEC` に `C:\Windows\System32\cmd.exe` を設定することで解決

![キャプチャ.PNG](https://qiita-image-store.s3.amazonaws.com/0/40662/e4eeb0fd-9701-f5d6-8d66-6fd65902082f.png)


参考
---

<http://stackoverflow.com/questions/29477035/error-when-loading-emacs-with-emacs-live>
