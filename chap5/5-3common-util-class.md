多分悪いコード


```

class GiftPoint {
  private static final int MIN_POINT = 0;
  final int value;

  GiftPoint(final int point) {
    if (point < MIN_POINT) {
      throw new IllegalArgumentException("ポイントが0以上ではありません。");
    }

    value = point;
  }

  /**
   * ポイントを加算する。
   *
   * @param other 加算ポイント
   * @return 加算後の残余ポイント
   */
  GiftPoint add(final GiftPoint other) {
    return new GiftPoint(value + other.value);
  }

  /**
   * @return 残余ポイントが消費ポイント以上であればtrue
   */
  boolean isEnough(final ConsumptionPoint point) {
    return point.value <= value;
  }

  /**
   * ポイントを消費する。
   *
   * @param point 消費ポイント
   * @return 消費後の残余ポイント
   */
  GiftPoint consume(final ConsumptionPoint point) {
    if (!isEnough(point)) {
      throw new IllegalArgumentException("ポイントが不足しています。");
    }

    return new GiftPoint(value - point.value);
  }
}


```

このGiftPointクラスにはポイントの加算メソッドや消費メソッドが定義されており
しかし、実はそうではない



```

GiftPoint standardMemberShipPoint = new (3000)
```


```
GiftPoint premiumMemberShipPoint = new (5000)
```

上記悪いコード
コンストラクタを公開すると様々な用途に使われがち結果関連したロジックが分散し勝ちになりメンテナンスが大変になる
入会ポイントを変更したいと奇異全ソースをチェックしなければならないからという理由です。


リファクタリング




private コンストラクタをしようすること
これはファクトリメソッドという

ファクトリメソッドとは？という説明をわかりやすくしてほしいです。

```package chapter05_encapsulationpractices.gift.v2;

class GiftPoint {
  private static final int MIN_POINT = 0;
  private static final int STANDARD_MEMBERSHIP_POINT = 3000;
  private static final int PREMIUM_MEMBERSHIP_POINT = 10000;
  final int value;

  // 外部からはnewできない。
  // クラス内部でのみnewできる。
  private GiftPoint(final int point) {
    if (point < MIN_POINT) {
      throw new IllegalArgumentException("ポイントが0以上ではありません。");
    }

    value = point;
  }

  /**
   * @return 標準会員向け入会ギフトポイント
   */
  static GiftPoint forStandardMembership() {
    return new GiftPoint(STANDARD_MEMBERSHIP_POINT);
  }

  /**
   * @return プレミアム会員向け入会ギフトポイント
   */
  static GiftPoint forPremiumMembership() {
    return new GiftPoint(PREMIUM_MEMBERSHIP_POINT);
  }
  // 省略
}


```

コンストラクタをprivateにすると内部のみで生成
GiftPointクラスにロジックが集中するので容易に変更がしやすくなる
生成ロジックが増えたらファクトリメソッドを検討すること

```
GiftPoint standardMemberShipPoint = GiftPoint.forStandardMembership();
```


```
GiftPoint premiumMemberShipPoint = GiftPoint.forPremiumMembership();
```