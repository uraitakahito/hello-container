# git-ssh — コンテナ内から GitHub へ SSH で git clone する

コンテナの中から、**秘密鍵をコンテナに持ち込まずに** GitHub へ SSH 認証し、
プライベートリポジトリを `git clone` する方法を学びます。

ホスト（macOS）の `ssh-agent` を `container run --ssh` でコンテナへ転送するのがポイントです。

> まだ導入していない場合は、先に **[../INSTALL.md](../INSTALL.md)** を実施してください。

## このトピックの「うまみ」

[hello-docker の git-ssh](https://github.com/uraitakahito/hello-docker/tree/main/git-ssh) では、
Docker Desktop の仮想ソケットを手作業でマウントしていました。

```console
# Docker（参考）：手動で2つ指定が必要だった
-v /run/host-services/ssh-auth.sock:/run/host-services/ssh-auth.sock
-e SSH_AUTH_SOCK=/run/host-services/ssh-auth.sock
```

Apple Container では、これが公式フラグ **`--ssh` のワンフラグ**に集約されます。
`--ssh` は「ホストの ssh-agent ソケットをコンテナにマウントし、`SSH_AUTH_SOCK` を
そのパスに自動設定する」処理をまとめて行います。**ソケットパスを自分で指定する必要はありません。**
コンテナを再起動した際も、ソケットパスの変化に自動で追従します。

> ゲスト内のソケットパスは `--ssh` が自動設定するので、利用者が意識する必要はありません。
> 参考までに、本バージョン（container 1.0.0）の実測値は `/var/host-services/ssh-auth.sock` でした
> （公式 how-to の記載は `/run/host-services/ssh-auth.sock`）。

## 1. ホスト側の準備（macOS）

SSH 鍵の作成と GitHub への公開鍵登録は済んでいる前提です（[GitHub SSH keys](https://github.com/settings/keys)）。
まだなら公開鍵の指紋を次で確認できます。

```console
% ssh-keygen -lf ~/.ssh/id_ed25519.pub
```

最近の Mac では、鍵が ssh-agent に自動で読み込まれ、パスフレーズが Keychain に保存されるよう
`~/.ssh/config` を設定します。

```
Host github.com
  # 接続時に鍵を ssh-agent へ追加する（再起動後に自動追加されるわけではない点に注意）
  AddKeysToAgent yes
  # パスフレーズを macOS の Keychain に保存する
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

秘密鍵を ssh-agent に追加し、パスフレーズを Keychain に保存します。

```console
% ssh-add --apple-use-keychain ~/.ssh/id_ed25519
% ssh-add -l   # ホスト側でロード済みか確認
```

> **`ssh-add` はコンテナ内ではなくホストの macOS で実行します。また、再起動のたびに必要です。**
> `--ssh` は「ホストの ssh-agent を転送する」仕組みなので、エージェントに鍵が無ければ何も渡せません。

## 2. ビルドと実行（`--ssh`）

```console
% container system start                       # 初回のみ：サービス起動（INSTALL.md 参照）
% cd git-ssh
% PROJECT=$(basename "$PWD") && container build -t "${PROJECT}-image" .
% container run -it --rm --ssh --name "${PROJECT}-container" "${PROJECT}-image" /bin/bash
```

`--ssh` 一つで、ホストの ssh-agent がコンテナへ転送されます。

## 3. コンテナ内での確認

```console
# env | grep SSH_AUTH_SOCK
SSH_AUTH_SOCK=/var/host-services/ssh-auth.sock   # --ssh が自動設定（値はバージョン依存）
# ssh-add -l                       # 転送された鍵が見えれば成功
# ssh -T git@github.com
Hi <user>! You've successfully authenticated, but GitHub does not provide shell access.
# git clone git@github.com:<owner>/<private-repo>.git
Cloning into '<private-repo>'...
```

秘密鍵はコンテナ内に存在しないことに注目してください（`ls ~/.ssh` は空）。
認証はすべてホストの ssh-agent 経由で行われています。

---

前の教材：**[../hello-world/](../hello-world/)** — はじめてのコンテナ。
