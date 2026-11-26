# syntax=docker/dockerfile:1

# ---------------------------------------------------------------------------
# Dependencies: installed once, from the lockfile, with dev packages excluded.
# ---------------------------------------------------------------------------
FROM node:24-alpine AS deps
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci --omit=dev && npm cache clean --force

# ---------------------------------------------------------------------------
# Runtime: only the production dependencies and the server code.
# ---------------------------------------------------------------------------
FROM node:24-alpine
WORKDIR /app

ENV NODE_ENV=production \
    PORT=8080

COPY --from=deps --chown=node:node /app/node_modules ./node_modules
COPY --chown=node:node . .

# The image ships an unprivileged `node` user (uid 1000); nothing here needs root.
USER node

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
  CMD wget -qO- "http://127.0.0.1:${PORT}/health" >/dev/null || exit 1

CMD ["node", "server.js"]
