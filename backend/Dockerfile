# ---- Build stage ----
FROM node:20.5.0-alpine AS build  # Use latest patch to reduce CVEs
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci --only=production && npm cache clean --force

# ---- Production stage ----
FROM node:20.5.0-alpine AS production  # pinned minor version for security

# Security: run as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Security: install minimal runtime dependencies with pinned versions
RUN apk add --no-cache dumb-init=1.2.5-r0  # Pin version to satisfy DL3018

WORKDIR /app

# Copy only production dependencies from build stage
COPY --from=build /app/node_modules ./node_modules
COPY src/ ./src/

# Security: own files by non-root user
RUN chown -R appuser:appgroup /app

USER appuser

EXPOSE 5000

# Security: use dumb-init for proper signal handling (PID 1 problem)
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "src/index.js"]
