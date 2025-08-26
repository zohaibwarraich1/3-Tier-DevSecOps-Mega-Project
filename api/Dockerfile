FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev \
    && npm cache clean --force

COPY . .

FROM gcr.io/distroless/nodejs20
WORKDIR /app
COPY --from=builder /app ./
EXPOSE 5000
CMD [ "app.js" ]
