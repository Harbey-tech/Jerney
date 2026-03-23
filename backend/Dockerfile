# ---- STAGE 1: Build Stage ----
# We use a full node image to install dependencies
FROM node:20.18.3-alpine3.21 AS build
WORKDIR /app

# Copy lockfiles first to leverage layer caching
COPY package.json package-lock.json* ./

# Install ALL dependencies (including devDeps if needed for building)
RUN npm ci && npm cache clean --force

# Copy the rest of the source code
COPY . .

# ---- STAGE 2: Production Stage ----
# We use a fresh, slim image for the final container
FROM node:20.18.3-alpine3.21 AS production

# 1. Add SHELL for Hadolint compliance
SHELL ["/bin/ash", "-o", "pipefail", "-c"]

# 2. Get the latest OS security patches and install dumb-init
RUN apk upgrade --no-cache && \
    apk add --no-cache dumb-init=1.2.5-r3

# 3. Create dedicated non-root group and user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# 4. Copy ONLY what is needed from the build stage
# This keeps the image size small and secure
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/src ./src
COPY --from=build /app/package.json ./package.json

# 5. Ensure non-root ownership
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

EXPOSE 5000

# Security: Use dumb-init for proper signal handling as PID 1
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "src/index.js"]
