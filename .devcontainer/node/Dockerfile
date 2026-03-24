# Use Go for building GitHub MCP Server
FROM golang:1.24-bullseye AS builder

RUN git clone https://github.com/github/github-mcp-server.git /tmp/github-mcp-server \
    && cd /tmp/github-mcp-server \
    && go build -o github-mcp-server ./cmd/github-mcp-server


FROM ubuntu:24.04

# Node.js バージョン
ARG NODE_VERSION=24
ARG USERNAME=vscode

ENV NODE_VERSION=${NODE_VERSION}
ENV DEBIAN_FRONTEND=noninteractive
ENV LANG=C.UTF-8

# Ubuntu 24.04のデフォルトubuntuユーザーを削除（UID/GID 1000の衝突を回避）
RUN touch /var/mail/ubuntu && chown ubuntu /var/mail/ubuntu && userdel -r ubuntu

# 基本パッケージとNode.jsのインストール
RUN apt-get update \
    && apt-get install -y \
        curl \
        wget \
        ca-certificates \
        gnupg \
        git \
        vim \
        sudo \
        build-essential \
        jq \
        tmux \
        lsb-release \
        locales \
        fonts-noto-cjk \
    && echo "ja_JP.UTF-8 UTF-8" >> /etc/locale.gen \
    && locale-gen \
    && update-locale LANG=ja_JP.UTF-8 \
    && curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

# Copy the built MCP server binary
COPY --from=builder /tmp/github-mcp-server/github-mcp-server /usr/local/bin/

# GitHub CLI のインストール
RUN curl -fsSL https://gist.githubusercontent.com/DIO0550/1830ced050035c34668aa426f7258346/raw/install-gh.sh | bash

# Playwright の依存関係インストール
RUN curl -fsSL https://gist.githubusercontent.com/DIO0550/16c39c3dc4682c7e2fd03c83d0f34b42/raw/install-playwright-deps.sh | bash

# 開発用ユーザーの作成
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME -s /bin/bash \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# npm グローバルディレクトリの設定
RUN mkdir -p /home/${USERNAME}/.npm-global && \
    mkdir -p /home/${USERNAME}/.npm && \
    chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/.npm-global && \
    chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/.npm && \
    npm config set prefix '/home/${USERNAME}/.npm-global'

ENV PATH="/home/${USERNAME}/.npm-global/bin:$PATH"

RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
    && mkdir /commandhistory \
    && touch /commandhistory/.bash_history \
    && chown -R $USERNAME /commandhistory

WORKDIR /workspace

USER $USERNAME

SHELL ["/bin/bash", "-l", "-c"]

# ユーザー環境での npm 設定
RUN npm config set prefix '/home/${USERNAME}/.npm-global' \
    && npm config set ignore-scripts true

RUN echo 'export PATH="/home/'${USERNAME}'/.npm-global/bin:$PATH"' >> /home/${USERNAME}/.bashrc

# npm の最新版にアップデート
RUN npm install -g npm@latest

# pnpm + safe-chain のセットアップ
RUN curl -fsSL https://gist.githubusercontent.com/DIO0550/dcc09dc9d2dcefceb9b0cda183e7ef8e/raw/pnpm-safe-chain-setup.sh | sh

# AI ツールのインストール
RUN curl -fsSL https://gist.githubusercontent.com/DIO0550/148b1392cf435ba3562da0d398bee442/raw/install-ai-tools.sh | bash

# tmux のセットアップ
RUN curl -fsSL https://gist.githubusercontent.com/DIO0550/356434ec2b182f71d99dc0c467555ba5/raw/tmux-setup.sh | bash

# Playwright ブラウザのインストール
RUN curl -fsSL https://gist.githubusercontent.com/DIO0550/6e5249a1b3682e702c875851756f0373/raw/install-playwright.sh | bash

# npm の実行を禁止（pnpm を使用）
RUN cat << 'EOF' > /home/${USERNAME}/.npm-global/bin/npm
#!/bin/sh
echo "----------------------------------------------------------"
echo "npm execution is disabled in this project."
echo "Please use pnpm instead."
echo "----------------------------------------------------------"
exit 1
EOF
RUN chmod +x /home/${USERNAME}/.npm-global/bin/npm
