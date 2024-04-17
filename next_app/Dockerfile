# ベースイメージをNode.js v21のものに変更
FROM node:21-bullseye-slim AS base

FROM base AS builder

WORKDIR /workspace/next_app

# apt-getを使用して依存パッケージを更新
RUN apt-get update && apt-get upgrade -y
# package.jsonとpackage-lock.jsonをコピー
COPY next_app/package.json next_app/package-lock.json* ./

# ソースコードとその他の必要ファイルをコピー
COPY next_app/src ./src
COPY next_app/public ./public
COPY next_app/tsconfig.json next_app/*.js ./

# npmを使用して依存関係をインストール
RUN npm ci

# Next.jsのテレメトリを無効にする
ARG NEXT_TELEMETRY_DISABLED=1
ENV NEXT_TELEMETRY_DISABLED=$NEXT_TELEMETRY_DISABLED

# npmを使用してビルド
RUN npm run build

FROM base AS runner

WORKDIR /workspace/next_app

USER node

# ビルドされたファイルをコピー
COPY --from=builder /workspace/next_app/public ./public

# Next.jsアプリケーションのスタンドアロンバージョンと静的ファイルをコピー
COPY --from=builder --chown=node:node /workspace/next_app/.next/standalone ./
COPY --from=builder --chown=node:node /workspace/next_app/.next/static ./.next/static

# 環境変数を設定
ENV NEXT_TELEMETRY_DISABLED=$NEXT_TELEMETRY_DISABLED

# サーバーを起動
CMD ["node", "server.js"]
