# Этап сборки
FROM golang:1.21-alpine AS builder

RUN apk add --no-cache git

WORKDIR /app

COPY backend/go.mod backend/go.sum ./
RUN go mod download

COPY backend/ ./

WORKDIR /app/cmd/api
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o /app/app

# Финальный образ
FROM scratch

COPY --from=builder /app/app /app

USER 1001

ENTRYPOINT ["/app"]