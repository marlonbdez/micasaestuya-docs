---
title: Testing
---
# Testing

## web

- Unit: Vitest (test:unit, test:unit:headless, test:unit:coverage).
- E2E: Cypress 13.4.0 (cypress/e2e/auth.cy.ts, cypress/e2e/i18n.cy.ts).

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
