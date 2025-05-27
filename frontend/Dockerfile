########################################
# 1. build – собираем Vue-приложение   #
########################################
FROM node:20-alpine AS build
WORKDIR /app

# Webpack 4 + Node 20 → нужен legacy-OpenSSL
ENV NODE_OPTIONS=--openssl-legacy-provider

COPY package*.json ./
RUN npm ci && npm cache clean --force

COPY . .
RUN npm run build


########################################
# 2. dev – live-reload для профиля dev #
########################################
FROM node:20-alpine AS dev
WORKDIR /app
ENV NODE_OPTIONS=--openssl-legacy-provider

COPY package*.json ./
RUN npm ci
COPY . .

EXPOSE 5173
CMD ["npm","run","serve","--","--host","0.0.0.0","--port","5173"]


########################################
# 3. runtime – продовый Nginx-слой     #
########################################
FROM nginxinc/nginx-unprivileged:stable-alpine AS runtime

# собранная статика
COPY --from=build /app/dist /usr/share/nginx/html

# кастом-конфиг (лежит рядом в ./frontend/nginx.conf)
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 8080
USER 101
CMD ["nginx","-g","daemon off;"]