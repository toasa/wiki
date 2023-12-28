https://www.amazon.co.jp/Fluent-Python-Concise-Effective-Programming/dp/1492056359

# 1. Data Structure

## Ch.1 The Python Data Model

`_` ２つで前後を囲ったメソッドを Special method という
（Dunder method とも。由来は Double underscore before and after から）。

### A Pythonic Card Deck

トランプのデッキを表す自作クラス `FrenchDeck` に、`__len__` と `__getitem__` の Special method を実装するサンプル。
この実装により、Python のビルトイン機能 (e.g. Iteration, Slicing) が使えるようになる。
Python ユーザは、少ないコード記述量で多くの機能が利用できる。
レバレッジが効いて、便利だねという売り文句。
jQuery の Write less, Do more に似た思想。

### How Special Methods Are Used

Special Method の重要な用途４例：

- Emulating numeric types<br>
    例として、自作 Vector クラスに `__add__`, `__mul__` Special method を実装することで、`+`, `*` が使えるようになるよ、というはなし。Rust のトレイト味がある。
- String representation of objects<br>
    `__repr__`を実装する。`__str__` との違いは StackOverflow にあるよ。
- Boolean value of an object<br>
    あるオブジェクトの Boolean 性を判定したい時、`__bool__` が定義されていたらそれを呼び、ダメだったら `__len__` を確認しに行くのは、合理的な気がする。確かに `__len__` が定義されてて、長さが 0 だったらそのオブジェクトは Falsy に思える。
- Implementing collections<br>
    Collection なオブジェクトの継承関係。

### Overview of Special Methods

Python には 80 以上の Special method がある。
算術、ビット、比較演算などの Operator に対応する method は Table1-2 にまとまっている。
Operator 以外の method は Table1-1 にまとまっている。

テーブルを眺める。v1.5 から `@`が導入され行列の積が書ける！ (Method は`__matmul__`) 。

### Why len is not a method?

`len()` の引数の型により、Python インタプリタの挙動が異なる：

1. `list`, `str` などの built-in 型
2. それ以外の型

1\. の場合は Fast-path が用意されており、 `PyVarObject` の `ob_size` フィールドを参照して結果を返す。2. の場合は `__len__` メソッドを実行する。
少なくとも１の場合はメソッド呼び出しではないので、`len()` はメソッドではない（という自分の理解）。