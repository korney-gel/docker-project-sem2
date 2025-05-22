# Этап сборки
FROM golang:1.21-alpine AS builder

# Обеспечение безопасности
RUN apk add --no-cache git

WORKDIR /app

COPY backend/go.mod backend/go.sum ./
RUN go mod download

COPY backend/ ./

# Сборка без тестов, оптимизированная
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o app

# Этап запуска — минимальный образ
FROM scratch

# Копируем бинарник
COPY --from=builder /app/app /app

# Указываем пользователя (не root)
USER 1001

ENTRYPOINT ["/app"]