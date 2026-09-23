---
title: Environments and Branches
---
# Entornos y ramas

Hoy solo existe producción.

| Entorno | Rama | web | api | Mongo | Redis |
|---------|------|-----|-----|-------|-------|
| Producción | main | Netlify (micasaestuya.com) | Render (micasaestuya-api.onrender.com) | Atlas | Upstash |
| Local | cualquiera | localhost:3000 | localhost:3001 | Mongo en docker-compose | Redis en docker-compose |

La CI (GitHub Actions) también dispara en push/PR a `develop`, pero esa rama todavía no existe en ningún repo — es config a futuro, no un entorno real hoy.
