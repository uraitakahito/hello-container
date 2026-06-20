# コンテナ内から GitHub へ SSH で git clone するための最小イメージ。
#
# git と SSH クライアントだけを入れた Debian。秘密鍵はイメージに含めず、
# 実行時に `container run --ssh` でホストの ssh-agent を転送して使う。
# 再現性のためレジストリを完全修飾し、タグを固定する（hello-world と統一）。
FROM docker.io/library/debian:bookworm-20260610-slim

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        git openssh-client && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
