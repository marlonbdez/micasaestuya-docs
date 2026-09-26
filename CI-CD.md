---
title: CI-CD
---
# CI/CD

Corre en GitHub Actions. Los tres repos siguen el mismo patrón: `.github/workflows/ci.yml` + `dependabot.yml`; web y api añaden además `codeql.yml`.

## web (`.github/workflows/ci.yml`)

Jobs, todos en `ubuntu-latest`:

| Job | Cuándo corre | Qué hace |
|-----|---------------|----------|
| lint | todo menos push a main | npm run lint (Node 20) |
| test-unit | todo menos push a main | Vitest + cobertura, sube el reporte como artifact |
| build | siempre | `npm run generate` (build estático), sube `.output/public` como artifact |
| e2e | todo menos push a main | Cypress en 2 shards, sobre el build estático servido con `serve` |
| accessibility | todo menos push a main | pa11y-ci sobre el build estático (`continue-on-error`, no bloquea) |
| publish-docker | solo push a main | build + push a `ghcr.io/marlonbdez/micasaestuya-web` |

Nota: lint/test/build corren con Node 20; la imagen Docker (`Dockerfile.prod`) usa Node 22-alpine. Es una diferencia real, no un error de esta página — ver [Decisiones (ADRs)](ADRs.md) ADR 004.

## api (`.github/workflows/ci.yml`)

| Job | Cuándo corre | Qué hace |
|-----|---------------|----------|
| lint | todo menos push a main | npm run lint (Node 22) |
| test | todo menos push a main | npm test, contra contenedores efímeros `mongo:6` / `redis:7-alpine` |
| publish-docker | solo push a main | build + push a `ghcr.io/marlonbdez/micasaestuya-api`, sin consumidor (Render construye desde GitHub) |

## infra (`.github/workflows/ci.yml`)

| Job | Qué hace |
|-----|----------|
| validate | valida `docker-compose.yml`, `.env.example` y los JSON de `seed/` |
| docker-build | `docker compose build` (continue-on-error) |

## CodeQL (web y api)

`codeql.yml` corre análisis de seguridad estático (JS/TS) en cada push/PR a main y en cron semanal.

## Dependabot (los tres repos)

`dependabot.yml` abre PRs semanales para dependencias npm, imágenes Docker y GitHub Actions.

## Nota sobre `develop`

Los tres workflows disparan también en push/PR a `develop`, pero esa rama no existe todavía en ninguno de los tres repos — hoy solo hay `main`. Ver [Entornos y ramas](Environments-and-Branches.md).

Ver también: [Infraestructura y despliegue](Infrastructure-and-Deployment.md)

## Por qué no hay Code Climate: la complejidad la mide ESLint

Cuando la CI vivía en GitLab, el informe de Code Quality lo generaba Code
Climate. GitLab lo deprecó en la 17.3 (y lo elimina en la 19.0), así que en vez
de afinar una herramienta con fecha de caducidad se pasó la métrica de
complejidad al propio ESLint, con la regla `complexity` (`['error', 10]` en el
`.eslintrc.cjs` de web). Antes había dos herramientas opinando con criterios
distintos sobre el mismo código; ahora hay una sola.

Con la migración a GitHub Actions desapareció el informe en sí: el job `lint`
solo ejecuta `npm run lint`, y un exceso de complejidad hace fallar el job en
vez de aparecer en un informe aparte. El formato `--format gitlab` ya no se usa.
