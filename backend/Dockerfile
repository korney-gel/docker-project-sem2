# Build stage
FROM golang:1.21-alpine AS builder

RUN apk add --no-cache git

WORKDIR /app

COPY backend/go.mod backend/go.sum ./
RUN go mod download

COPY backend/ ./
WORKDIR /app/cmd/api
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o /app/app

# Final stage
FROM alpine:latest

RUN apk add --no-cache ca-certificates curl && \
    rm -rf /var/cache/apk/*

COPY --from=builder /app/app /app

EXPOSE 8081

USER 1001

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl http://localhost:8081/ || exit 1

ENTRYPOINT ["/app"]