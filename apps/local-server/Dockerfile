# Multi-stage build for Christian Heritage Archive Server
# This creates an offline-capable Docker image with pre-extracted database

FROM node:22-alpine as base

# Install build dependencies (py3-setuptools restores distutils for node-gyp)
RUN apk add --no-cache python3 py3-setuptools make g++ sqlite curl

WORKDIR /app

# Copy the entire monorepo to resolve workspace dependencies
COPY . .

# Install pnpm globally
RUN npm install -g pnpm

# Install dependencies at the workspace root
RUN pnpm install --frozen-lockfile

# Build stage - compile TypeScript
FROM base as builder

# Stay in the root workspace directory to resolve cross-dependencies
WORKDIR /app

# Build the server and all of its required monorepo packages
RUN pnpm --filter @surgbc/egw-research-server... build

# Production stage
FROM node:22-alpine as production

# Install runtime dependencies
RUN apk add --no-cache sqlite curl 7zip

WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Switch to the local server directory for production
WORKDIR /app/apps/local-server

# Copy the completely built workspace (bypasses isolated workspace install errors)
COPY --from=builder /app /app

# Copy or create database directory
RUN mkdir -p ./data

# Copy existing database if available from the root context
COPY apps/local-server/dat[a]/ ./data/

# Copy extraction script if ZIP files are provided from the root context
COPY apps/local-server/script[s]/ ./scripts/

# Build-time extraction removed. Database will be extracted at runtime to bypass Docker limits.

# Create directories for PDF generation
RUN mkdir -p /tmp/pdf-generation && \
    chown -R nodejs:nodejs /app /tmp/pdf-generation

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check (Extended start-period to 5 minutes to allow for runtime DB extraction)
HEALTHCHECK --interval=30s --timeout=10s --start-period=300s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Default command - Extract at runtime to bypass Docker limits, then start HTTP server
CMD sh -c "if [ ! -f ./data/egw-writings.db ]; then echo '📦 Stitching and extracting chunks at runtime...' && cat /app/data/egw-writings-part-* > ./data/temp.zip && 7z x -y -o./data/ ./data/temp.zip && rm ./data/temp.zip; fi && node dist/server.js --http --port 3000"

# Labels for metadata
LABEL org.opencontainers.image.title="Christian Heritage Archive Server"
LABEL org.opencontainers.image.description="Offline API server for Christian heritage texts with PDF generation"
LABEL org.opencontainers.image.version="1.0.0"
LABEL org.opencontainers.image.author="surgbc"
LABEL org.opencontainers.image.licenses="MIT"
