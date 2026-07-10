# ---- Build stage ----
FROM node:26-alpine AS build
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci --only=production && npm cache clean --force

# ---- Production stage ----
FROM node:26-alpine AS production

# Security: non-root user, pinned dumb-init, and remove the bundled npm CLI —
# npm is not needed at runtime and its vendored deps (e.g. undici) show up in
# vulnerability scans
RUN addgroup -S appgroup && adduser -S appuser -G appgroup \
    && apk add --no-cache dumb-init=1.2.5-r4 \
    && rm -rf /usr/local/lib/node_modules/npm /usr/local/bin/npm /usr/local/bin/npx

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
