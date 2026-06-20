# Apple Container のインストール

教材シリーズ共通の前提となる、Apple [`container`](https://github.com/apple/container) の導入手順です。

## 要件

- **macOS 26**（Tahoe）以降 — Apple Container は macOS 26 でのみサポートされます
- **Apple Silicon** の Mac
- 一部の機能（コンテナ内仮想化など）は **M3 以降**が必要

確認:

```console
% sw_vers | grep ProductVersion   # 26.x であること
% uname -m                        # arm64 であること
```

## インストール（Homebrew・推奨）

本シリーズでは [Homebrew](https://brew.sh) での導入を推奨します。更新・削除を `brew` で一元管理できて手軽です。

```console
% brew install container
% container --version
```

> `container` は homebrew-core の formula です。`/opt/homebrew` 配下にインストールされます。

## サービスの起動

Apple Container は、**コンテナごとに専用の軽量 Linux VM** を起動します。
まずはバックグラウンドのサービスを起動します。**初回は Linux カーネルの導入を尋ねられます**（`y` で進めます）。

```console
% container system start
Verifying apiserver is running...
Installing base container filesystem...
No default kernel configured.
Install the recommended default kernel from [https://github.com/kata-containers/.../kata-static-...-arm64.tar.xz]? [Y/n]: y
Installing kernel...
```

動作確認 — まだコンテナが無いので、空の一覧が返れば成功です。

```console
% container ls -a
ID  IMAGE  OS  ARCH  STATE  IP
```

## 更新・アンインストール

```console
% brew upgrade container      # 更新
% container system stop       # サービス停止
% brew uninstall container    # アンインストール
```

## 代替手段：公式の署名済み `.pkg`

Apple は署名済みの `.pkg` インストーラも配布しています（公式 README が案内する正式な方法。`/usr/local` 配下に入ります）。

- [Releases](https://github.com/apple/container/releases) から `.pkg` をダウンロードしてダブルクリック
- 更新: `/usr/local/bin/update-container.sh`（`-v <version>` で版指定）
- 削除: `/usr/local/bin/uninstall-container.sh -k`（`-d` でユーザーデータも削除）

> **注意:** Homebrew 版（`/opt/homebrew`）と `.pkg` 版（`/usr/local`）を**両方同時にインストールしない**でください。`container` バイナリやサービスが二重になり、どちらが動くか分からなくなります。本シリーズでは Homebrew に統一します。

---

導入できたら、最初の一歩 **[hello-world/](./hello-world/)** へ進んでください。
