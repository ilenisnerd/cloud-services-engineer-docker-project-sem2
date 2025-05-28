FROM node:16-alpine AS build

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ENV NODE_ENV=production
ENV VUE_APP_API_URL=/api
RUN npm run build


FROM nginx:1.25-alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html/momo-store

RUN adduser -D -s /sbin/nologin nonroot && \
    mkdir -p /tmp/nginx/client_temp /tmp/nginx/proxy_temp && \
    sed -i 's|pid .*;|pid /tmp/nginx/nginx.pid;|' /etc/nginx/nginx.conf && \
    chown -R nonroot /tmp/nginx


USER nonroot
EXPOSE 80

HEALTHCHECK --interval=20s --timeout=3s --start-period=5s --retries=3 \
    CMD wget -qO- http://127.0.0.1/ || exit 1
