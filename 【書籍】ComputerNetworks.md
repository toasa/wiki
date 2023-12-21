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