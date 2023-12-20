### 2020/08/10

この３日ほど strings/replace.go を読んでいる。が全く理解が進んでいない。replacer は全部で５つ。Replacer, genericReplacer, singleStringReplacer, byteReplacer, byteStringReplacer. genericReplacer がトライ木を使っている。Ruiさんの Youtute でトライ木ではなく DFA で置き換え可能と言っていた。一山当ててやろうと目論んでいたがそもそもトライ木のコードがよくわからない。置き換え対象なので分かる必要は無いのだが、やっぱりわかりたい、わからないつらい。
まぁトライ木なんて忘れて自分で考えつくDFAの方法を試してみますかね。やるべきことは makeGenericReplacer の中身をDFAをつくるように置き換えればよいのかな。

```go
// DFA をつくる
makeGenericReplacer(oldnew []string) *genericReplacer;

// 置換後の文字列を w （バッファ）に書く。成功した場合置き換えた文字数？も返す。
WriteString(w io.Writer, s string) (n int, err error);

NewReplacer(oldstr1, newstr1, oldstr2, newstr2, ...);
```

ランニング中に考えたが思いつかなかった。最も簡単な場合「aをbに置き換える」というルールを持つreplacer を作りたい場合でもどのようなDFAをつくればよいのだろう？これ以上はメンタルに悪いとして、別のライブラリに移っても良いかも。

まだ粘っている。goのReplacerは置き換えの優先度があるのね。例えば以下。

```go
	r := strings.NewReplacer("a", "A", "aa", "BB", "aaa", "CCC")
	s := r.Replace("aaaaa") // => "AAAAA"

	r := strings.NewReplacer("aa", "BB", "a", "A", "aaa", "CCC")
	s := r.Replace("aaaaa") // => "BBBBA"

	r := strings.NewReplacer("aaa", "CCC", "aa", "BB", "a", "A")
    s := r.Replace("aaaaa") // =>> "CCCBB"
```

エイホ-コラシック法という文字列探索アルゴリズムがあるようだ。そしてこれはなにやらDFAを作るようだ。調べてみる。
=> これもトライ木かーい。

(伊藤直也さんのブログポスト)[https://naoya-2.hatenadiary.org/entry/20090405/aho_corasick]がわかりやすかった。

archive/zip/struct.goを読んでいる。structって何よと思ったけど、zip規格の様々な定数。file ヘッダのシグネチャなど。

archive/zip/writer.go を読んでいる。zipファイルを作ることが目的。`Writer` メソッドの１つに `CreateHeader` がある。これは zip ファイルのメタデータをつくることができるようだ。そのコメントが面白かった。

```
	// The ZIP format has a sad state of affairs regarding character encoding.
	// Officially, the name and comment fields are supposed to be encoded
	// in CP-437 (which is mostly compatible with ASCII), unless the UTF-8
	// flag bit is set. However, there are several problems:
	//
	//	* Many ZIP readers still do not support UTF-8.
	//	* If the UTF-8 flag is cleared, several readers simply interpret the
	//	name and comment fields as whatever the local system encoding is.
```

### 2020/08/11

time/time.go を読んでいる。計算を簡略化するため前月までの日数をハードコードして保持する。これは便利やね。

```go
// daysBefore[m] counts the number of days in a non-leap year
// before month m begins. There is an entry for m=12, counting
// the number of days before January of next year (365).
var daysBefore = [...]int32{
	0,
	31,
	31 + 28,
	31 + 28 + 31,
	31 + 28 + 31 + 30,
	31 + 28 + 31 + 30 + 31,
	31 + 28 + 31 + 30 + 31 + 30,
	31 + 28 + 31 + 30 + 31 + 30 + 31,
	31 + 28 + 31 + 30 + 31 + 30 + 31 + 31,
	31 + 28 + 31 + 30 + 31 + 30 + 31 + 31 + 30,
	31 + 28 + 31 + 30 + 31 + 30 + 31 + 31 + 30 + 31,
	31 + 28 + 31 + 30 + 31 + 30 + 31 + 31 + 30 + 31 + 30,
	31 + 28 + 31 + 30 + 31 + 30 + 31 + 31 + 30 + 31 + 30 + 31,
}
```

### 2020/08/13

container/heap/heap.go を読んでいる。
heap とはそれぞれのノードが自身のサブツリーの中で最小（または最大の）値を持つ、木構造のこと. [参考](https://ja.wikipedia.org/wiki/%E3%83%92%E3%83%BC%E3%83%97)

heap とは優先度付きキューを実装するための一般的な手段みたい。へーそうなんだ。

idempotent: 冪等性 は久々に聞いた。メソッド up と down は割と読めた。up はヒープ木のノードを適切な位置まで親ノードとswap しながら押し上げるメソッド。逆に down はノードを押し下げるメソッド。

### 2020/08/14

sort/sort.go を読んでいる。
median (中央値) が出てきた。これは例えば5人いた場合年齢の中央値は3番目に年寄りな人。medianOfThree という関数は完全にわかった。3つの値 v1, v2, v3 においてこれらの中央値を v2 にセットするというもの。スワップを最大3回行い実現する。

thresold - しきい値

コメント文でなるほど。
IsSorted reports whether data is sorted. この場合 report という単語を使えばよいのか。

Wolog: Without loss of generality
一般性を失わない（初めてみた）

container/ring/ring.go のコードを読んでいる。
リング、循環リストのこと。
このコードは可読性が高いと思った。

```go
func (r *Ring) Move(n int) *Ring {
	if r.next == nil {
		return r.init()
	}
	switch {
	case n < 0:
		for ; n < 0; n++ {
			r = r.prev
		}
	case n > 0:
		for ; n > 0; n-- {
			r = r.next
		}
	}
	return r
}
```

あと自分のローカルでテストコードが動いたことが地味に感動。今まではライブラリを知る方法はネット検索に引っかかった適当な記事だったが、標準ライブラリを眺めていて良さそうなデータ構造を手元で試してみるってのは、なかなかかっこいい気がする。

### 2020/08/15

unicode/utf8/utf8.go を読んでいる。
ソースコードに加えてWikiで UTF-8 の記事も読んでいる。そこにはいくつか特徴が書かれていた:

- Backward compatibility
  - 1バイトの場合ASCIIと完全に互換性があるのは知っていた。ただWikiではさらに "Moreover, 7-bit bytes (bytes where the most significant bit is 0) never appear in a multi-byte sequence" とある。つまりascii のビット表現はマルチバイト文字には登場しないということ？んなアホな。

### 2020/08/16

net/ip.go
IPv4は4バイト、IPv6は16バイト。IPv4 の先頭に12バイトのプレフィックス `0x0000_0000_0000_0000_0000_FFFF` をつければIPv6に変換できる。

### 2020/08/21

time/time.go

Time構造体の wall フィールドは uint64 型である。最上位ビットは hasMonotonic というフラグで、次の33bitは秒、30ビットはナノ秒を表すようだ。

