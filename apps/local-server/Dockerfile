FROM node:22-alpine as base
RUN apk add --no-cache python3 make g++ sqlite curl
WORKDIR /app
COPY package*.json ./
COPY tsconfig.json ./
RUN npm install -g pnpm
RUN pnpm install

FROM base as builder
WORKDIR /app
COPY src/ ./src/
RUN pnpm build

FROM node:22-alpine as production
RUN apk add --no-cache sqlite curl
WORKDIR /app
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
COPY package*.json ./
RUN npm install -g pnpm && pnpm install --prod
COPY --from=builder /app/dist/ ./dist/

RUN mkdir -p ./data ./scripts
RUN touch ./data/egw-writings.db

RUN mkdir -p /tmp/pdf-generation && chown -R nodejs:nodejs /app /tmp/pdf-generation
USER nodejs
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 CMD curl -f http://localhost:3000/health || exit 1
CMD ["node", "dist/server.js", "--http", "--port", "3000"]
