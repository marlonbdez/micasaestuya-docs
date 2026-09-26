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
stores/                       -> Pinia (auth, region, global, adFlow, listingDraft)
layouts/                      -> default (completo) y minimal (header y footer mínimos, para Publicar)
plugins/services.ts           -> provee $services
```

Las reglas completas de estilo de `web` están en `micasaestuya-web/CLAUDE.md` y
`micasaestuya-web/docs/design-system.md`.

## Criterio general: simple y fiable

Se aplica a todos los repos, y es la versión de código de los principios de
[product-vision.md](product-vision.md) (menos es más, sobriedad y fiabilidad):

- **Lo que nadie usa se borra:** código, iconos, tokens, dependencias, claves de
  traducción. Nada "por si acaso".
- **Antes de escribir algo a mano o añadir una dependencia, mira lo que ya hay.**
  En `web`, los componentes `Base*`, los tokens y VueUse.
- **Un solo camino para cada cosa:** los mismos componentes, los mismos tokens,
  los mismos patrones. Lo repetido con pequeñas variaciones es una lista de
  datos, no varias copias.
- **Ligero:** los iconos y otros recursos pesan lo mínimo, y el color lo pone el
  CSS con una variable, no el fichero.
- **Verificar de verdad:** lint y tests, y el flujo en el navegador, con y sin
  sesión, en claro y en oscuro, en móvil y en escritorio.
- **Robusto en los bordes:** validar en el sitio donde se puede saltar
  (formularios, endpoints) y no fiarse de un valor por defecto.

## Git / CI

- web: hooks de husky. `pre-commit` corre `npm run lint:fix` y los tests
  unitarios; `pre-push` corre `npm run build`. api no tiene hooks.
- El lint de web incluye Prettier sobre todo el repo, **también los `.md`**: un
  `CLAUDE.md` o un doc sin formatear rompe la CI.
- data-cy como convención de test id en templates (no data-test-id, ya migrado).
