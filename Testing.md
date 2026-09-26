---
title: Testing
---
# Testing

## web

- Unit: Vitest (test:unit, test:unit:headless, test:unit:coverage).
- E2E: Cypress 13.4.0 (cypress/e2e/auth.cy.ts, cypress/e2e/i18n.cy.ts).

### Dónde corren los e2e

| Dónde | Contra qué | Límite de peticiones |
|-------|-----------|----------------------|
| CI | API efímera con mongo y redis vacíos (`NODE_ENV=test`); ver [CI-CD](CI-CD.md) | Saltado |
| Local | Tu stack de Docker: web en `:3000`, api en `:3001` (`npx cypress run --spec cypress/e2e/auth.cy.ts --browser chrome`) | **Activo** salvo que pongas `RATE_LIMIT=off` |

**Los e2e nunca deben apuntar a producción.** Crean usuarios y no hay forma de
limpiarlos después.

**El límite en local.** `api` limita login y registro a 20 peticiones por IP cada 15
minutos ([API.md § Límite de peticiones](API.md)). Tras dos o tres ejecuciones
seguidas, los e2e locales devuelven `429`. Dos formas de resolverlo:

- Apagarlo a propósito: `RATE_LIMIT=off` en el `.env` de `micasaestuya-infra` y `docker compose up -d express`. Solo funciona en `development`; en producción se ignora.
- O reiniciar el contenedor para poner la cuenta a cero: `docker compose restart express`.

**Datos de prueba.** El usuario del fixture (`cypress/fixtures/users.json`) es
`e2e.user@example.com`, no un email real. Su nombre y apellido **solo pueden llevar
letras**: `SignUp.vue` los valida con `alphaSpaceRegex`, y un nombre con dígitos
(como `E2E`) deja el formulario sin enviar, y el test falla con `No request ever occurred`.

### Cómo funciona Cypress aquí

- `describe` / `context` agrupan specs (context es alias de describe, se usa para subgrupos legibles). `it` es el caso individual.
- `before` corre una vez por bloque; `beforeEach` corre antes de cada it. auth.cy.ts usa before para crear el usuario de prueba una sola vez, y beforeEach para visitar / en cada test.
- Los comandos cy.* se encolan y reintentan automáticamente hasta cumplir la aserción o hacer timeout (defaultCommandTimeout: 10000 en cypress.config.ts) — por eso se encadena cy.get(...).should(...) en vez de usar await.
- Comandos custom en cypress/support/commands.ts y commands/auth.command.ts: getByTestId (wrapper de [data-cy="..."]), loginUI, signUpUI, openAuthModal, ensureUserExists, loginAPI.
- cy.intercept(...).as('alias') + cy.wait('@alias') para esperar peticiones reales sin usar timeouts fijos.
- Fixtures: cy.fixture('users') lee cypress/fixtures/users.json, tipado con cypress/types/fixtures.ts.

## api

- Unit/integración: node:test nativo (npm test), con --test-concurrency=1 (secuencial entre archivos). supertest para tests de integración de endpoints.
- En CI corre contra contenedores efímeros mongo:6 / redis:7-alpine — nunca toca datos reales.
