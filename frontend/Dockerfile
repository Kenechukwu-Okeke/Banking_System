# syntax=docker/dockerfile:1

# ---------------------------------------------------------------------------
# Build: React compiles to static files. The API URL is baked in at build time,
# so it is passed as a build argument rather than an environment variable.
# ---------------------------------------------------------------------------
FROM node:24-alpine AS build
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .

ARG REACT_APP_API_URL=""
ENV REACT_APP_API_URL=${REACT_APP_API_URL}

RUN npm run build

# ---------------------------------------------------------------------------
# Runtime: nginx without root, serving the static bundle and proxying /api.
# ---------------------------------------------------------------------------
FROM nginxinc/nginx-unprivileged:1.27-alpine

COPY nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget -qO- http://127.0.0.1:8080/ >/dev/null || exit 1
