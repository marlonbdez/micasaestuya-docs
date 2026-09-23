---
title: Architecture
---
# Arquitectura

## Diagrama de alto nivel

```
[Usuario] -> [web (Nuxt 3, Netlify)] -> [api (Express, Render)] -> [MongoDB Atlas]
                                                                 -> [Redis (Upstash)]
```

- `web`: SSG/Nuxt, se sirve estático desde Netlify (CDN), sin servidor Node corriendo en producción para el frontend.
- `api`: Express corriendo como Web Service en Render, habla con Mongo Atlas (datos de negocio: usuarios, propiedades) y con Redis/Upstash (caché de localización).
- No hay comunicación directa entre `web` y las bases de datos — todo pasa por `api`.

## web — capas internas

```
web/
  core/
    services/repository/
      fetchFactory.ts      <- clase base HTTP (antes HttpFactory)
      modules/
        auth.ts             <- AuthModule
        region.ts            <- RegionModule
    types.d.ts
    httpClient.ts
  composables/
    useAuthToken.ts         <- wrapper de useCookie() para el JWT
  stores/
    auth.ts                 <- Pinia
    region.ts                <- Pinia
  plugins/
    services.ts             <- provee $services (auth, region)
  sentry.client.config.ts
```

## api — capas internas

```
routes/        -> define rutas, llama al controller
controllers/    -> lógica HTTP (req/res)
models/         -> lógica de negocio + queries Mongoose
schemas/        -> Mongoose schemas
utils/          -> config, logger, middleware, redisClient
```

Regla: no hay lógica de negocio en routes/controllers, ni queries Mongoose fuera de models/.

## Por qué estas decisiones

Ver [Decisiones (ADRs)](ADRs.md).
