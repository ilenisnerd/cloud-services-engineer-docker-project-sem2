FROM node:18-alpine AS builder

RUN apk update && \
    apk upgrade && \
    apk add --no-cache python3 make g++
ENV DOCKER_CONTENT_TRUST=1
WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

ENV NODE_ENV=production
ENV VUE_APP_API_URL=/api
ENV NODE_OPTIONS=--openssl-legacy-provider

RUN npm run build

# Минимальный образ
FROM nginx:1.25-alpine

ENV DOCKER_CONTENT_TRUST=1

RUN apk update && \
    apk upgrade && \
    apk add --no-cache curl

# Создание непривилегированной группы и пользователя
# RUN addgroup -g 1001 -S appgroup && \
#    adduser -u 1001 -S appuser -G appgroup

# Копирование nginx
COPY --chown=appuser:appgroup nginx.conf /etc/nginx/conf.d/default.conf

# Копирование скомпилированного приложения (build)
COPY --from=builder --chown=appuser:appgroup /app/dist /usr/share/nginx/html
# Установка прав
RUN apk update && \
    apk add --no-cache curl &&\
    addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G root &&\
    chown -R appuser:root /var/cache/nginx && \
    chown -R appuser:root /var/log/nginx && \
    chown -R appuser:root /etc/nginx/conf.d && \
    touch /var/run/nginx.pid && \
    chown -R appuser:root /var/run/nginx.pid &&\
    chown -R appuser:root /etc/nginx &&\
    chown -R appuser:root /tmp


USER appuser

EXPOSE 80

# healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

CMD ["nginx", "-g", "daemon off;"]