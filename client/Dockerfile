FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev \
    && npm cache clean --force
COPY . .
RUN npm run build

FROM nginx:alpine 
COPY --from=builder /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
RUN mkdir -p /var/log/nginx \
    && chown -R nginx:nginx /var/log/nginx
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
