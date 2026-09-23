---
title: Database
---
# Base de datos

## Producción

- MongoDB Atlas: fuente de datos de negocio (usuarios, alojamientos, estancias).
- Upstash (Redis): caché de datos de localización/regiones (Cuba + República Dominicana).

## Desarrollo

Hoy no existe base de datos de desarrollo en la nube — solo Mongo/Redis local vía infra/docker-compose.yml. Ver [Entornos y ramas](Environments-and-Branches.md).

## Tests

test:unit (api, en CI) corre contra contenedores mongo:6 / redis:7-alpine efímeros, creados y destruidos en cada pipeline — nunca toca Atlas ni Upstash.

## Modelos existentes (api)

| Modelo | Fuente | Notas |
|--------|--------|-------|
| User | Mongo (schemas/user) | JWT auth, bcrypt |
| Region | Redis | Caché de regiones (autocompletado y jerarquía) |

Pendientes para el MVP (ver `product-vision.md`): `Listing` (el alojamiento que ofrece un anfitrión) y, más adelante, `Stay` (una estancia confirmada). Todavía no existen — hoy el backend solo tiene `User` y `Region`.
