---
title: Code Conventions
---
# Convenciones de código

## api — arquitectura de capas

```
routes/        -> solo define rutas y llama al controller
controllers/    -> lógica HTTP (req/res), llama al model
models/         -> lógica de negocio + queries Mongoose
schemas/        -> Mongoose schemas
utils/          -> config, logger, middleware, redisClient
```

Prohibido:

- Lógica de negocio en routes o controllers.
- Queries Mongoose en controllers (van en models).
- console.log en producción — usar utils/logger.js.
- Passwords en texto plano o en logs.
- Exponer passwordHash en respuestas de la API.

## web — arquitectura relevante

```
core/services/repository/    -> FetchFactory + módulos (AuthModule, RegionModule)
core/types.ts + core/types/  -> tipos, siempre en .ts (nunca .d.ts)
composables/useAuthToken.ts  -> wrapper de useCookie() para el JWT
stores/                       -> Pinia (auth, region, global, adFlow)
plugins/services.ts           -> provee $services
```

Las reglas completas de estilo de `web` están en `micasaestuya-web/CLAUDE.md` y
`micasaestuya-web/docs/design-system.md`.

## Git / CI

- web: hooks de husky. `pre-commit` corre `npm run lint:fix` y los tests
  unitarios; `pre-push` corre `npm run build`. api no tiene hooks.
- El lint de web incluye Prettier sobre todo el repo, **también los `.md`**: un
  `CLAUDE.md` o un doc sin formatear rompe la CI.
- data-cy como convención de test id en templates (no data-test-id, ya migrado).
