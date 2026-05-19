# Stage 1: Install dependencies
FROM node:22-alpine AS deps

RUN corepack enable && corepack prepare pnpm@10.15.0 --activate

WORKDIR /app

COPY pnpm-lock.yaml package.json ./
COPY ../../pnpm-workspace.yaml ./

RUN pnpm install --frozen-lockfile

# Stage 2: Build the application
FROM deps AS builder

WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

RUN pnpm build

# Stage 3: Run the application
FROM node:22-alpine AS runner

WORKDIR /app

ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next ./dist/.next
COPY --from=builder /app/package.json ./

USER nextjs

EXPOSE 3000

ENV PORT=3000

CMD ["node", "dist/server.js"]