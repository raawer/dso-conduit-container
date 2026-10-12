FROM node:20-alpine AS builder
ENV HUSKY=0
WORKDIR /app
COPY ./package.json ./package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
LABEL org.opencontainers.image.source=https://github.com/raawer/dso-conduit-container
COPY --from=builder /app/dist/angular-conduit /usr/share/nginx/html
ENV BACKEND_HOST=backend \
    BACKEND_PORT=8000 \
    NGINX_ENVSUBST_FILTER="^BACKEND_"
COPY nginx.conf.template /etc/nginx/templates/default.conf.template
EXPOSE 80
