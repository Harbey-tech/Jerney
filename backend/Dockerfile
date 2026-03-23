# ---- Build stage ----
# ---- Production stage ----
FROM node:20.18.3-alpine3.21 AS production

# Add this to get the latest OS security patches
RUN apk upgrade --no-cache && \
    apk add --no-cache dumb-init=1.2.5-r3
WORKDIR /app

# Copy lockfiles first to leverage layer caching
COPY package.json package-lock.json* ./

# Using npm ci ensures reproducible builds for DevSecOps
RUN npm ci --only=production && npm cache clean --force

# Copy the source code
COPY . .

# ---- Production stage ----
FROM node:20-alpine AS production

# Security: Pin version to satisfy Hadolint DL3018 (Version 1.2.5-r3 for Alpine)
RUN apk add --no-cache dumb-init=1.2.5-r3

# Security: Create dedicated non-root group and user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy only production dependencies and source from build stage
COPY --from=build /app/node_modules ./node_modules
COPY src/ ./src/

# Security: Ensure non-root ownership
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

EXPOSE 5000

# Security: Use dumb-init for proper signal handling as PID 1
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "src/index.js"]
