# ---------- build stage ----------
FROM node:20-alpine AS builder
WORKDIR /app

ENV NODE_OPTIONS=--openssl-legacy-provider


COPY package*.json ./
RUN npm ci && npm cache clean --force

COPY . .
RUN npm run build


# ---------- final stage ----------
FROM nginxinc/nginx-unprivileged:stable-alpine

COPY --from=builder /app/dist /usr/share/nginx/html

COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 8080
USER 101
CMD ["nginx", "-g", "daemon off;"]

# ----- dev stage -----
FROM node:20-alpine AS dev
WORKDIR /app

ENV NODE_OPTIONS=--openssl-legacy-provider

COPY package*.json ./
RUN npm ci

COPY . .
EXPOSE 5173
CMD ["npm", "run", "serve", "--", "--host", "0.0.0.0", "--port", "5173"]