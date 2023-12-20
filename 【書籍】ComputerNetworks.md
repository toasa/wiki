https://www.amazon.co.jp/Computer-Networks-Global-Andrew-Tanenbaum/dp/1292374063

### 1.6.3 

OSI モデルが流行らなかった原因は Timing, Desing, Implementation, Politics が悪かった。

- Timing の悪さ。仕様が受け入れられるにはタイミングが重要。アカデミックな研究は成熟したが、市場の製品としては出回っていないタイミング（いわゆる "Apocalypse of the two elephants" の象の間のタイミング）が望ましい。OSIモデルが現れた頃は、すでに TCP/IP が広く使われていた。
- Design の悪さ。綺麗にレイヤが分かれていない。セッション層とプレゼンテーション層はほぼ空で、データリンク層とネットワーク層は大き過ぎる。別の問題点として、同じような機能が異なるレイヤで重複して定義されている。
- Implementation の悪さ。
- Polictics の悪さ。