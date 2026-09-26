---
title: ADRs
---
# Decisiones de arquitectura (ADRs)

Registro corto de decisiones importantes y el porqué — 3-5 líneas cada una, no ensayos.

## ADR 001 — Abandono de Kubernetes, migración a PaaS

Se intentó desplegar con k8s (web/api tenían jobs deploy a un cluster). Se desistió por complejidad operativa sin equipo dedicado a infra. Se migró a Netlify (web), Render (api), MongoDB Atlas y Upstash (Redis) — todos PaaS administrados, sin cluster que mantener.

## ADR 002 — web se despliega vía Netlify nativo, no Docker

web es un Nuxt 3 generado estático (SSG). Netlify lo construye directo del repo (git) y lo sirve por su CDN — rápido, gratis, sin servidor que mantener. Se descartó que Netlify consumiera la imagen Docker publicada en el Container Registry: no aporta nada para un sitio estático y hubiera significado salir del hosting nativo de Netlify. La imagen de web se sigue publicando como respaldo/histórico, sin consumidor activo.

## ADR 003 — Redis en Upstash, no en Render

Redis de producción vive en Upstash, no como servicio de Render. (Corrige documentación anterior que decía "Render.com" — era incorrecta.)

## ADR 004 — Node 22-alpine estandarizado en los Dockerfiles

api ya estaba en Node 22-alpine en todos lados (Dockerfiles + CI). web estaba en 20-alpine. Se igualó el Dockerfile de web a 22-alpine (ago 2026) para evitar drift entre entornos de producción. Pendiente: los jobs de lint/test/build de la CI de web (GitHub Actions) todavía corren en Node 20 — no se ha migrado ese lado.

## ADR 005 — Alojamiento de código en GitHub

Los repos de código (web, api, infra) están alojados en GitHub bajo la cuenta `marlonbdez`, con nombres prefijados (`micasaestuya-web`, `micasaestuya-api`, `micasaestuya-infra`). CI/CD corre en GitHub Actions (ver [CI/CD](CI-CD.md)), las imágenes se publican en el Container Registry de GitHub (`ghcr.io`), y el repo cuenta con CodeQL (análisis de seguridad estático) y Dependabot (actualización de dependencias). La documentación también vive en GitHub, en su propio repo, `micasaestuya-docs`. (Corrige documentación anterior que la situaba en una wiki de GitLab, `docs.wiki`: ya no se usa.)

## ADR 006 — Pivote: de portal inmobiliario a plataforma de intercambio de alojamiento por colaboración

Se abandona la idea original de micasaestuya como portal de compraventa/alquiler de inmuebles. El nuevo modelo conecta a personas que ofrecen alojamiento y comida con personas dispuestas a colaborar en tareas domésticas u otras a cambio — en la línea de Workaway, Worldpackers o HelpX, pero pensado desde el Caribe (Cuba y República Dominicana como mercado de lanzamiento) con vocación global desde el principio. La plataforma es gratuita y se sostiene solo con donaciones voluntarias (sin comisión, sin cuota); actúa como simple conector — anfitrión y viajero negocian y cierran el trato por canales externos (WhatsApp, videollamada), la plataforma no media ni verifica identidades más allá de un login social. Toda la documentación técnica anterior sobre anuncios de propiedades queda obsoleta y se está reescribiendo. Detalle completo en [product-vision.md](product-vision.md).

## ADR 007 — Sin roles de producto: se quita `User.role`

`User` tenía `role: guest | host | admin` sin que nada lo usara. Se quita. Una misma cuenta puede publicar su casa y viajar, así que anfitrión y viajero no son roles sino lo que cada uno hace: alguien es anfitrión si tiene al menos un `Listing` (`Listing.exists({ owner })`), sin guardarlo aparte, para que no pueda desincronizarse. Guardarlo en el JWT, además, lo dejaría congelado: el token no caduca. El único uso real de un rol serían los permisos de moderación (`product-vision.md` § Confianza), que no están en el MVP; cuando lleguen se añade lo mínimo que pidan, con su propio ADR. En los textos se dice "viajero", no "huésped". El código se retira en `api` y `web` en la misma tarea que implemente `Listing`.

## ADR 008 — Los e2e corren contra una API efímera; staging, aplazado

Los e2e de `web` apuntaban a la API real de Render. Sus tests de auth creaban usuarios en la base de datos de producción (con un email real), y todos salían de la IP compartida del runner de GitHub, así que dos ejecuciones seguidas (la de la PR y la del push) agotaban el límite de peticiones y devolvían `429`.

**Decisión:** los tests nunca hablan con producción. En la CI, el job de e2e levanta una API efímera (servicios `mongo` y `redis` vacíos y la `api` clonada y arrancada con `NODE_ENV=test`), y se destruye al acabar. En local, el límite se puede apagar a propósito con `RATE_LIMIT=off`, que **se ignora en producción**; por defecto el límite nunca está apagado.

**Alternativas descartadas:**
- Desactivar el límite en producción para los tests: debilita la seguridad de producción y no evita que se ensucie la base real.
- Una base de staging en el mismo clúster de Atlas: comparte credenciales con producción, y un error de configuración podría tocarla.
- Un entorno de staging completo ya: sobredimensionado con tan pocos usuarios.

**Staging** se monta cuando haga falta probar cambios de base de datos antes de aplicarlos en producción (ver [Entornos y ramas](Environments-and-Branches.md) § Staging).
