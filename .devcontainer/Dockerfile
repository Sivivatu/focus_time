# syntax=docker/dockerfile:1

FROM mcr.microsoft.com/devcontainers/go:1-1.22-bookworm

ARG DEBIAN_FRONTEND=noninteractive
ARG NODE_MAJOR=20

USER root

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
      curl ca-certificates gnupg git unzip xz-utils tzdata \
      build-essential pkg-config \
      sqlite3 libsqlite3-dev \
      duckdb; \
    \
    curl -fsSL https://deb.nodesource.com/setup_${NODE_MAJOR}.x | bash -; \
    apt-get install -y --no-install-recommends nodejs; \
    npm i -g npm@latest pnpm yarn; \
    \
    rm -rf /var/lib/apt/lists/*

# Go tooling commonly used in devcontainers
RUN bash -lc ' \
    set -eux; \
    go install golang.org/x/tools/gopls@latest; \
    go install github.com/go-delve/delve/cmd/dlv@latest; \
    go install github.com/golangci/golangci-lint/cmd/golangci-lint@v1.60.3; \
    true'

# Default workspace path used by VS Code Dev Containers
WORKDIR /workspaces/focus_time

# Non-root user is provided by base image (vscode)
USER vscode

ENV PATH=/home/vscode/go/bin:${PATH}

