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
RUN cd domain && npx tsc -p tsconfig.json
RUN cd application && npx tsc -p tsconfig.json
RUN cd infrastructure/backend/express && npx tsc -p tsconfig.json && npx tsc-alias -p tsconfig.json
# ---- Prod ----
FROM node:20-slim AS prod
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
COPY infrastructure/backend/express/package*.json ./infrastructure/backend/express/
RUN npm ci --omit=dev --workspace=@myges/express
COPY --from=builder /app/infrastructure/backend/express/dist ./infrastructure/backend/express/dist
CMD ["node", "infrastructure/backend/express/dist/server.js"]
