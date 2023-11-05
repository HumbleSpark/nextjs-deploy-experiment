FROM node:20-alpine AS base

# libc6-compat reason
# https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine 


#####
#
# BUILDER
#
######

FROM base AS builder
RUN apk add --no-cache libc6-compat
RUN apk update
WORKDIR /app

RUN yarn global add turbo
COPY . .
RUN turbo prune nextjs-deploy-experiment --docker

#####
#
# INSTALLER
#
######

FROM base AS installer
RUN apk add --no-cache libc6-compat
RUN apk update
WORKDIR /app

COPY .gitignore .gitignore
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/yarn.lock ./yarn.lock

RUN yarn

COPY --from=builder /app/out/full/ .
COPY turbo.json turbo.json

# ARG TURBO_TEAM
# ENV TURBO_TEAM=$TURBO_TEAM
# ARG TURBO_TOKEN
# ENV TURBO_TOKEN=$TURBO_TOKEN

RUN yarn turbo run build --filter=nextjs-deploy-experiment...

#####
#
# RUNNER
#
######

FROM base AS runner
WORKDIR /app

ARG PORT=3000
ENV PORT=$PORT

# Don't run production as root
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs


COPY --from=installer /app/apps/nextjs-deploy-experiment/next.config.js .
COPY --from=installer /app/apps/nextjs-deploy-experiment/package.json .

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=installer --chown=nextjs:nodejs /app/apps/nextjs-deploy-experiment/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/nextjs-deploy-experiment/.next/static ./apps/nextjs-deploy-experiment/.next/static
# COPY --from=installer --chown=nextjs:nodejs /app/apps/nextjs-deploy-experiment/public ./apps/nextjs-deploy-experiment/public

CMD node apps/nextjs-deploy-experiment/server.js
