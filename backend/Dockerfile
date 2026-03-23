# ==============================================================
# STAGE 1: Build (The "CI" Stage)
# ==============================================================
# Use a specific, stable version of Node.js on Alpine for a small footprint
FROM node:20-alpine AS build

# Set the working directory
WORKDIR /app

# Copy package files first to leverage Docker layer caching
# If package.json hasn't changed, 'npm ci' will be skipped in future builds
COPY package.json package-lock.json* ./

# Use 'npm ci' instead of 'npm install' for reproducible builds
# --only=production ensures we don't bring build tools into the next stage
RUN npm ci --only=production && npm cache clean --force

# Copy the rest of the application source code
COPY . .

# If your app requires a build step (e.g., TypeScript or React), uncomment the line below:
# RUN npm run build

# ==============================================================
# STAGE 2: Production (The "Secure" Stage)
# ==============================================================
FROM node:20-alpine AS production

# Set environment to production
ENV NODE_ENV=production

# 1. Security: Pin version for dumb-init to satisfy Hadolint (DL3018)
# dumb-init acts as PID 1 to handle signals like SIGTERM correctly
RUN apk add --no-cache dumb-init=1.2.5-r3

# 2. Security: Create a dedicated non-root user and group
# Running as 'root' inside a container is a major security risk
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Set working directory
WORKDIR /app

# 3. Security: Copy only the necessary artifacts from the build stage
# This prevents source code and build tools from being in the final image
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/package.json ./package.json

# If you have a 'dist' or 'src' folder, copy it specifically
COPY --from=build /app/src ./src

# 4. Security: Change ownership of the app directory to the non-root user
RUN chown -R appuser:appgroup /app

# 5. Security: Switch to the non-root user
USER appuser

# Expose the application port
EXPOSE 5000

# 6. Security: Use dumb-init as the entrypoint for proper process management
ENTRYPOINT ["dumb-init", "--"]

# Start the application
CMD ["node", "src/index.js"]
