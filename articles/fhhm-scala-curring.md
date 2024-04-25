---
title: "カリー化(Scala)のメモ"
emoji: "⚡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - "scala"
published: true
---

```scala:
def asyncFunc()(implicit ec: ExecutionContext): Future[Unit] = {
  ...
}
```
`(implict ec: ExecutionContext)`がよく分からないので、まずはカリー化から勉強してみる。

# カリー化
## 構文
`合計金額 = 単価 * 個数`を計算する下記の関数について考える。
```scala
def calcTotal(price: Int, qty: Int): Int = {
  price * qty
}

total:Int = calcTotal(100, 3)  // -> total: 300
```

これをカリー化すると、↓のようになる。
```scala
def calcTotalCurried(price: Int): Int => Int = {
  qty => price * qty
}

total: Int = calcTotalCurried(100)(3)  // -> total: 300
```
- `Int => Int`の部分は、戻り値が**Int型の変数を受け取り、Int型を返す関数**であることを表している

`calcTotalCurried()`は↓のようにも書ける。(↓の方が一般的)
```scala
def calcTotalCurried(price: Int)(qty: Int): Int = {
  price * qty
}
```

---

引数が3つの場合も同様に書ける。
- notカリー化
```scala
def calcTotal(price: Int, qty: Int, taxRate: BigDecimal): Int = {
  val beforeTax: BigDecimal = BigDecimal(price * qty)

  (beforeTax + (beforeTax * taxRate))
    .setScale(0, BigDecimal.RoundingMode.DOWN)
    .toInt
}
```

- カリー化
```scala
def calcTotalCurried(price: Int): Int => BigDecimal => Int = {
  qty => {
    taxRate => {
      val beforeTax: BigDecimal = BigDecimal(price * qty)

      (beforeTax + (beforeTax * taxRate))
        .setScale(0, BigDecimal.RoundingMode.DOWN)
        .toInt
    }
  }
}
```
```scala
def calcTotalCurried(price: Int)(qty: Int)(taxRate: BigDecimal): Int = {
  val beforeTax: BigDecimal = BigDecimal(price * qty)

  (beforeTax + (beforeTax * taxRate))
    .setScale(0, BigDecimal.RoundingMode.DOWN)
    .toInt
}
```

## 部分適用
いくつかの引数を設定した関数を事前に受け取り、残りの引数を後から受け取る関数を作成すること。  
具体的には↓のようなもの。
```scala
// カリー化された関数
def calcTotal(price: Int)(qty: Int): Int = price * qty

// 単価を部分適用
// 食べ物はどれでも450円、飲み物はどれでも250円
val calcTotalFood = calcTotal(450) _
val calcTotalDrink = calcTotal(250) _

// それぞれのテーブルについて、食べ物、飲み物の数に応じた料金を計算
val priceTableA: Int = calcTotalFood(3) + calcTotalDrink(5)
val priceTableB: Int = calcTotalFood(2) + calcTotalDrink(2)
```

## メリット
- 呼び出し方がシンプルで分かりやすくなる
  - ↓に比べてカリー化、部分適用を使ったほうがシンプル
    - 多分実際は部分適用した関数をimportするだろうから↑はよりシンプルになるはず
```scala
def calcTotal(price: Int, qty: Int): Int = price * qty

val foodPrice: Int = 450
val drinkPrice: Int = 250

val priceTableA: Int = calcTotal(foodPrice, 3) + calcTotal(drinkPrice, 5)
val priceTableB: Int = calcTotal(foodPrice, 2) + calcTotal(drinkPrice, 2)
```

# 参考
- https://qiita.com/Yametaro/items/99cc1c8ebcfc703b1410
