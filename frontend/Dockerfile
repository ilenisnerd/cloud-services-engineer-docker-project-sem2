FROM node:16-alpine AS build

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ENV NODE_ENV=production
ENV VUE_APP_API_URL=/api
RUN npm run build

FROM nginx:1.25-alpine

# Копируем конфигурации
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html/momo-store

# Полностью настраиваем пути для работы без root
RUN mkdir -p /tmp/nginx && \
    mkdir -p /tmp/nginx/client_temp && \
    mkdir -p /tmp/nginx/proxy_temp && \
    mkdir -p /tmp/nginx/fastcgi_temp && \
    mkdir -p /tmp/nginx/uwsgi_temp && \
    mkdir -p /tmp/nginx/scgi_temp && \
    chown -R nginx:nginx /tmp/nginx && \
    chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /var/run && \
    chown -R nginx:nginx /usr/share/nginx/html && \
    chown -R nginx:nginx /var/log/nginx

# Модифицируем конфигурацию nginx для использования временных директорий в /tmp
RUN sed -i \
    -e 's|/var/run/nginx.pid|/tmp/nginx/nginx.pid|g' \
    -e 's|/var/cache/nginx/|/tmp/nginx/|g' \
    /etc/nginx/nginx.conf

# Отключаем entrypoint скрипты, которые требуют root
RUN rm -rf /docker-entrypoint.d/*

USER nginx
EXPOSE 80

HEALTHCHECK --interval=20s --timeout=3s --start-period=5s --retries=3 \
    CMD wget -qO- http://127.0.0.1/ || exit 1