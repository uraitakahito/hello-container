# vnc — コンテナの中で GUI を動かして VNC で覗く

画面のない（headless な）コンテナの中で GUI アプリを動かし、ホストの macOS から
その画面を VNC（あるいはブラウザの noVNC）で覗きます。

> まだ導入していない場合は、先に **[../INSTALL.md](../INSTALL.md)** を実施してください。

## このトピックの「うまみ」

Apple Container では、**コンテナ＝軽量 VM がそれぞれ固有の IP を持つ**ので、
`-p` を書かなくても **その IP へ直接接続**できます。

```console
# Docker（参考）: -p で localhost へ公開しないと届かない
% docker run ... -p 5901:5901 -p 6080:6080 ...
% open vnc://localhost:5901

# Apple Container: -p なしで IP に直結できる
% container run -d --rm --init --name vnc-container vnc-image
% open vnc://192.168.64.76:5901        # IP は container ls の IP 列
```

## 1. 仕組み

Linux の GUI は **X11** で動きます。アプリは「X サーバ」に描画を依頼し、X サーバが画面へ描きます。
コンテナに物理ディスプレイは無いので、**Xvfb**（メモリ上だけの仮想画面）を X サーバとして立て、
その画面を **x11vnc** が VNC で配信し、**websockify + noVNC** がブラウザ向けに変換します。

```text
xeyes（GUI アプリ）
   │ 描画を依頼
   ▼
Xvfb :1（仮想ディスプレイ）        ← 画面のない X サーバ
   │ 画面を取得
   ▼
x11vnc ───────────────▶ :5901     ← ネイティブ VNC クライアント用
   │ WebSocket に変換
   ▼
websockify + noVNC ───▶ :6080     ← ブラウザ用（クライアント不要）
```

**fluxbox** は軽量なウィンドウマネージャで、窓に枠を付けて移動・操作できるようにします。

## 2. ビルドと起動

```console
% container system start   # 初回のみ：サービス起動（INSTALL.md 参照）
% cd vnc
% container build -t vnc-image .
% container run -d --rm --init --name vnc-container vnc-image
% container ls             # IP 列にこのコンテナ固有の IP が出る
ID             IMAGE             OS     ARCH   STATE    IP
vnc-container  vnc-image:latest  linux  arm64  running  192.168.64.76/24
```

## 3. 接続する

### A. IP 直結（Apple Container らしいやり方）

```console
% open vnc://192.168.64.76:5901        # ネイティブ VNC（画面共有）
```

ブラウザなら `http://192.168.64.76:6080/vnc.html` を開いて `Connect` を押します
（VNC クライアント不要）。枠付きの xeyes（目玉がカーソルを追う）と
fluxbox のタスクバーが見えれば成功です。

> IP をコマンドで取るなら:
>
> ```console
> % container inspect vnc-container | grep -o '"ipv4Address" *: *"[^"]*"'
> ```

### B. `-p` で localhost 公開（Docker と同じ体験）

`-p` も使えます。Docker に慣れた手にはこちらが馴染みます。

```console
% container run -d --rm --init -p 5901:5901 -p 6080:6080 --name vnc-container vnc-image
% open vnc://localhost:5901
```

両経路は排他ではありません。`-p` を付けても IP 直結は引き続き使えます。

> **`-p` の待ち受けは既定で `0.0.0.0`（全インターフェース）です**（実測）。
> 本教材の x11vnc はパスワード無しなので、`-p 127.0.0.1:5901:5901` のように
> ループバックへ限定する書き方も覚えておくと安全です。

## 4. `--init` と停止

1 つのコンテナで複数プロセス（Xvfb / fluxbox / xeyes / websockify / x11vnc）を動かすため、
`--init`（「シグナルを転送しプロセスを回収する init を挟む」公式フラグ）を付けています。

`container stop` の既定猶予は **5 秒**です（Docker は 10 秒）。実測では:

- `--init` あり → 約 3 秒。init が SIGTERM を**全プロセスへ転送**し、Xvfb まで順に終了
- `--init` なし → 約 0.2 秒。PID 1 の x11vnc だけが SIGTERM で終了し、残りは VM ごと消える

どちらも 5 秒の SIGKILL には達しません（x11vnc が SIGTERM を処理するため）。
`--init` の本当の役目は停止の速さではなく、**全プロセスへの秩序だった終了通知と、
実行中のゾンビプロセス回収**です。

## 5. セキュリティ（重要）

学習用の割り切りで、x11vnc は**パスワード無し（`-nopw`）**です。
コンテナ IP（`192.168.64.0/24`）に届くのはこの Mac だけですが、
`-p` を全インターフェースで公開すれば LAN からも届きます。共有ネットワークでは使わず、
必要なら `x11vnc -usepw` や `-p 127.0.0.1:...` への限定を使ってください。

## 6. 後片付け

```console
% container stop vnc-container   # --rm 付きなので停止と同時に削除される
% container system stop          # サービスごと止める場合
```
