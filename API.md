---
title: API
---
# API

Base: api/CLAUDE.md tiene el detalle completo de convenciones; esta página es el resumen de referencia rápida.

## Endpoints actuales

```
GET  /api/health           -> health check
GET  /api/users            -> lista de usuarios (requiere auth)
POST /api/users/create     -> registro
POST /api/users/login      -> login (devuelve JWT)
GET  /api/users/current    -> usuario actual (requiere auth)
GET  /api/users/profile    -> alias de /current (requiere auth)
GET  /api/regions/suggest  -> autocompletado de regiones (Redis)
GET  /api/regions/children -> hijos de una región (Redis)
```

## Autenticación

- JWT en header Authorization: `Bearer <token>`.
- Token sin expiración todavía (pendiente: expiración + refresh).
- Roles pendientes: guest | host | admin.
- Middleware auth en utils/middleware.js.

## Manejo de errores

- express-async-errors captura errores async automáticamente.
- Middleware errorHandler centralizado.
- No usar try/catch en controllers — se propaga al error handler. Sí se puede usar en models para logging puntual.
