---
title: EC-CUBEで利用しているデザインパターン
tags:
  - PHP
  - EC-CUBE
  - DesignPatterns
  - EC-CUBE4
private: false
updated_at: '2019-12-20T13:21:52+09:00'
id: 0eaf1da92ec88fc24fc8
organization_url_name: ec-cube
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---
一見簡単そうに見えるカートシステムですが、EC-CUBEを実際に開発してみると、とても複雑な処理を実現しているのがわかります。
EC-CUBE4系では、とても複雑な処理を、できるかぎり汎用的にするため、様々なデザインパターンを使用しています。
そのうちの一部をご紹介します。

## [Chain of Responsibility パターン](https://ja.wikipedia.org/wiki/Chain_of_Responsibility_パターン)

エンティティの setter などで使用しています。
各メソッドをチェーンのようにつなげることができます。

```php
        $Customer = new Customer();
        $Customer
            ->setName01($data['name01'])
            ->setName02($data['name02'])
            ->setKana01($data['kana01'])
            ->setKana02($data['kana02'])
            ->setCompanyName($data['company_name'])
            ->setEmail($data['email'])
            ->setPhonenumber($data['phone_number'])
            ->setPostalcode($data['postal_code'])
            ->setAddr01($data['addr01'])
            ->setAddr02($data['addr02']);
```

## [Decorator パターン](https://ja.wikipedia.org/wiki/Decorator_パターン)

商品明細を簡単に扱うため、 `\Doctrine\Common\Collections\ArrayCollection` を拡張した [OrderItemCollection](https://github.com/EC-CUBE/ec-cube/blob/4.0/src/Eccube/Service/Calculator/OrderItemCollection.php) で使用しています。

以下は商品の ProductClass の配列を取得する例です。
(filter メソッドを使ってもいいですが、わかりやすく foreach を使います...)

```php

// OrderItemCollection を使用しない場合
$ProductOrderItem = [];
foreach ($Order->getOrderItems() as $OrderItem) {
    if ($OrderItem->isProduct()) {
        $ProductOrderItem[] = $OrderItem->getProductClass();
    }
}

// OrderItemCollection を使用
// Order::getOrderItems() で OrderItemCollection が返ってくる
$ProductOrderItem = $Order->getOrderItems()->getProductClasses()->toArray();
```

## [State パターン](https://ja.wikipedia.org/wiki/State_パターン)

購入フローの OrderStateMachine で使用しています。

![ステートマシン図](https://user-images.githubusercontent.com/16895409/45342499-251d2d80-b5d9-11e8-9613-a7e037cf62fc.png)

- `OrderStateMachine::can()` で、次のステータスに遷移可能かチェックし
- `OrderStateMachine::apply()` で、注文ステータスを変更します。


```php
$Order = ...;
$NextStatus = $orderStatusRepository->find(OrderStatus::PAID);
// ステータスを遷移させることができるかどうか
if ($orderStateMachine->can($Order, $NextStatus)) {
    // ステータスの遷移
    $orderStateMachine->apply($Order, $NextStatus);
}
```

例えば、**出荷完了後の返品は、在庫を戻さない** といった処理を実現しています。

## [Strategy パターン](https://ja.wikipedia.org/wiki/Strategy_パターン)

PurchaseFlow で使用しています。
カート、購入フロー、注文管理画面で共通して必要な要件は、  **商品の明細を集計して、合計金額を算出する** ことです。
しかし、それぞれ商品の状態が異なります。

- カート
  - 商品の内容、数量は未確定
  - 在庫は確保されていない
  - 送料・手数料は計算しない


- 購入フロー
  - 商品の内容、数量は確定している
  - 在庫は確保されていない
  - 送料・手数料を計算する
  - ポイントを計算する(加算しない・使用する)

- 注文管理画面
  - 商品は購入済み
  - 在庫を確保済み
  - 送料・手数料を計算する
  - ポイントを計算する(ステータスによっては加算済み・使用する)

共通する処理もありますが、それぞれ処理の流れが異なったり、独自の処理が必要になったりします。

PurchaseFlow では、ItemProcessor や ItemValidator など処理を **Strategy** クラスとし、 Context である PurchaseFlow クラスで実行しています。
どのような Strategy を組み合わせるかは、 [purchaseflow.yaml](https://github.com/EC-CUBE/ec-cube/blob/4.0/app/config/eccube/packages/purchaseflow.yaml) に定義しています。

この Strategy パターンの良いところは、プラグインなどで Strategy を作成して、独自の処理を簡単に追加できることです。
処理の順番を変えたい場合も、 purchaseflow.yaml を変更するだけでできます。
また、 Strategy クラス一つ一つは、小さくシンプルなプログラムですので、単体テストも簡単に作成できます。

このように、小さな処理をするクラスを組み合わせて、様々なアルゴリズムに対応するのが Strategy パターンです。
文章だけだと大変わかりづらいですが、 [PurchaseFlow をツリー表示](https://github.com/EC-CUBE/ec-cube/pull/4223) するメソッドをコールすると、ちょっとイメージしやすくなると思います。

例えば、購入フローを処理する **shopping Flow** はこんな感じ

```
├ shopping flow
ItemValidator
│├ Eccube\Service\PurchaseFlow\Processor\DeliverySettingValidator
│├ Eccube\Service\PurchaseFlow\Processor\ProductStatusValidator
│└ Eccube\Service\PurchaseFlow\Processor\PriceChangeValidator
ItemHolderValidator
│├ Eccube\Service\PurchaseFlow\Processor\StockMultipleValidator
│├ Eccube\Service\PurchaseFlow\Processor\SaleLimitMultipleValidator
│├ Eccube\Service\PurchaseFlow\Processor\EmptyItemsValidator
│└ Plugin\Coupon4\Service\PurchaseFlow\Processor\CouponProcessor
ItemPreprocessor
ItemHolderPreprocessor
│├ Eccube\Service\PurchaseFlow\Processor\TaxProcessor
│├ Eccube\Service\PurchaseFlow\Processor\OrderNoProcessor
│├ Eccube\Service\PurchaseFlow\Processor\DeliveryFeePreprocessor
│├ Eccube\Service\PurchaseFlow\Processor\DeliveryFeeFreeByShippingPreprocessor
│├ Eccube\Service\PurchaseFlow\Processor\PaymentChargePreprocessor
│├ Eccube\Service\PurchaseFlow\Processor\TaxProcessor
│└ Plugin\Coupon4\Service\PurchaseFlow\Processor\CouponProcessor
DiscountProcessor
│└ Eccube\Service\PurchaseFlow\Processor\PointProcessor
ItemHolderPostValidator
　├ Eccube\Service\PurchaseFlow\Processor\AddPointProcessor
　├ Eccube\Service\PurchaseFlow\Processor\PaymentTotalLimitValidator
　├ Eccube\Service\PurchaseFlow\Processor\PaymentTotalNegativeValidator
　├ Eccube\Service\PurchaseFlow\Processor\PaymentChargeChangeValidator
　└ Eccube\Service\PurchaseFlow\Processor\DeliveryFeeChangeValidator
```

しかし現在のバージョンでは、処理の流れが見えづらいのは否めず、ステップ実行を駆使しないと開発が難しくなってしまったのが難点ですね。。。
