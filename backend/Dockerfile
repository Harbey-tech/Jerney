# ---- Build stage ----
FROM node:20-alpine AS build
WORKDIR /app

# Copy lockfiles first to leverage layer caching
COPY package.json package-lock.json* ./

# Note: Using npm ci is correct for DevSecOps (reproducible builds)
RUN npm ci --only=production && npm cache clean --force

# ---- Production stage ----
FROM node:20-alpine AS production

# Security: install minimal runtime dependencies with version pinning
# satisfy Hadolint DL3018
RUN apk add --no-cache dumb-init=1.2.5-r3

# Security: run as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy only production dependencies from build stage
COPY --from=build /app/node_modules ./node_modules
COPY src/ ./src/

# Security: own files by non-root user
RUN chown -R appuser:appgroup /app

# Switch to non-root user before exposing ports/CMD
USER appuser

EXPOSE 5000

# Security: use dumb-init for proper signal handling (PID 1)
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "src/index.js"]
