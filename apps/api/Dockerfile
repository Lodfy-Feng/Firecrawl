# ---------- Node Base ---------- #
FROM node:22-slim AS base

# ✅ 添加 HTTP/HTTPS 代理（使用 V2RayA 的 HTTP 代理端口）
ENV http_proxy=http://127.0.0.1:20171
ENV https_proxy=http://127.0.0.1:20171

ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"

RUN corepack enable
COPY . /app
WORKDIR /app

# ---------- Production Dependencies ---------- #
FROM base AS prod-deps
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --prod --frozen-lockfile

# ---------- Build Frontend ---------- #
FROM base AS build
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install && pnpm run build

# ---------- Go Parser ---------- #
FROM golang:1.24 AS go-base

# ✅ 添加代理
ENV http_proxy=http://127.0.0.1:20171
ENV https_proxy=http://127.0.0.1:20171

COPY sharedLibs/go-html-to-md /app/sharedLibs/go-html-to-md

RUN cd /app/sharedLibs/go-html-to-md && \
    go mod download && \
    go build -o html-to-markdown.so -buildmode=c-shared html-to-markdown.go && \
    chmod +x html-to-markdown.so

# ---------- Rust Parser ---------- #
FROM rust:1-slim AS rust-base

# ✅ 添加代理
ENV http_proxy=http://127.0.0.1:20171
ENV https_proxy=http://127.0.0.1:20171

COPY sharedLibs/html-transformer /app/sharedLibs/html-transformer
COPY sharedLibs/pdf-parser /app/sharedLibs/pdf-parser
COPY sharedLibs/crawler /app/sharedLibs/crawler

RUN cd /app/sharedLibs/html-transformer && \
    cargo build --release --locked && \
    chmod +x target/release/libhtml_transformer.so

RUN cd /app/sharedLibs/pdf-parser && \
    cargo build --release --locked && \
    chmod +x target/release/libpdf_parser.so

RUN cd /app/sharedLibs/crawler && \
    cargo build --release --locked && \
    chmod +x target/release/libcrawler.so

# ---------- Final Image ---------- #
FROM base

# ✅ 继续添加代理
ENV http_proxy=http://127.0.0.1:20171
ENV https_proxy=http://127.0.0.1:20171

COPY --from=build /app/dist /app/dist
COPY --from=prod-deps /app/node_modules /app/node_modules
COPY --from=go-base /app/sharedLibs/go-html-to-md/html-to-markdown.so /app/sharedLibs/go-html-to-md/html-to-markdown.so
COPY --from=rust-base /app/sharedLibs/html-transformer/target/release/libhtml_transformer.so /app/sharedLibs/html-transformer/target/release/libhtml_transformer.so
COPY --from=rust-base /app/sharedLibs/pdf-parser/target/release/libpdf_parser.so /app/sharedLibs/pdf-parser/target/release/libpdf_parser.so
COPY --from=rust-base /app/sharedLibs/crawler/target/release/libcrawler.so /app/sharedLibs/crawler/target/release/libcrawler.so

RUN apt-get update && apt-get install -y git procps && rm -rf /var/lib/apt/lists/*

EXPOSE 8080

RUN sed -i 's/\r$//' /app/docker-entrypoint.sh

ENTRYPOINT ["/app/docker-entrypoint.sh"]
