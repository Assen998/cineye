FROM golang:1.24-alpine AS builder

RUN apk add --no-cache git

WORKDIR /build

COPY go.mod go.sum ./
RUN go mod download

COPY . .

ARG VERSION=dev
ARG GIT_COMMIT=unknown
RUN CGO_ENABLED=0 go build \
    -ldflags="-s -w -X main.Version=${VERSION} -X main.BuildTime=$(date -u +%Y-%m-%dT%H:%M:%SZ) -X main.GitCommit=${GIT_COMMIT}" \
    -o cineye ./cmd/server

FROM alpine:3.20

RUN apk add --no-cache \
    ca-certificates \
    ffmpeg \
    tzdata \
    curl \
    && rm -rf /var/cache/apk/*

RUN addgroup -g 1000 -S cineye && \
    adduser -u 1000 -S cineye -G cineye

WORKDIR /app

COPY --from=builder /build/cineye .
COPY configs/config.yaml ./configs/config.yaml

RUN mkdir -p /app/data /app/recordings /app/logs && \
    chown -R cineye:cineye /app

USER cineye

EXPOSE 8080 8081

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD curl -f http://127.0.0.1:8080/health || exit 1

ENTRYPOINT ["./cineye"]
