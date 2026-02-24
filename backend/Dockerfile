FROM node:22-alpine AS builder

RUN apk update && apk upgrade --no-cache

WORKDIR /app

COPY package*.json ./

RUN npm ci --only=production && npm cache clean --force

FROM node:22-alpine AS production

RUN apk update && apk upgrade --no-cache

WORKDIR /app

RUN addgroup -g 1001 -S backendTeam && \
    adduser -S worker -u 1001

COPY --from=builder --chown=worker:backendTeam /app/node_modules ./node_modules
COPY --chown=worker:backendTeam package*.json  ./
COPY --chown=worker:backendTeam server.js      ./
COPY --chown=worker:backendTeam app/           ./app/

USER worker

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:8080', (res) => { process.exit(res.statusCode === 200 ? 0 : 1) })"

CMD ["node", "server.js"]
