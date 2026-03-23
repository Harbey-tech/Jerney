# ---- Build stage ----
# Use an explicit tag and avoid trailing comments to prevent parse errors
FROM node:20-alpine AS build
WORKDIR /app

# Copy lockfiles first to leverage layer caching
COPY package.json package-lock.json* ./

# Using npm ci ensures reproducible builds by strictly following the lockfile
RUN npm ci --only=production && npm cache clean --force

# ---- Production stage ----
FROM node:20-alpine AS production

# Security: Pin versions to satisfy Hadolint DL3018 and ensure immutability
# Note: 1.2.5-r3 is a common stable version for Alpine 3.19/3.20
RUN apk add --no-cache dumb-init=1.2.5-r3

# Security: Create a dedicated non-root group and user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy only production dependencies from build stage to reduce attack surface
COPY --from=build /app/node_modules ./node_modules
COPY src/ ./src/

# Security: Ensure the non-root user owns the application directory
RUN chown -R appuser:appgroup /app

# Switch to non-root user before exposing ports
USER appuser

EXPOSE 5000

# Security: Use dumb-init to handle signals (like SIGTERM) properly as PID 1
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "src/index.js"]
