# micasaestuya — Documentación

Plataforma de intercambio de alojamiento y comida por colaboración. Empieza
por aquí: **[product-vision.md](product-vision.md)** dice qué es el proyecto y
por qué; **[status.md](status.md)** dice dónde estamos ahora mismo.

Este repo es la fuente de verdad técnica y de producto del proyecto entero.
Los `CLAUDE.md` de cada repo de código remiten aquí para el porqué — ellos
solo explican cómo se escribe código en ese repo concreto.

## Repos

| Repo  | GitHub                                                              | Stack                                  |
| ----- | -------------------------------------------------------------------- | --------------------------------------- |
| docs  | (este repo)                                                           | Documentación — fuente de verdad         |
| web   | [marlonbdez/micasaestuya-web](https://github.com/marlonbdez/micasaestuya-web) | Nuxt 3, Vue 3, TypeScript, Pinia, SCSS |
| api   | [marlonbdez/micasaestuya-api](https://github.com/marlonbdez/micasaestuya-api) | Node.js, Express, MongoDB, Redis, JWT |
| infra | [marlonbdez/micasaestuya-infra](https://github.com/marlonbdez/micasaestuya-infra) | Docker Compose (entorno local)        |

## Despliegue

| Servicio        | Plataforma    | URL                                     |
| ---------------- | ------------- | ----------------------------------------- |
| web (frontend)   | Netlify       | https://www.micasaestuya.com              |
| api (backend)    | Render.com    | https://micasaestuya-api.onrender.com     |
| MongoDB          | MongoDB Atlas | —                                          |
| Redis            | Upstash       | —                                          |

Netlify y Render despliegan directo desde los repos de GitHub.

## Índice

- [Visión de producto](product-vision.md) — qué es, por qué, modelo económico, principios
- [Estado del proyecto](status.md) — dónde estamos, qué falta, deuda anotada
- [Arquitectura](Architecture.md)
- [Vocabulario del dominio](Domain-Vocabulary.md) — `locale` / `region` / `address`
- [Entornos y ramas](Environments-and-Branches.md)
- [Infraestructura y despliegue](Infrastructure-and-Deployment.md)
- [CI/CD](CI-CD.md)
- [Variables de entorno](Environment-Variables.md)
- [Base de datos](Database.md)
- [API](API.md)
- [Testing](Testing.md)
- [Onboarding](Onboarding.md)
- [Convenciones de código](Code-Conventions.md)
- [Decisiones de arquitectura (ADRs)](ADRs.md)
