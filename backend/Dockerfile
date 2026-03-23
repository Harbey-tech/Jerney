# ---- STAGE 1: Build stage ----
FROM node:20.18.3-alpine3.21 AS build
WORKDIR /app

# Copy lockfiles first for layer caching
COPY package.json package-lock.json* ./

# Using npm ci for a reproducible DevSecOps build
RUN npm ci --legacy-peer-deps && npm cache clean --force

# Copy the source code
COPY . .

# ---- STAGE 2: Production stage ----
FROM node:20.18.3-alpine3.21 AS production

# 1. Add SHELL to satisfy Hadolint (pipefail)
SHELL ["/bin/ash", "-o", "pipefail", "-c"]

# 2. Apply OS security patches AND install signal handler (dumb-init)
# This clears the "1 Vulnerability" Trivy error
RUN apk update && apk upgrade --no-cache && \
    apk add --no-cache dumb-init=1.2.5-r3

# 3. Security: Create dedicated non-root group and user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# 4. Copy ONLY production files from the build stage
# This keeps the image size small and secure
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/src ./src
COPY --from=build /app/package.json ./package.json

# 5. Security: Ensure non-root ownership of the app directory
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

EXPOSE 5000

# 6. Security: Use dumb-init for proper signal handling as PID 1
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "src/index.js"]
