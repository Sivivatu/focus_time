# syntax=docker/dockerfile:1

ARG GO_VERSION=1.25.2
FROM mcr.microsoft.com/devcontainers/base:bookworm

ARG DEBIAN_FRONTEND=noninteractive
ARG NODE_MAJOR=20
ARG GO_VERSION

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

# Install Go ${GO_VERSION}
RUN set -eux; \
    arch="$(uname -m)"; case "$arch" in x86_64) goarch=amd64 ;; aarch64) goarch=arm64 ;; *) echo "unsupported arch: $arch" && exit 1 ;; esac; \
    curl -fsSL "https://go.dev/dl/go${GO_VERSION}.linux-${goarch}.tar.gz" -o /tmp/go.tgz; \
    rm -rf /usr/local/go; \
    tar -C /usr/local -xzf /tmp/go.tgz; \
    rm /tmp/go.tgz

ENV GOROOT=/usr/local/go
ENV GOPATH=/home/vscode/go
ENV PATH=${GOROOT}/bin:${GOPATH}/bin:${PATH}

# Go tooling commonly used in devcontainers
RUN bash -lc ' \
    set -eux; \
    go version; \
    go install golang.org/x/tools/gopls@latest; \
    go install github.com/go-delve/delve/cmd/dlv@latest; \
    go install github.com/golangci/golangci-lint/cmd/golangci-lint@v1.60.3; \
    true'

# Default workspace path used by VS Code Dev Containers
WORKDIR /workspaces/focus_time

# Non-root user is provided by base image (vscode)
USER vscode

