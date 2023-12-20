https://www.amazon.co.jp/-/en/Michael-Sipser/dp/113318779X

### 2020/07/21

開始

### 2020/07/30

p.74 Claim1.65の途中

### 2020/07/31

p.77 Nonregular languages のイントロの途中。
無限個のstringをもつ言語で、regular なものがあるとあり驚き。
その例が以下らしい。

```
D = { w | w has an equal number of occurrences of 01 and 10 as substrings}
```

### 2020/08/01

p.78
thm1.70 Pumping lemmma の statement だけ読んだ。
ある言語Aが regular であることの必要条件を与える定理。
用途は A が non-regular であることを示したい場合にこの条件が成立しないことを確かめればよい。

### 2020/08/02

Pumping lemma の証明 done.
例題への適用は明日以降。

### 2020/08/03

p.81の途中
ex 1.73
{ 0^n1^1 | n >= 0 } が not regular であることをpumping lemma を使って証明した。
ex 1.74
{ w | w has an equal number of 0s and 1s } が not regular であることをpumping lemma を使って証明した。

### 2020/08/04

{ ww | w ∈ {0,1}* } がnot regular であることを pumping lemmma をつかって示した。
{ 1^(n^2) | n >= 0 } がnot regular であることを pumping lemmma をつかって示した。

### 2020/08/05

第２章に入った。CFGの formal definition をした。

### 2020/08/06

p.106
CFGのデザイン。
p.107
ambiguity な grammar.

ambiguous grammar によってのみ生成される context-free language がある。これらを inherently ambiguous という。

p.109
Definition of Chomsky normal form.

thm2.9の証明done (一応、あまり納得できんかったけど)

### 2020/08/07

CFG を Chomsky normal form に変換する例を見た（全然理解できん）

p.111
2.2
Pushdown automata 🎉

### 2020/08/08

p.113 pushdown automata の formal definition DONE.

Nondeterministic pushdown automata と deterministic pushdown automata とは recognize できる言語に差がある、つまり力が違うことに驚き. 前者のほうがよりrecognize できる言語が多い。（finite automaton の場合は DFA でも NFAでも力は同じ）

本書では CFGと同等の計算能力である Nondeterministic pushdown automata を主に紹介している

p.114
PDA(pushdown automaton) の例(Ex2.14)を読んでいるが全くわからん。なぜこれで {0^n1^n | n >= 0 } がrecongnize できているのだろう？0の個数はどうやってカウントしているのだね。まったく。


### 2020/08/09

PDA(pushdown automaton) の例つづき。あいかわらず δ の遷移表をみてもよくわからない。ただPDAの状態遷移図は読めるようになってきた。よい。ただ未だにスタックを図にして push, pop のたびに中身が増減するよくある図のほうがわかりやすいがね。

ex2.16
{ a^ib^jc^k | i, j, k >= 0 and i = j or i = k } を recognize するようなPDAを作った。 i = j の場合と i = k の場合、どちらにも対応できるような diagram にするため Nondeterministic であることが効いている。（そして実際、Deterministic な PDA ではこの language を recognize できないみたい。Problem2.57. ふーん。）

p.117
thm 2.20
A language is context free if and only if some pushdown automaton recognizes it.


第三章turing machine まではあと40ページ弱だ、ガンバレ。（多くない？）

### 2020/08/10

thm2.20 (CFG と PDA(pushdown automaton) の同値性) を示す。

CFG => PDA は三割ぐらいわかった。ただ記号があまり頭に入ってきてない。うーん証明すっとばしたらあかんかな。あかんか。

### 2020/08/11

lem2.27 PDA => CFG を見た。わからん。原因はおそらく PDA が文字列 w を accept する場合の formal な定義が理解しきれていないからだと思う。ただ、スタックが絡んでくる formal definition わからないんだよね。いったん飛ばすことにする。よってpushdown automata と CFG は同値であることがわかった（証明はすっとばしたけど）

p.125 Non-context-free languages
CFG版の pumping lemma のステートメントだけ読んだ。なんだか段々と難しくなって行ってないか？

### 2020/08/12

CFG版pumping lemmma の証明もすっとばした。（ええんかいな）
Pumping lemma を使っていくつかの言語が CFL(Context-free language) ではないことを示した。例えば以下:

{ a^nb^nc^n | n >= 0 }

これはCFLではない。一方で{ a^nb^n | n >= 0 } はRegular ではなく、CFLである。

### 2020/08/13

Deterministic context-free languages(DCFL) を定義した。DCFLに重要な実用上の応用先がある。それはプログラミング言語のコンパイラのパーサのデザイン。ふーん、そうなんだ。

p.131 最後
DPDA (Deterministic pushdown automaton) に関する議論は幾分テクニカルになりこれまでの節より難しくなる. 本節の残りは三章以降では使わないので、飛ばしたきゃ飛ばして良いよ
=> ありがとうございます！
=> ただ、DCFLはコンパイラのパーサのデザインに関連しているとも書いてあったので、スキップすることは心苦しくもある。もし興味があったりしたら戻って読み直そう
=> いよいよ Part Ⅱ Computability Theory. Ch.3 The Church-Turing Thesis 🎉

### 2020/08/14

3.1 Turing machinses に突入。
p.168
Turing machine を定義した (Def3.3)

turing machine の Configuration

- the current state
- the current tape contents
- the current head location

C1, C2: configuration のとき,
  C1 yields C2 :<=> the turing machine can legally go from C1 to C2 in a single step.

### 2020/08/15

昨日、TM(Turing machine) の現在の計算の状態を表すものとして Configuration を定義した。このConfiguration を使って、あるTMが input w を accept するということを定義した。ざっくりいうと、あるConfiguration の列が存在し、先頭の Configuration は Start Configuration であり、 C_i は C_i+1 を yield し、 最後の Configuration は accept configuration であるというもの。

ある言語が *Turing-recognizable* であるとは、とあるTMが存在しそれを recognize すること。

p.171
A = { 0^(2^n) | n >= 0 } が Turing-recognizable であることを示すために、Aをrecognize するような TM を構築した。state diagram も図として紹介されているのだが、これは超絶技巧に感じる。たしかに矢印をたどると `00` は accept するし、 `000` は reject する. だが、これを1から作れと言われても途方に暮れると思う。まぁ見てる文には面白いが。（対岸の火事的な？）

### 2020/08/16

TM が decide できる言語の例をいくつか勉強した。

- `{ w#w | w ∈ { 0, 1 }* }`
- `{ a^ib^jc^k | ij = k, i, j >= 1}`

とくに２つ目は state diagram を構築せず、文章のみの説明だったのであまり理解できていない.

### 2020/08/17

Turing machine の別のモデルをいくつか考える：

- TM with multiple tape
- TM with nondeterminism
- Enumerator

特に2つはTMの variants と呼ばれる。素TMと上3つは equivalent(証明はすっとばしたけど）

### 2020/08/18

ヒルベルトの23の問題。これは当時未解決だった23個の数学の問題。1900年にヒルベルトが提唱した。その中の第10問題がCSと関係が深い。第10問題とは

整数係数の多変数多項式が根を持つ（つまりディオファントス方程式が解を持つ）か否かを判定できるようなアルゴリズムを求めよ

というもの。この証明には`アルゴリズム`の厳密な定義をする必要がある。アルゴリズムは1936年 Alonzo Church と Alan Turing によって定義された。Church はλ計算でアルゴリズムを定義し、 Turing は `machines` という呼び名でそれを定義した。アルゴリズムの不正確な定義と厳密な定義との架け橋を定義した二人の提唱は `Church-Turing thesis` と呼ばれる。

第10問題は1970年ロシアの数学者ユーリ・マチャセビッチによって否定的に解決された。

### 2020/08/19

チャーチ＝チューリングのテーゼ
「計算が可能な関数とは、その計算を実行できるような有限のアルゴリズムが存在するような関数」というもの（wiki参照).
つまり計算とは何かと、計算の定義を提唱したもの。

p.194
Decidability に突入。邦訳は決定性で良いのかな？より厨二病ぽくなって楽しみ。

### 2020/08/20

p.170 decideとは
TMが取りうる状態として、accept, reject, loop がある。特にTMがいつも accept or reject と決定でき(つまり loop 状態に突入しない）るとき、TMは Decider と呼ばれる。Decider が language を recognize することをその language と decide するともいう。

p.185 <O> について
TM の入力はいつも文字列。だがTMの入力として、多項式、グラフ、文法などのオブジェクトを考慮したいことがある。このときオブジェクト `O` を文字列に encode したものを <O> と書く。

