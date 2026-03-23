# ---- STAGE 2: Production Stage ----
FROM node:20.18.3-alpine3.21 AS production

# Add SHELL for Hadolint (Fixes DL4006)
SHELL ["/bin/ash", "-o", "pipefail", "-c"]

# CRITICAL: This line patches the OS vulnerabilities Trivy is finding
RUN apk update && apk upgrade --no-cache && \
    apk add --no-cache dumb-init=1.2.5-r3

# Rest of your setup...
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/src ./src
RUN chown -R appuser:appgroup /app
USER appuser
...
