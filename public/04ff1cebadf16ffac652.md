---
title: Emacs でゲームボーイをやりたい時にするたった一つの方法
tags:
  - Emacs
  - elisp
  - emacs-lisp
  - gameboy
  - eboy
private: false
updated_at: '2019-04-02T14:43:40+09:00'
id: 04ff1cebadf16ffac652
organization_url_name: ec-cube
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---
たまに Emacs でゲームしますよね。 `M-x tetris` とか定番ですが、ゲームボーイで懐しのゲームしたくなる時もありますよね。

https://github.com/vreeze/eboy

**ご利用は自己責任で**

## セットアップ

```shell
git clone https://github.com/vreeze/eboy.git
cd eboy
## byte compile
emacs -q -l eboy-macros.el -l eboy-cpu.el -l eboy.el -batch -f batch-byte-compile *.el
```

## ROM をとってくる

**gameboy rom tetris** とかでぐぐってください

## 遊び方

```shell
emacs -q -l eboy-macros.elc -l eboy-cpu.elc -l eboy.elc
```

`M-x eboy-load-rom` で、 ダウンロードしてきた ROM のファイルを指定すると遊べます

| Gameboy     | Eboy     |
|------------:|---------:|
| Start       | Enter    |
| Select      | Space    |
| B           | D        |
| A           | S        |
| down        | k        |
| up          | i        |
| left        | j        |
| right       | l        |

<img width="1368" alt="スクリーンショット 2019-04-02 12.12.15.png" src="https://qiita-image-store.s3.amazonaws.com/0/40662/193423fa-5dde-5f0f-1f2c-3de0de82667a.png">
