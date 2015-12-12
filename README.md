# Fantasy Land 仕様書

[![Join the chat at https://gitter.im/fantasyland/fantasy-land](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/fantasyland/fantasy-land?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

(別名 "代数的なJavaScriptの仕様書")

![](logo.png)

このプロジェクトは次のような普遍的な代数的データ構造の互換性を記します

* [Setoid](#setoid)
* [Semigroup](#semigroup)
* [Monoid](#monoid)
* [Functor](#functor)
* [Apply](#apply)
* [Applicative](#applicative)
* [Foldable](#foldable)
* [Traversable](#traversable)
* [Chain](#chain)
* [Monad](#monad)
* [Extend](#extend)
* [Comonad](#comonad)

![](figures/dependencies.png)

## 概略

代数とは、 代数がそれらに対し閉じている値一式もしくは演算子一式と、いくつかの代数が従うべき法則のことです。

Fantasy Land のそれぞれの代数は別々の仕様書となっています。代数は他の実装されている代数に依存している場合があります。代数は他の代数の実装されていなくてもよいメソッドとそのメソッドが新しいメソッドからどのように派生するか述べることがあります。

## 専門用語

1. "value" は下で定義している構造を含むJavaScriptの値を指します。
2. "equivalent"は与えられた値が同等であるかという適切な定義を指します。この定義は二つの値が問題なく抽象性を遵守するプログラム中で入れ替え可能であることを保証するべきです。例えば

    - 二つのリストはすべてのインデックスにおいて等しいならば等しい。
    - 二つのディクショナリーとして解釈されるJavaScriptのプレーンなオブジェクトはすべてのキーにおいて等しいならば等しい。
    - 二つのプロミスは等しい値を生成する時に等しい。
    - 二つの関数は等しい入力に対し等しい値を返す時に等しい。

## 代数

### Setoid

1. `a.equals(a) === true` (反射性)
2. `a.equals(b) === b.equals(a)` (対称性)
3. If `a.equals(b)` and `b.equals(c)`, then `a.equals(c)` (推移性)

#### `equals` メソッド

Setoidを持った値は `equals` メソッドを備えなければなりません。 `equals` メソッドは以下のような1つの引数をとります。

    a.equals(b)

1. `b` は同じSetoidである値でなくてはなりません。
    1. `b` が同じSetoidでない場合、 `equals` の動作は明確ではありません( `false` を返すことが推奨されています)。

2. `equals` はboolean値( `true` もしくは `false`)を返さなくてはなりません。

### Semigroup

1. `a.concat(b).concat(c)` は `a.concat(b.concat(c))` に等しいです(結合性)。

#### `concat` メソッド


Semigroupを持った値は `concat` メソッドを備えなければなりません。`concat` メソッドは以下のような1つの引数をとります。

    s.concat(b)

1. `b` は同じSemigroupの値でなければなりません。

    1. `b` が同じsemigroupでない場合、`concat`の動作は明確ではありません。

2. `concat` は同じSemigroupである値を返さなければなりません。

### Monoid

Monoidの仕様を実装した値はSemigroupの仕様も実装していなければなりません。

1. `m.concat(m.empty())` は `m` に等しいです (右同一性)
2. `m.empty().concat(m)` は `m` に等しいです (左同一性)

#### `empty` メソッド

Monoidを持った値は `empty` メソッドを自身もしくは自身の `constructor`オブジェクトが備えなければなりません。`empty` メソッドは引数をとりません。

    m.empty()
    m.constructor.empty()

1. `empty` は同じMonoidである値を返さなければなりません。

### Functor

1. `u.map(a => a)` は `u` に等しいです (同一性)
2. `u.map(x => f(g(x)))` は `u.map(g).map(f)` に等しいです (合成)

#### `map` メソッド

Functorを持った値は `map`メソッドを備えなければなりません。`map` メソッドは以下のような1つの引数をとります。

    u.map(f)

1. `f` は関数でなければなりません。

    1. `f` が関数でない場合、`map`の動作は明確ではありません。
    2. `f` はどのような値を返してもよいです。

2. `map` は同じFunctorである値を返さなければなりません。

### Apply

Applyの仕様を実装した値はFunctorの仕様も実装していなければなりません。

1. `a.map(f => g => x => f(g(x))).ap(u).ap(v)` は `a.ap(u.ap(v))` に等しいです (合成)

#### `ap` メソッド

Applyを持った値は `ap` メソッドを備えなければなりません。 `ap` メソッドは以下のような1つの引数をとります。


    a.ap(b)

1. `a` は関数のApplyでなければなりません。

    1.  `a` が関数のApplyでない場合、 `ap` の動作は明確ではありません。

2. `b` は何かしらの値のApplyでなければなりません。

3. `ap` はApply `a`の中の関数をApply `b`の中の値に適用しなければなりません。

###Applicative

Applicativeの仕様を実装した値はApplyの仕様も実装していなければなりません。

Applicativeの仕様を満たす値は以下を実装しなくてよいです。

* Functorの `map`; は `function(f) { return this.of(f).ap(this); }` から導くことができます。

1. `a.of(x => x).ap(v)` は `v` に等しいです (同一性)
2. `a.of(f).ap(a.of(x))` は `a.of(f(x))` に等しいです (準同型)
3. `u.ap(a.of(y))` は `a.of(f => f(y)).ap(u)` に等しいです (交換性)




#### `of` メソッド

Applicativeを持った値は `of` メソッドを自身もしくは自身の `constructor`オブジェクトが備えなければなりません。`of` メソッドは以下のような1つの引数をとります。

    a.of(b)
    a.constructor.of(b)

1. `of` は同じApplicativeである値を返さなければなりません。

    1. `b` は確認されるべきではないです(訳者注: 意味がうまく取れなかったので誤訳の可能性があります)。


### Foldable

1. `u.reduce` は `u.toArray().reduce` に等しいです。

* `toArray`; は `function() { return this.reduce((acc, x) => acc.concat(x), []); }` から導くことができます。


#### `reduce` メソッド

Foldableを持った値は `reduce` メソッドを備えなければなりません。 `reduce` メソッドは以下のような2つの引数をとります。

    u.reduce(f, x)

1. `f` は2引数の関数でなくてはなりません。

    1. `f` が関数でない場合、`reduce` の動作は明確ではありません。
    2. `f` の第一引数は `x` と同じ型でなくてはなりません。
    3. `f` は `x` と同じ型の値を返さなければなりません。

1. `x` はリダクションの初めのアキュムレーターの値です。


### Traversable

Traversableの仕様を実装した値はFunctorの仕様も実装していなければなりません。

1. `t` が `f` から `g`への自然な変換である場合、 `t(u.sequence(f.of))` は `u.map(t).sequence(g.of)` に等しいです (自然性)

2. `u.map(x => Id(x)).sequence(Id.of)` は `Id.of` に等しいです (同一性)


3. `u.map(Compose).sequence(Compose.of)` は `Compose(u.sequence(f.of).map(x => x.sequence(g.of)))` に等しいです (合成)


* `traverse`;  は `function(f, of) { return this.map(f).sequence(of); }` から導くことができます。

#### `sequence` メソッド

Traversableを持った値は `sequence` メソッドを備えなければなりません。 `sequence` メソッドは以下のような1つの引数をとります。

    u.sequence(of) 

1. `of` は `u`が含むApplicativeを返さなければなりません。

### Chain

Chainの仕様を実装した値はApplyの仕様も実装していなければなりません。

Chainの仕様を満たす値は以下を実装しなくてよいです。

* Applyの `ap`;  は `function ap(m) { return this.chain(f => m.map(f)); }` から導くことができます。



1. `m.chain(f).chain(g)`  は`m.chain(x => f(x).chain(g))` に等しいです (結合性)


#### `chain` メソッド

Chainを持った値は `chain` メソッドを備えなければなりません。 `chain` メソッドは以下のような1つの引数をとります。

    m.chain(f)

1. `f` は値を返す関数でなければなりません。

    1. `f` が関数でない場合、 `chain` の動作は明確ではありません。
    2. `f` は同じChainである値を返さなければなりません。

2. `chain` は同じChainである値を返さなければなりません。

###Monad


Monadの仕様を実装した値はApplicativeとChainの仕様も実装していなければなりません。

Monadの仕様を満たす値は以下を実装しなくてよいです。

* Applyの `ap`;  は `function(m) { return this.chain(f => m.map(f)); }` から導くことができます。
* Functorの `map`;  は `function(f) { var m = this; return m.chain(a => m.of(f(a)))}` から導くことができます。



1. `m.of(a).chain(f)` は `f(a)` に等しいです (左同一性)
2. `m.chain(m.of)` は `m` に等しいです (右同一性)


### Extend

1. `w.extend(g).extend(f)` は `w.extend(_w => f(_w.extend(g)))` に等しいです


#### `extend` メソッド

Extendは `extend` メソッドを備えなければなりません。 `extend` メソッドは以下のような1つの引数をとります。
     
     w.extend(f)

1. `f` は値を返す関数でなければなりません。


    1. `f` が関数でない場合、 `extend` の動作は明確ではありません。
    2. `f` must return a value of type `v`, for some variable `v` contained in `w`. は 型が `v` の値を返さなければなりません。`v` というのは `w` の中に格納されたいくつかの変数のことです。(訳者注: よく意味が取れなかったので誤訳の可能性があります)

2. `extend` は同じExtendである値を返さなければなりません。



###Comonad

Comonadの仕様を実装した値はFunctorとExtendの仕様も実装していなければなりません。

1. `w.extend(_w => _w.extract())` は `w` に等しいです。
2. `w.extend(f).extract()` は `f(w)` に等しいです。
3. `w.extend(f)` は `w.extend(x => x).map(f)` に等しいです。


#### `extract` メソッド

Comonadを持った値は `extract` メソッドを備えなければなりません。 `extract` メソッドは引数をとりません。
    
    c.extract()

1. `extract` は型が `v` である値を返さなければなりません。`v` というのは `w` に格納されたいくつかの変数です。 (訳者注: よく意味が取れなかったので誤訳の可能性があります)

    1. `v` は `f`が `extend` の中で返す型と同じ型を持たなければなりません。



## Notes

1. 2つ以上のメソッドと法則の実装がある場合、実装は他の使い道のためのラッパーを選び提供するべきです。
2. 明記されたメソッドをオーバーロードすることは推奨されていません。これは壊れ、バグだらけの動作という結果になります。
3. 明記されていない動作の時は例外を投げることが推奨されます。

4. 全てのメソッドを実装した `Id` コンテナは `id.js` 中にあります。
