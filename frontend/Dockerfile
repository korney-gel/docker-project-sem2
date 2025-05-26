# ---------- build stage ----------
FROM node:20-alpine AS builder
WORKDIR /app

ENV NODE_OPTIONS=--openssl-legacy-provider


COPY frontend/package*.json ./
RUN npm ci && npm cache clean --force

COPY frontend/ ./
RUN npm run build


# ---------- final stage ----------
FROM nginxinc/nginx-unprivileged:stable-alpine

COPY --from=builder /app/dist /usr/share/nginx/html

COPY frontend/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 8080
USER 101
CMD ["nginx", "-g", "daemon off;"]

# ----- dev stage -----
FROM node:20-alpine AS dev
WORKDIR /app

ENV NODE_OPTIONS=--openssl-legacy-provider

COPY frontend/package*.json ./
RUN npm ci

COPY frontend/ ./
EXPOSE 5173
CMD ["npm", "run", "serve", "--", "--host", "0.0.0.0", "--port", "5173"]