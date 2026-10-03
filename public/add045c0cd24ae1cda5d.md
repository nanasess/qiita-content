---
title: EC-CUBE2系で SendGrid を使用して RFC 違反のメールアドレスで送信する
tags:
  - PHP
  - mail
  - EC-CUBE
  - SendGrid
private: false
updated_at: '2020-07-17T13:49:08+09:00'
id: add045c0cd24ae1cda5d
organization_url_name: ec-cube
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---
`@docomo.ne.jp`, `@softbank.ne.jp`, `@ezweb.ne.jp` といった、携帯キャリアのメールアドレスを利用するユーザーの中には、歴史的な理由から、 以下のようなRFC違反のメールアドレスを利用しているユーザーがいます。

- 先頭がドット `.abcde@example.com`
- 連続したドットが含まれている `ab..cde@example.com`
- アットマークの前がドット `abcde.@example.com`

現在は、こういったメールアドレスを新たに作成することはできませんが、2009年3月以前には作成できており、当時作成したユーザーは現在においても、こういったメールアドレスを使用することができます。
RFC違反のメールアドレスだと、迷惑メールが届きにくいという背景もあり、まだ多くのユーザーが存在します。

これらのメールアドレスの構文は [RFC5321](https://tools.ietf.org/html/rfc5321) や [RFC5322](https://tools.ietf.org/html/rfc5322) に違反しており、Gmail をはじめ、多くのメールサーバーでは送受信することができません。

**EC-CUBE3系及び、4系でも送信することはできません。**

しかし、30代以上で、携帯キャリアのメールアドレスを多用する客層をもつ店舗では、未だにこのようなメールアドレスのユーザーが多く、対応せざるえないのが現状です。

携帯キャリアメールの到達率の良さから、 SendGrid を使用する場合が多いのですが、 RFC違反のメールアドレス宛には、そのままでは送信できません。
SendGrid で RFC 違反のメールを使用する場合は、 **[localpart(@の前) を "" で囲う](https://support.sendgrid.kke.co.jp/hc/ja/articles/360000019762-どういったメールアドレスに送信できますか-) 必要があります。**

具体的には、 `data/class/SC_SendMail.php` の `setTo()` を以下のように修正することで、 送信する瞬間に "" で囲って送信することができるようになります。

```php
    // 宛先の設定
    public function setTo($to, $to_name = '')
    {
        // ↓追加ここから ------------------------------
        $wsp           = '[\x20\x09]';
        $vchar         = '[\x21-\x7e]';
        $quoted_pair   = "\\\\(?:$vchar|$wsp)";
        $qtext         = '[\x21\x23-\x5b\x5d-\x7e]';
        $qcontent      = "(?:$qtext|$quoted_pair)";
        $quoted_string = "\"$qcontent*\"";
        $atext         = '[a-zA-Z0-9!#$%&\'*+\-\/\=?^_`{|}~]';
        $dot_atom      = "$atext+(?:[.]$atext+)*";
        $local_part    = "(?:$dot_atom|$quoted_string)";
        $domain        = $dot_atom;
        $addr_spec     = "{$local_part}[@]$domain";

        $dot_atom_loose   = "$atext+(?:[.]|$atext)*";
        $local_part_loose = "(?:$dot_atom_loose|$quoted_string)";
        $addr_spec_loose  = "{$local_part_loose}[@]$domain";

        $regexp = "/\A{$addr_spec}\z/";
        if (!preg_match($regexp, $to)) {
            $to = preg_replace('/^(.*)@(.*)$/', '"$1"@$2', $to);
        }
        // ↑追加ここまで ------------------------------
        if ($to != '') {
            $this->to = $this->getNameAddress($to_name, $to);
            $this->setRecip('To', $to);
        }
    }
```

以下はパッチです(開発者向け)

```diff

diff --git a/data/class/SC_SendMail.php b/data/class/SC_SendMail.php
index 53b24281e..4c74759cd 100644
--- a/data/class/SC_SendMail.php
+++ b/data/class/SC_SendMail.php
@@ -74,6 +74,26 @@ class SC_SendMail
     // 宛先の設定
     public function setTo($to, $to_name = '')
     {
+        $wsp           = '[\x20\x09]';
+        $vchar         = '[\x21-\x7e]';
+        $quoted_pair   = "\\\\(?:$vchar|$wsp)";
+        $qtext         = '[\x21\x23-\x5b\x5d-\x7e]';
+        $qcontent      = "(?:$qtext|$quoted_pair)";
+        $quoted_string = "\"$qcontent*\"";
+        $atext         = '[a-zA-Z0-9!#$%&\'*+\-\/\=?^_`{|}~]';
+        $dot_atom      = "$atext+(?:[.]$atext+)*";
+        $local_part    = "(?:$dot_atom|$quoted_string)";
+        $domain        = $dot_atom;
+        $addr_spec     = "{$local_part}[@]$domain";
+
+        $dot_atom_loose   = "$atext+(?:[.]|$atext)*";
+        $local_part_loose = "(?:$dot_atom_loose|$quoted_string)";
+        $addr_spec_loose  = "{$local_part_loose}[@]$domain";
+
+        $regexp = "/\A{$addr_spec}\z/";
+        if (!preg_match($regexp, $to)) {
+            $to = preg_replace('/^(.*)@(.*)$/', '"$1"@$2', $to);
+        }
         if ($to != '') {
             $this->to = $this->getNameAddress($to_name, $to);
             $this->setRecip('To', $to);
```

SendGrid 以外にも、 @より前の部分を "" で囲うことにより、送信できるサーバーもあると思います。
もし RFC 違反のメールアドレスが送信できないトラブルが発生した場合はお試しください。
