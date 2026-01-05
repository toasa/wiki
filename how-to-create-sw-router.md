## 概要

Linux の Network Namaspace と veth を使い、
ソフトウェアルータを作成する。

1. ステップ1：最小構成（2つのホストを直結）
2. ステップ2：ルータの導入（3つのNamespace）
3. ステップ3：ルーティングの設定

## ステップ1：最小構成（2つのホストを直結）

### 構築

まず `host-a` と `host-b` という名前のNamespaceを作成する：

```bash
# Network Namespaceの作成
$ sudo ip netns add host-a
$ sudo ip netns add host-b

# 確認（リストに表示されればOK）
$ ip netns list
```

次に veth ペアを作成する (veth-a <--> veth-b)
ここでは `veth-a` と `veth-b` という両端を持つケーブルを作る

```bash
# vethペアの作成
$ sudo ip link add veth-a type veth peer name veth-b

# まだホストOS上にある（確認）
$ ip link show | grep veth
```

vethペアのそれぞれを namespace に所属させる：

```bash
# Namespaceへインターフェースを移動
$ sudo ip link set veth-a netns host-a
$ sudo ip link set veth-b netns host-b
```

IPアドレスの設定とリンクアップを行う。
アドレスは、同一サブネット（192.168.1.0/24）にする：

```bash
# host-a の設定 (192.168.1.1)
$ sudo ip netns exec host-a ip addr add 192.168.1.1/24 dev veth-a
$ sudo ip netns exec host-a ip link set veth-a up

# host-b の設定 (192.168.1.2)
$ sudo ip netns exec host-b ip addr add 192.168.1.2/24 dev veth-b
$ sudo ip netns exec host-b ip link set veth-b up
```

### 動作確認

`host-a` から `host-b` に向かって ping を飛ばす：

```bash
$ sudo ip netns exec host-a ping 192.168.1.2
```

## ステップ2：ルータの導入（3つのNamespace）

ステップ1では「同じネットワーク（192.168.1.0/24）」だったが、
今回は 「異なるネットワーク（サブネット）同士を繋ぐ」構成を作成する。
これがルータの本来の役割。

間に `router` というNamespaceを挟み、左右で異なるIP帯域を使用する。
構成図は以下：


```text
      [Network A: 10.0.1.0/24]             [Network B: 10.0.2.0/24]
+------------+           +----------------------+           +------------+
|   host-a   |           |        router        |           |   host-b   |
| (10.0.1.2) |----veth---| (10.0.1.1)(10.0.2.1) |---veth----| (10.0.2.2) |
+------------+           +----------------------+           +------------+
      gw:10.0.1.1                                     gw:10.0.2.1

```

### 構築

前の設定が残っていると衝突するので、一度リセットしてから始める。

```bash
$ sudo ip netns del host-a
$ sudo ip netns del host-b
# vethペアはNamespace削除と共に自動的に消える
```

Namespaceの作成（3つ）：

```bash
$ sudo ip netns add host-a
$ sudo ip netns add host-b
$ sudo ip netns add router
```

veth ペアの名前以下のようにする：

* `veth-a` (host-a側) <---> `veth-r-a` (routerのhost-a側)
* `veth-b` (host-b側) <---> `veth-r-b` (routerのhost-b側)

vethペアを２組作成し、Namespaceへ移動する：

```bash
# host-a と router を繋ぐケーブル
$ sudo ip link add veth-a type veth peer name veth-r-a
# host-b と router を繋ぐケーブル
$ sudo ip link add veth-b type veth peer name veth-r-b

# Namespaceへの移動
$ sudo ip link set veth-a netns host-a
$ sudo ip link set veth-r-a netns router

$ sudo ip link set veth-b netns host-b
$ sudo ip link set veth-r-b netns router
```

サブネットを分けてIPアドレスを設定する。
10.0.1.0/24用：

```bash
# --- Host-A  ---
$ sudo ip netns exec host-a ip addr add 10.0.1.2/24 dev veth-a
$ sudo ip netns exec host-a ip link set veth-a up

# --- Router (Host-A側) ---
$ sudo ip netns exec router ip addr add 10.0.1.1/24 dev veth-r-a
$ sudo ip netns exec router ip link set veth-r-a up
```

10.0.2.0/24用：

```bash
# --- Router (Host-B側) ---
$ sudo ip netns exec router ip addr add 10.0.2.1/24 dev veth-r-b
$ sudo ip netns exec router ip link set veth-r-b up

# --- Host-B ---
$ sudo ip netns exec host-b ip addr add 10.0.2.2/24 dev veth-b
$ sudo ip netns exec host-b ip link set veth-b up
```

### 実験（まだ繋がらない）

この状態で `host-a` から `host-b` へ ping を打つ：

```bash
$ sudo ip netns exec host-a ping 10.0.2.2
```

おそらく疎通しない。
なぜなら、フォワーディングの設定と相手への行き方（デフォルトゲートウェイ）を知らないため。

## ステップ3：ルーティングの設定

### 構築

PC（host-a, host-b）は、自分の知らないネットワーク（相手側）への行き方を知らない。
「わからなかったらRouterへ送る」という設定をする。

```bash
# host-a のデフォルトゲートウェイを router (10.0.1.1) に設定
$ sudo ip netns exec host-a ip route add default via 10.0.1.1

# host-b のデフォルトゲートウェイを router (10.0.2.1) に設定
$ sudo ip netns exec host-b ip route add default via 10.0.2.1
```

IPフォワーディングの有効化する：

```bash
$ sudo ip netns exec router sysctl -w net.ipv4.ip_forward=1
```

### 動作確認

もう一度 ping を打つ。

```bash
$ sudo ip netns exec host-a ping 10.0.2.2
```

今度は通るはず。