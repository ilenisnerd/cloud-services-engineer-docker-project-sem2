# Этап 1: Сборка приложения
FROM golang:1.21-alpine AS builder

WORKDIR /app

# Копируем go.mod и go.sum для кэширования зависимостей
COPY go.mod go.sum ./
RUN go mod download

# Копируем исходный код
COPY . .

# Сборка бинарного файла с отключением CGO для статической сборки
RUN CGO_ENABLED=0 GOOS=linux go build -trimpath -ldflags="-s -w" -o backend ./cmd/api

# Этап 2: Финальный образ
FROM alpine:3.21

WORKDIR /app

# Копируем скомпилированный бинарник из предыдущего этапа
COPY --from=builder /app/backend .

RUN adduser -D -s /sbin/nologin nonroot && \
    chown nonroot:nonroot /app/backend && chmod +x /app/backend

# Указываем пользователя без привилегий для повышения безопасности
USER nonroot

# Открываем порт, на котором работает приложение
EXPOSE 8081

# Команда для запуска приложения
CMD ["./backend"]
