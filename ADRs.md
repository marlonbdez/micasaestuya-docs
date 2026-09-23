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
