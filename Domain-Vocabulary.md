---
title: Domain Vocabulary
---
# Vocabulario del dominio

Tres palabras que ya se han nombrado mal alguna vez en este proyecto.
Compartidas entre `web` y `api` — un cambio aquí afecta a los dos repos.

| Palabra   | Qué es                                                              | Nunca es                    |
| --------- | -------------------------------------------------------------------- | ----------------------------- |
| `locale`  | País + idioma (`es-CU`, `en-DO`)                                     | Una región ni una dirección  |
| `region`  | Un nodo del árbol administrativo (provincia, municipio, localidad)   | Nunca lleva calle ni coordenadas |
| `address` | La región **más** la calle y el número                              | No sustituye a `region`      |

Viene del vocabulario de schema.org, que es el que Google lee: `PostalAddress`
tiene `addressRegion` para la división administrativa y `streetAddress` para la
calle, ambas dentro de la dirección.

**La regla que no hay que romper:** una región nunca lleva calle ni
coordenadas. Si algún día hay coordenadas, van en `address`, al lado de
`region`, nunca dentro.

**Por qué son tres y no dos.** Durante un tiempo hubo dos palabras para lo
mismo y ninguna para una tercera cosa — arreglarlo costó dos renombrados. No
merece la pena volver a pasar por eso.

**Excepciones vivas que no hay que "corregir":**

- Las claves de Redis siguen siendo `regions-index:`, `regions-data`, etc. —
  ya eran correctas antes del renombrado.
- En `web/core/localeUtils.ts` el campo se llama `country`, no `region`, a
  propósito, para que "region" siga significando una sola cosa. El texto que
  ve el usuario en `LocaleModal` sí dice "Región" — es copy, no código, y no
  se cruza con nada.

## Dónde está la implementación

- **web** — componentes (`RegionCascade`, `RegionSuggest`) y cómo se
  construyó la cascada: `micasaestuya-web/docs/regions.md`.
- **api** — el modelo, el índice de Redis y sus trampas:
  `micasaestuya-api/docs/gotchas.md`.
