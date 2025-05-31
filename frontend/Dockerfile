# Этап 1: Сборка приложения
FROM node:22-alpine AS build

WORKDIR /app

# Копируем package.json и package-lock.json для установки зависимостей
COPY package*.json ./
RUN npm ci

# Копируем исходный код приложения
COPY . .

# Устанавливаем переменные окружения и собираем приложение
ENV NODE_ENV=production
ENV VUE_APP_API_URL=/api
RUN npm run build

# Этап 2: Финальный образ
FROM nginx:1.25-alpine

# Копируем конфигурацию Nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Копируем собранные файлы приложения из предыдущего этапа
COPY --from=build /app/dist /usr/share/nginx/html/momo-store

# Устанавливаем права доступа и создаем необходимые директории
RUN adduser -D -s /sbin/nologin nonroot && \
    mkdir -p /tmp/nginx && \
    sed -i 's|pid .*;|pid /tmp/nginx/nginx.pid;|' /etc/nginx/nginx.conf && \
    chown -R nonroot /var/cache/nginx /var/log/nginx /tmp/nginx
USER nonroot

# Открываем порт, на котором работает приложение
EXPOSE 80

# Проверка работы приложения
HEALTHCHECK --interval=20s --timeout=3s --start-period=5s --retries=3 \
    CMD wget -qO- http://127.0.0.1/ || exit 1