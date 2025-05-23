# Этап сборки
FROM node:20-alpine AS builder

WORKDIR /app

ENV NODE_OPTIONS=--openssl-legacy-provider

# Установка зависимостей
COPY frontend/package*.json ./
RUN npm ci

COPY frontend/ ./
RUN npm run build

# Этап запуска на Nginx
FROM nginx:alpine

# Удаляем дефолтную конфигурацию
RUN rm -rf /usr/share/nginx/html/*

# Копируем собранный фронт
COPY --from=builder /app/dist /usr/share/nginx/html

# Безопасность: только нужные файлы, пользователь по умолчанию nginx
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]