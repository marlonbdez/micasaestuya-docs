---
title: Infrastructure and Deployment
---
# Infraestructura y despliegue

| Servicio | Plataforma | Cómo despliega | URL |
|----------|-----------|-----------------|-----|
| web | Netlify | Build+deploy nativo desde el repo de GitHub, directo desde main. No usa Docker. | https://www.micasaestuya.com |
| api | Render.com | Web Service conectado al repo de GitHub (`marlonbdez/micasaestuya-api`). | https://micasaestuya-api.onrender.com |
| MongoDB | MongoDB Atlas | Cluster de producción. | — |
| Redis | Upstash | Base de producción. | — |

k8s: abandonado, no se usa.

## Container Registry: GitHub Container Registry (ghcr.io)

| Repo | Path del registry | Estado |
|------|--------------------|--------|
| api | ghcr.io/marlonbdez/micasaestuya-api | Activo, Render lo consume. |
| web | ghcr.io/marlonbdez/micasaestuya-web | Publicado, sin consumidor (Netlify no usa Docker). |
