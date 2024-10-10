https://www.amazon.co.jp/Rust-Action-Tim-McNamara/dp/1617294551

---

# Ch.2 Language foundations

## 2.3 Numbers

### 2.3.3 Comparing numbers

Floating-point hazards.
浮動小数点数の同値比較は直感に反する結果になる場合がある。
例えば `assert!(0.1 + 0.2 == 0.3);` は自分の実行環境では失敗した。
比較したい場合は[計算機イプシロン](https://ja.wikipedia.org/wiki/%E8%A8%88%E7%AE%97%E6%A9%9F%E3%82%A4%E3%83%97%E3%82%B7%E3%83%AD%E3%83%B3)を使う。以下のテストは成功した：

```rust
let a = 0.1 + 0.2;
let b = 0.3;
assert!(((a - b) as f64).abs() <= f64::EPSILON);
```

## 2.7 Project: Rendering the Mandelbrot set

[マンデルブロ集合](https://github.com/rust-in-action/code/blob/1st-edition/ch2/ch2-mandelbrot/src/main.rs)を書く例が面白い。
決められた描画範囲の各複素数点に対し、
繰り返し乗算を行い発散するかをチェックする。
発散の速度も見ていて、[速度によって点の濃淡を書き分け](https://github.com/rust-in-action/code/blob/1st-edition/ch2/ch2-mandelbrot/src/main.rs#L55-L63)ている。

# Ch.5 Data in depth

## 5.4 Floating-point numbers

単項マイナスはメソッド呼び出しより優先順位が低い。そのため `-1.0_f32.powf(0.0)` は $-(1^0)$ と解釈されてしまう。

# Ch.8 Networking

## 8.5 Ergonomic error handling for libraries

以下２つのコードは戻り値の型が違うのに、どちらもコンパイルが通る：

```rust
fn main() -> std::io::Result<()> {
    let _f = std::fs::File::open("foo.txt")?;
    Ok(())
}
```

```rust
fn main() -> Result<(), std::io::Error> {
    let _f = std::fs::File::open("foo.txt")?;
    Ok(())
}
```

理由は、std::io に Result 型が型エイリアスとして[定義されている](https://doc.rust-lang.org/std/io/type.Result.html)ため：

```
pub type Result<T> = std::result::Result<T, std::io::Error>;
```