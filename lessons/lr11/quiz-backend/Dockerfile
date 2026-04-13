FROM docker.io/oven/node:1-alpine AS builder //берем официальный образ 
WORKDIR /app //создаем рабочую папку
// тут созаем контейнер 
COPY package.json package-lock.json ./ 
RUN npm install --frozen-lockfile

COPY . .
RUN npx prisma generate
//мультисборка для оптимизации так сказала документация 
FROM docker.io/oven/node:1-alpine AS runner 
WORKDIR /app
//устанавливаем библию для работы 
RUN apk add --no-cache curl
//создаем группы пользователей и пользвателя с правами чтения 
RUN addgroup --system --gid 1001 appgroup \
 && adduser  --system --uid 1001 --ingroup appgroup appuser
//даем рпава на файлы и папки 
COPY --from=builder --chown=appuser:appgroup /app/node_modules  ./node_modules
COPY --from=builder --chown=appuser:appgroup /app/src           ./src
COPY --from=builder --chown=appuser:appgroup /app/prisma        ./prisma
COPY --from=builder --chown=appuser:appgroup /app/prisma.config.ts ./
COPY --from=builder --chown=appuser:appgroup /app/tsconfig.json ./
COPY --from=builder --chown=appuser:appgroup /app/package.json  ./
//создаем папку и выдаем права 
RUN mkdir -p /app/data && chown appuser:appgroup /app/data

USER appuser

ENV NODE_ENV=production
EXPOSE 3000 //порт 
//запускаем эндпоинт 
HEALTHCHECK --interval=10s --timeout=5s --start-period=20s --retries=5 \
  CMD curl -sf http://localhost:3000/health | grep -q ok || exit 1