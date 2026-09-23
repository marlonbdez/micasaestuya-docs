---
title: Environment Variables
---
# Variables de entorno

Catálogo — no incluye valores reales/secretos, solo qué existe y dónde vive.

## web

| Variable | Uso | Dónde se define |
|----------|-----|-------------------|
| NUXT_PUBLIC_API_BASE | URL base de la api que consume el frontend | .github/workflows/ci.yml, Netlify |
| NUXT_API_SECRET | Secreto de servidor (runtimeConfig privado) | Netlify |
| NUXT_SENTRY_AUTH_TOKEN | Token para que el build suba source maps a Sentry | Netlify |
| NUXT_PUBLIC_SENTRY_DSN | DSN del proyecto en Sentry | Netlify |
| NUXT_PUBLIC_SENTRY_ORG_SLUG | Org de Sentry | Netlify |
| NUXT_PUBLIC_SENTRY_PROJECT_SLUG | Proyecto de Sentry | Netlify |
| NUXT_PUBLIC_SENTRY_ENVIRONMENT | Entorno reportado a Sentry (production/development) | Netlify |

`NUXT_PUBLIC_API_SECONDARY`, que aparecía aquí como "candidata a eliminar", ya se eliminó del código y de la CI — no existe más.

## api

| Variable | Uso | Dónde se define |
|----------|-----|-------------------|
| PORT | Puerto del servidor Express | Render, .env local |
| NODE_ENV | development / test / production | Render, .env local |
| MONGODB_URI | Conexión a Mongo (prod/dev según entorno) | Render, .env local |
| MONGODB_TEST_URI | Conexión a Mongo usada solo por npm test | CI, .env local |
| REDIS_URI | Conexión a Redis (prod/dev según entorno) | Render, .env local |
| REDIS_TEST_URI | Conexión a Redis usada solo por npm test | CI, .env local |
| SECRET | Firma de JWT | Render, .env local |

## infra (docker-compose local)

| Variable | Uso |
|----------|-----|
| MONGO_DB_USERNAME / MONGO_DB_PASSWORD / MONGO_DB_NAME | Credenciales del Mongo local |
| MONGO_DB_URI / MONGO_DB_TEST_URI | Se inyectan al servicio express |
| SECRET | JWT para desarrollo local |


Ver también: [Entornos y ramas](Environments-and-Branches.md)
