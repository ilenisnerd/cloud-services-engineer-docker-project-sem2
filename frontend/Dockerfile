# Этап 1: Сборка приложения
FROM node:18-alpine AS builder

# Рабочая директория
WORKDIR /app

# Копируем package.json и package-lock.json для кэширования зависимостей
COPY package*.json ./
RUN npm ci

# Копируем исходный код
COPY . .

# Устанавливаем переменные окружения
ENV NODE_ENV=production
ENV VUE_APP_API_URL=/api
ENV NODE_OPTIONS=--openssl-legacy-provider

# Сборка приложения
RUN npm run build

# Этап 2: Финальный образ с Nginx
FROM nginx:1.25-alpine

# Копируем собранные статические файлы
COPY --from=builder /app/dist /usr/share/nginx/html/momo-store

# Копируем кастомную конфигурацию Nginx (если есть)
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Указываем пользователя без привилегий
USER nginx

# Открываем порт
EXPOSE 80

# Запускаем Nginx
CMD ["nginx", "-g", "daemon off;"]