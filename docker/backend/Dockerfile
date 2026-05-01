FROM node:20-slim AS base
WORKDIR /app
COPY package*.json ./
COPY domain/package*.json ./domain/
COPY application/package*.json ./application/
COPY infrastructure/backend/express/package*.json ./infrastructure/backend/express/
# ---- Dev ----
FROM base AS dev
RUN npm ci
CMD ["npx", "tsx", "watch", "infrastructure/backend/express/server.ts"]
# ---- Builder ----
FROM base AS builder
RUN npm ci
COPY . .
RUN npm run build:express
RUN find /app/infrastructure/backend/express/dist -name "*.js" -exec sed -i 's|require("@application/|require("/app/application/dist/|g' {} \;
RUN find /app/infrastructure/backend/express/dist -name "*.js" -exec sed -i 's|require("@domain/|require("/app/domain/dist/|g' {} \;
# ---- Prod ----
FROM node:20-slim AS prod
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
COPY infrastructure/backend/express/package*.json ./infrastructure/backend/express/
RUN npm ci --omit=dev --workspace=@myges/express
COPY --from=builder /app/infrastructure/backend/express/dist ./infrastructure/backend/express/dist
COPY --from=builder /app/domain/dist ./domain/dist
COPY --from=builder /app/application/dist ./application/dist
CMD ["node", "infrastructure/backend/express/dist/server.js"]
