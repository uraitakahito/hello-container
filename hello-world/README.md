# hello-world — はじめてのコンテナ

Apple Container をインストールした後の「最初の一歩」です。
既製イメージで Hello World を動かし、自作イメージをビルドして実行し、基本的なライフサイクル操作を体験します。

> まだ導入していない場合は、先に **[../INSTALL.md](../INSTALL.md)** を実施してください。

## 0. 前提

```console
% container system start   # サービス起動（初回はカーネル導入。INSTALL.md 参照）
% container ls -a          # 空の一覧が返ればOK
ID  IMAGE  OS  ARCH  STATE  IP
```

## 1. 既製イメージで Hello World

Docker Hub の `alpine` イメージを取得して `echo` を実行します。
`--rm` を付けると、実行後にコンテナは自動的に削除されます。

```console
% container run --rm docker.io/library/alpine echo 'Hello, Apple Container!'
Hello, Apple Container!
```

> Apple Container の既定レジストリは Docker Hub です。`alpine` と未修飾でも解決しますが、
> 教材では出所が分かるように `docker.io/library/...` を明示しています。
>
> **`!` を含む文字列はシングルクォート `'...'` で囲みます。** macOS の既定シェル zsh では、
> ダブルクォートだと末尾の `!"` が履歴展開の特殊記法と解釈されて閉じ引用符ごと消え、
> `echo "...!"` が `dquote>`（クォート閉じ待ち）で止まってしまうためです。

## 2. 対話シェルで「軽量VM」を体感する

`-it`（= `--interactive --tty`）でコンテナ内のシェルに入ります。

```console
% container run -it --rm docker.io/library/alpine sh
/ # uname -a
Linux ... aarch64 Linux        # 専用カーネルで動く Linux 環境
/ # exit
```

Apple Container では、**コンテナごとに専用の軽量 Linux VM** が起動します
（Docker Desktop の単一共有VMとは異なります）。各コンテナは固有の IP を持ちます。

## 3. 自作イメージをビルドして動かす

このディレクトリの [Dockerfile](./Dockerfile) は、メッセージを表示するだけの最小イメージです。

```dockerfile
FROM docker.io/library/alpine:3.24
CMD ["echo", "Hello from my own Apple Container image!"]
```

ビルドして実行します。

```console
% cd hello-world
% container build -t hello-apple .
% container image ls
NAME         TAG     DIGEST
hello-apple  latest  ...
% container run --rm hello-apple
Hello from my own Apple Container image!
```

## 4. 基本のライフサイクル

バックグラウンド（`-d` = `--detach`）で簡単な Web サーバを起動し、操作してみます。

```console
% container run -d --name web docker.io/library/python:alpine python3 -m http.server 80
% container ls                       # 稼働中のコンテナと固有IP
% container logs web                 # ログを見る
% container exec -it web sh          # 中に入る
% container stats --no-stream web    # リソース使用量
% container stop web                 # 停止
% container rm web                   # 削除
```

各コンテナは固有の IP を持つので、ブラウザから `http://<IP>` で直接アクセスできます
（`container ls` の `IP` 列を参照）。

## 5. 後片付け

```console
% container system stop              # サービスを止める
```

---

次の教材：**[../git-ssh/](../git-ssh/)** — コンテナ内から GitHub へ SSH で `git clone` する。
