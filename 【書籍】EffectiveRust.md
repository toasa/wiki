https://www.amazon.co.jp/Effective-Rust-Specific-Ways-Improve/dp/1098151402

---

# Ch.1 Types

## Item 1: Use the type system to express your data structures

Rust でよく使われる型の紹介。中でも `enum` 型は推し：

> This brings us to the jewel in the crown of Rust’s type system, the enum. 

型を設計するときの指針：

> This small example illustrates a key piece of advice: make invalid states inexpressible in your types.

## Item 2: Use the type system to express common behavior

メソッドは `struct` だけでなく `enum` に対しても定義できる。確かに。