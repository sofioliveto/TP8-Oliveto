# ====================================
# STAGE 1: Build Stage (only dev deps)
# ====================================
FROM node:20-alpine AS builder

WORKDIR /app

# Copy dependency files first (better cache)
COPY package*.json ./

# Install ALL dependencies (including dev)
RUN npm ci

# Copy source code
COPY . .

# Run tests to ensure code quality
RUN npm test

# ====================================
# STAGE 2: Production Stage (lean)
# ====================================
FROM node:20-alpine AS production

# Security: Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy only package files
COPY package*.json ./

# Install ONLY production dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy application code from builder
COPY --from=builder /app/index.js ./
COPY --from=builder /app/db.js ./

# Create directory for frontend files (will be populated at runtime or by pipeline)
RUN mkdir -p /app/frontend && chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start app
CMD ["node", "index.js"]
