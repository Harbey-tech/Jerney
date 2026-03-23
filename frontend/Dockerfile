# ---- STAGE 1: Build stage ----
# Use a specific, stable version for reproducibility
FROM node:20.18.3-alpine3.21 AS build
WORKDIR /app

# Copy lockfiles first to leverage layer caching
COPY package.json package-lock.json* ./

# Using npm ci ensures reproducible builds
RUN npm ci --legacy-peer-deps && npm cache clean --force

# Copy the source code and build the app
COPY . .
RUN npm run build

# ---- STAGE 2: Production stage ----
# Use a pinned Nginx version for stability
FROM nginx:1.27.4-alpine AS production

# 1. Add SHELL to satisfy Hadolint DL4006 (pipefail)
SHELL ["/bin/ash", "-o", "pipefail", "-c"]

# 2. Update OS packages to fix "1 Vulnerability" found in scans
RUN apk upgrade --no-cache

# 3. Security: Clean up default files
RUN rm -rf /etc/nginx/conf.d/default.conf /usr/share/nginx/html/*

# 4. Copy custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# 5. Copy ONLY the production build from Stage 1
COPY --from=build /app/dist /usr/share/nginx/html

# 6. Security: Combine CHOWN commands to reduce layers (Hadolint DL3059)
# Also ensures Nginx can run as non-root
RUN touch /var/run/nginx.pid && \
    chown -R nginx:nginx /usr/share/nginx/html /var/cache/nginx /var/log/nginx /var/run/nginx.pid

# Switch to non-root user
USER nginx

EXPOSE 80

# Use the default Nginx entrypoint
CMD ["nginx", "-g", "daemon off;"]
