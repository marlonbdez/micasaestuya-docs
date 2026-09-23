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
composables/useAuthToken.ts  -> wrapper de useCookie() para el JWT
stores/                       -> Pinia (auth, region)
plugins/services.ts           -> provee $services
```

## Git / CI

- lint-staged en pre-commit hook (reemplaza git add -u).
- data-cy como convención de test id en templates (no data-test-id, ya migrado).
