# ---- STAGE 1: Build stage ----
FROM node:20.18.3-alpine3.21 AS build
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci --legacy-peer-deps && npm cache clean --force
COPY . .
RUN npm run build

# ---- STAGE 2: Production stage ----
# Use a pinned Nginx version to avoid "latest" tag vulnerabilities
FROM nginx:1.27.4-alpine AS production

# Add SHELL to satisfy Hadolint (pipefail)
SHELL ["/bin/ash", "-o", "pipefail", "-c"]

# Apply OS security patches to reach 0 vulnerabilities
RUN apk upgrade --no-cache

# Security: Clean up default files
RUN rm -rf /etc/nginx/conf.d/default.conf /usr/share/nginx/html/*

# Copy custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy build from Stage 1
COPY --from=build /app/dist /usr/share/nginx/html

# Security: Combine CHOWN commands and setup non-root execution
RUN touch /var/run/nginx.pid && \
    chown -R nginx:nginx /usr/share/nginx/html /var/cache/nginx /var/log/nginx /var/run/nginx.pid

USER nginx
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
