ARG BUN_VERSION=1.1.26

ARG YARN_PKG_MANAGER="this.packageManager=\"yarn@1.22.22\""
ARG BUN_PKG_MANAGER="this.packageManager=\"bun@${BUN_VERSION}\""

FROM oven/bun:${BUN_VERSION} AS bun
WORKDIR /app

# ================= TURBO PRUNE ===================

FROM bun AS pruned
ARG YARN_PKG_MANAGER
COPY . .
RUN bunx json -I -f package.json -e ${YARN_PKG_MANAGER}
RUN bunx turbo prune --scope="server" --docker

# =============== INSTALL & BUILD =================

FROM bun AS builder
ARG BUN_PKG_MANAGER
COPY --from=pruned /app/out/full/ .
RUN bunx json -I -f package.json -e ${BUN_PKG_MANAGER}
RUN bun install
RUN bunx clean-modules -y "**/*.ts" "**/@types/**"
RUN bunx turbo build --filter="server"
RUN bun --cwd /app/apps/server build-binary

# ================== RELEASE ======================

FROM gcr.io/distroless/base-debian12 AS release
USER nonroot
COPY --from=builder --chmod=755 /app/apps/server/binary/server /app/server
ENTRYPOINT ["/app/server"]