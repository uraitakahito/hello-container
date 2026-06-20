# hello-container

Apple の [`container`](https://github.com/apple/container) を学ぶための教材シリーズです。
[hello-docker](https://github.com/uraitakahito/hello-docker) の Apple Container 版にあたります。

## 前提

- **macOS 26**（Tahoe）以降
- **Apple Silicon** の Mac
- 一部の機能（コンテナ内仮想化など）は **M3 以降**が必要

> Apple Container は macOS 26 でのみサポートされます。古い macOS では動作しません。

## インストール

Apple Container の導入手順は **[INSTALL.md](./INSTALL.md)** にまとめています。
まずはそちらを実施してください（Homebrew での導入を推奨）。

## トピック

| ディレクトリ | 内容 |
| --- | --- |
| [hello-world/](./hello-world/) | インストール後の最初の一歩。既製イメージでコンテナを起動し、自作イメージをビルドして動かす |
| `git-ssh/` |（予定）コンテナ内から GitHub へ SSH で `git clone` する |

## ライセンス

[LICENSE](./LICENSE) を参照してください。
