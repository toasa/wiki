https://www.oreilly.co.jp/books/9784814400850/

---

# 2章 ELF Hack

## \#25 procfs / sysfs の基本を把握する

* `proc/<PICおよびTID>`
  * `exe`: 各プロセスの実行ファイルへのシンボリックリンク
  * `cmdline`: プロセスが実行されたコマンドラインを保存
  * `environ`: プロセスが起動された時に渡された環境変数を取得
  * `fd/`: 現在プロセスが開いているファイルディスクリプタの一覧

pgrep コマンドで実行中のプロセスをリスト表示できる。

```
$ pgrep -u root -l # -u オプションでユーザを指定、-l でプロセス名も表示
1 systemd
9 orbstack-agent:
125 systemd-journal
153 systemd-udevd
224 systemd-logind
225 cron
245 agetty
264 apache2
```