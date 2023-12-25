https://www.amazon.co.jp/Computer-Networks-Global-Andrew-Tanenbaum/dp/1292374063

## 1.6 Reference Models

モデルは各レイヤから成る。各レイヤの実装としてプロトコルがある。
例えば、TCP/IP reference モデルは Application, Transport, Internet, Link レイヤから成る。
例えば、Transport レイヤのプロトコルとして TCP, UDP がある。

### 1.6.3

OSI モデルが流行らなかった原因は Timing, Desing, Implementation, Politics が悪かった。

- Timing の悪さ。仕様が受け入れられるにはタイミングが重要。アカデミックな研究は成熟したが、市場の製品としては出回っていないタイミング（いわゆる "Apocalypse of the two elephants" の象の間のタイミング）が望ましい。OSIモデルが現れた頃は、すでに TCP/IP が広く使われていた。
- Design の悪さ。綺麗にレイヤが分かれていない。セッション層とプレゼンテーション層はほぼ空で、データリンク層とネットワーク層は大き過ぎる。別の問題点として、同じような機能が異なるレイヤで重複して定義されている。
- Implementation の悪さ。モデル、プロトコルの悪さから実装も巨大に。"Everyone who tried them got burned"。その後時間とともに実装は良くなったが、"Once people think something is bad, its goose is cooked."。
- Polictics の悪さ。

### 1.6.5

OSI モデルの強みはモデルそのもの（ただし Presentaion レイヤと Session レイヤは除く）。
TCP/IP モデルの強みはプロトコルが広く使われていること。
なので本書では両者の良いとこどりをした新たなモデルを使う。
モデルは、下から Physical, Link, Network, Transport, Application レイヤから成る。

## 1.7

De facto standard と対になる De jure standard という用語を知る。
意味は De facto の反対。つまり公的に標準化された規格のこと。

### 1.8.3

フィッシングサイトのフィッシング、Fishing と思っていたが、Phishing らしい。
語源は諸説あるらしい：https://ja.wikipedia.org/wiki/%E3%83%95%E3%82%A3%E3%83%83%E3%82%B7%E3%83%B3%E3%82%B0_(%E8%A9%90%E6%AC%BA)#%E8%AA%9E%E6%BA%90

### 1.8.4

Browser fingerprinting を知らなかったので以下のサイトで学ぶ：

https://www.saitolab.org/fp_site/

> Webサイト管理者はアクセスしてきた利用者のブラウザの種類、画面解像度、プラグインの名前やインストール済みフォントの一覧などを取得することができます。これらの情報は利用者ごとに微妙に異なるので、利用者を識別するための特徴点となります。 1つの特徴点だけに注目した場合、同じブラウザや同じ画面解像度を利用している人が複数存在するので利用者の識別は難しいですが、複数の特徴点を組み合わせた場合、すべての特徴点が一致することは稀なので利用者の識別に利用できるとされています。このように利用者の識別につながる特徴点の集合をFingerprintと呼びます。

## 1.10

各章の Overview。
3章のBluetooth, 7章の email のプロトコル、CDN サービスが興味ある

## 2.1.5

光ファイバの物理的特性からくる転送レートの限界は50Tbps。
現在の光ファイバは 100Gbps ほどで、そのボトルネックは電気と光の変換のところにある。
CPUのクロック周波数が物理的限界に近づいていることとは対称的。
[APN](https://www.rd.ntt/iown/0002.html) だな、将来は。