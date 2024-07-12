---
marp: true
math: mathjax
theme: katas
---
<!-- 
size: 16:9
paginate: true
-->
<!-- header: 勉強会# ― エンジニアとしての解像度を高めるための勉強会-->

# 読みやすいコードの作り方 - 状態(2)

_Code Readability_

---

## タネ本

### 『読みやすいコードのガイドライン<br>　 持続可能なソフトウェア開発のために』

- 石川宗寿(著)
- 技術評論社 2022/11/4 初版

![bg right:30% 90%](assets/12-book.jpg)

---

## 状態とは？ (recap)

プログラムの振る舞いを決定するデータとその組み合わせ。

1. 変数の値（値の変化＝状態の変化）
2. オブジェクトの状態（メンバー変数の変化＝状態の変化）
3. プログラムや処理のフロー（状態変数の変化＝状態の変化）
4. 処理の状態（処理状況の変化＝状態の変化）

フラグ変数が５個あるだけで32($=2^5$)種類もの状態が存在することになる

<!--
突き詰めて言うと、プログラムは入力がまったく同じである場合は同じように動き、一方で入力のほんの一部でも異なっていれば異なる動きをする(ことがある)。
これはつまりそのプログラムが「変化しうる変数や入力情報のすべての組み合わせからなる状態数」を持っているということになる
-->

---

## 複雑な状態への対処方法

1. 変数の**直交性**を意識する
    * **手法1: 関数への置き換え**  ← ｲﾏｺｺ
    * **手法2: 直和型での置き換え**
2. 状態の**遷移**を設計する
    * 不変性
    * 冪(べき)等性
    * 非巡回

![bg right:30% 90%](assets/12-book.jpg)

<!-- この本ではどのような点に注意すると良いと言っているか -->

---

## （説明のねらい）

* <b>状態数が多いほどプログラムは複雑になり不具合も生まれやすい</b>
    * とはいえ絶対に避けることは出来ない
* <b>プログラミングにおいて、人間が管理できることには限界がある</b>
    * 安全に管理する処理を都度書いても、どこかで実装漏れが出てしまう
    * 問題発見がに遅れてしまうことがある(リリース後とか！)

↓ だから…

* <b>状態そのものの数を減らしたい</b>
* <b>安全に管理できている状態をクラス設計時点から作り出したい</b>
    * 異常な状態をコードとしてそもそも書けないようにする
    * なんならコンパイル時点で異常をエラーにしたい

<!-- (状態シリーズを自分ごとにするために) -->
<!-- プログラミング技法が
　「機械語(何でもあり)→構造化プログラミング(goto禁止)→オブジェクト指向プログラミング(情報隠蔽etc.)→関数型プログラミング(冪等性)」
と制限を増やす方向で進化していったように、より制限のある環境を作り出すことで安全なソフトウェアを実現するというのが、状態シリーズの説明を理解するうえでのポイントです。 -->


---

## 非直交 = 不具合の温床

![bg right:30% 90%](assets/12-book.jpg)

なんとかしなければならない。

↓

設計として非直交を回避するにはどうしたら良いか？

> * **手法1**: 関数への置き換え
> * **手法2**: 直和型での置き換え

<!-- 今の２つの例で見たものは、これらのデータを扱うソースコード側で気をつけるようにすれば不具合を紛れ込ませないで済む。
でもそれを将来に渡って絶対に守り続けることはできると言えますか？私は罪を犯したことがないと女性に石を投げ続けることができますか -->

---

## 手法1: 関数への置き換え

複数の状態から「**従属**の関係」を導き出し、従属している状態を関数で置き換える

```cpp
/* GOOD */
struct OwnedDisplayModel {
    // コイン所持量
    int ownedCoins;
    // コイン所持量を表示するときの文字列を返す
    std::string getOwnedCoinText() {
        /* ownedCoinsを使い、カンマ区切り・単数形/複数形を考慮した文字列を作成する */
    }
};
```

→ ownedCoinTextという状態が減り、矛盾した状態を表現できないようになった

---

## 手法1: 関数への置き換え

複数の状態から「**従属**の関係」を導き出し、従属している状態を関数で置き換える

```cpp
/* GOOD */
struct OwnedDisplayModel {
    const int ownedCoins;
    const std::string ownedCoinText;

    OwnedDisplayModel(int coins) {
        this.ownedCoins = coins;
        this.ownedCoinText = /* 文字列生成処理 */
    }
};
```

→ コンストラクタで生成して書き換えられないようにすることで、状態を減らすことができた

<!-- そのほかにもコンストラクタをprivateにしておき、ファクトリーメソッドを用意してあげることで不正な作成方法を回避する方法もある -->

---

## 手法2: 直和型での置き換え

```cs
/* 「ContactはEメール,住所のどちらかを持っていなければならない」を表したクラス */

/* BAD */
struct Contact {
    var name: String
    var emailContactInfo: String?  // どうやって「どちらか１つは非null」を
    var postalContactInfo: String? // 安全に実現したら良い？
};
```

<!-- ２つの値が従属の関係にないときは、値を関数に置き換える方法は使えません。片方がnullであったとしても、もう片方が非nullであることを保証できません

この例のような直交でも従属でもない関係に対しては、直和型での置き換えを検討します -->
