https://www.amazon.co.jp/Computer-Systems-Programmers-Perspective-3rd-dp-013409266X/dp/013409266X

# 2. Running Programs on a System

## Ch.7 Linking

### 7.2 Static Linking

Static linker (e.g. LD) の仕事は、複数の Relocatble object file を入力として受け取り、１つの Executable object file を出力として作成すること。
Executable を作るための linker が行う処理：

1. Symbol resolution<br>
  シンボルとは、関数、グローバル変数、static 変数のこと。オブジェクトファイルはシンボルの定義、またはシンボルの参照を含む。"Symbol resolution" の目的は、各シンボルの参照を１つのシンボルの定義に紐付けること。
2. Relocation

### 7.3 Object Files

Object file の種類：

1. Relocatable object file
2. Executable object file
3. Shared object file

### 7.6 Symbol Resolution

### 7.7 Relocation

Relocation 時の処理２ステップ：

1. Relocating sections and symbol definition
    1. 複数の Relocatable object file の各セクションをマージして、Executable の各セクションを作成する。イメージ図は以下。
    2. 各セクション、各シンボル定義に、実行時のメモリアドレスを割り当てる。
2. Relocating symbol references within sections
    1. コードおよびデータセクション内の全てのシンボル参照を、実行時のアドレスに修正する。

1-1 のイメージ図：
<img src="https://i.imgur.com/oxkQLcZ.jpg" width="500">

### 7.9 Loading Executable Object Files

Loader はプロセスのメモリイメージを作る過程で、
Executable ファイルの中の、Code segment と Data segment をメモリにコピーする。

<img src="https://i.imgur.com/qOVL17r.jpg" width="500">

### 7.12 Position-Independent Code (PIC)

Linker が Code segment を修正することなく、共有ライブラリをメモリ上のどこにでもロードできるようにする。
このように Relocation なしにロード可能なコードを PIC という。

Global-Offset Table (GOT)。グローバル変数への PIC 参照を作るために、コンパイラが作るテーブル。
これは、オブジェクトをメモリ上のどこにロードしても、データセグメントはコードセグメントから同じ距離にある、という事実に基づく。