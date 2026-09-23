---
title: Listing
---

# Listing — especificación para `api`

> **Estado: propuesta.** Todavía no hay código de `Listing` en `api`: solo existe
> la pantalla Publicar en `web`, conectada a un servicio simulado. Este documento
> es el contrato que ese servicio simula. Se revisa antes de implementarlo.

Un `Listing` es el alojamiento que publica un anfitrión: qué ofrece, qué tareas
pide a cambio y cuántas personas caben a la vez (`product-vision.md` § Qué es el
MVP). Los campos son exactamente los del formulario de `web`
(`micasaestuya-web/pages/publish-listing/index.vue`) y los tipos salen de
`micasaestuya-web/core/types/listing.ts`.

## Schema (`schemas/listing.js`)

| Campo         | Tipo                  | Obligatorio | Reglas                                                                                                |
| ------------- | --------------------- | ----------- | ----------------------------------------------------------------------------------------------------- |
| `title`       | String                | sí          | `trim`, 1–100 caracteres                                                                              |
| `region`      | Subdocumento `region` | sí          | Ver [Región](#región)                                                                                 |
| `description` | String                | sí          | `trim`, no vacío                                                                                      |
| `tasks`       | [String]              | sí          | Al menos 1. Enum: `COOKING`, `GARDENING`, `CLEANING`, `CHILDCARE`, `PET_CARE`, `MAINTENANCE`, `OTHER` |
| `capacity`    | Number                | sí          | Entero, 1–20                                                                                          |
| `whatsapp`    | String                | sí          | `+`, código de país y número, sin espacios: `^\+[1-9]\d{7,19}$`                                       |
| `photos`      | [String]              | no          | Por defecto `[]`. Ver [Fotos](#fotos)                                                                 |
| `owner`       | ObjectId → `User`     | sí          | Lo pone la API desde el token, nunca el cliente                                                       |
| `createdAt`   | Date                  | automático  | `timestamps: true`                                                                                    |
| `updatedAt`   | Date                  | automático  | `timestamps: true`                                                                                    |

`toJSON` como en `User`: `_id` pasa a `id` y se quita `__v`.

`web` ya envía el WhatsApp normalizado (sin espacios ni guiones, en
`stores/listingDraft.ts` · `toCreateInput`), pero la API valida igual: el
cliente no es de fiar.

### Región

Se guarda con la misma forma que ya produce `RegionCascade` en `web` (el tipo
`IAdRegion`, que es `IRegion` sin `highlighted_text`). No es una forma nueva:

```js
region: {
  country_code: String, // 'CU' | 'DO', obligatorio
  level1: String,       // provincia, obligatorio
  level2: String,       // municipio
  level3: String,       // localidad (CU) o sector (DO)
  level_type: Number,   // 1 | 2 | 3: cuántos niveles hay, obligatorio
  term: String          // "Matanzas, Cárdenas, Varadero", para pintar
}
```

- **Nunca lleva calle ni coordenadas** (`Domain-Vocabulary.md`). Si algún día
  hacen falta, irán en un `address` al lado de `region`, no dentro.
- `level_type` tiene que coincidir con los niveles rellenos.
- **La región llega hasta el último nivel que existe en su rama.** `web` no deja
  publicar a medias. La API debería comprobarlo contra el árbol en memoria que ya
  usa `/regions/children` (`data/regions_*.json`): si el último nivel enviado
  tiene hijos, es un 400. Así la validación no depende de que el cliente se
  porte bien.

### Fotos

En esta primera versión **no se suben ficheros**: no hay presupuesto ni
almacenamiento decidido. Las fotos que el anfitrión elige se quedan en su
navegador (IndexedDB) y no viajan en la petición.

- El schema ya tiene `photos: [String]` (URLs públicas), vacío por defecto.
  Añadir la subida más adelante no cambia la forma del documento ni obliga a
  migrar nada.
- `POST /api/listings` **ignora** `photos` si llega en el body, para que nadie
  pueda meter URLs arbitrarias antes de que exista la subida.
- Cuando se decida el almacenamiento, lo previsible es un endpoint aparte
  (`POST /api/listings/:id/photos`) que reciba los ficheros ya reescalados por
  `web` (1600 px, JPEG, sin EXIF: ver `micasaestuya-web/docs/post-ad-flow.md` § 9)
  y devuelva las URLs.

## Endpoint de creación

```
POST /api/listings
Authorization: Bearer <token>        (mismo JWT que /api/users/*)
Content-Type: application/json
```

Ruta con el middleware `auth` ya existente, como en el ejemplo de
`micasaestuya-api/CLAUDE.md` § Rutas:
`listingRouter.post('/', auth, ListingController.create)`.

**Body:**

```json
{
  "title": "Casa con jardín en Playa Girón",
  "region": {
    "country_code": "CU",
    "level1": "Matanzas",
    "level2": "Ciénaga de Zapata",
    "level3": "Playa Girón",
    "level_type": 3,
    "term": "Matanzas, Ciénaga de Zapata, Playa Girón"
  },
  "description": "Casa tranquila cerca del mar…",
  "tasks": ["COOKING", "GARDENING"],
  "capacity": 2,
  "whatsapp": "+5351234567"
}
```

**Respuestas:**

| Código | Cuándo                                | Body                                                                |
| ------ | ------------------------------------- | ------------------------------------------------------------------- |
| 201    | Creado                                | El `Listing` completo (`id`, `owner`, `photos: []`, `createdAt`, …) |
| 400    | Falta un campo o no cumple las reglas | `{ "message": "..." }`, como los 400 de `/regions`                  |
| 401    | Sin token o token inválido            | Lo que ya devuelve el middleware `auth`                             |

`owner` sale de `req.user.id` (lo pone el middleware `auth`); si viene en el
body, se ignora.

**Enchufarlo en `web`:** el único fichero que cambia es
`micasaestuya-web/core/services/repository/modules/listing.ts`. El cuerpo
simulado de `create()` se sustituye por la llamada real, que ya está escrita en
el comentario de ese método.

## `User.role` al publicar — propuesta a confirmar

Está marcado como decisión abierta en `status.md`. La propuesta:

- **Publicar no cambia `role`.** Un usuario sigue siendo `guest` aunque publique.
- "Es anfitrión" **se deduce** de tener al menos un `Listing`
  (`Listing.exists({ owner })`), no se guarda aparte. Así no pueden
  desincronizarse (un `host` sin alojamientos, o alguien con alojamientos que
  sigue siendo `guest`).
- Una misma persona puede ser anfitrión y viajero a la vez. Un campo único no
  puede representarlo sin volverse un array.
- `role` se queda para **permisos**: `admin` para la moderación (reportar,
  bloquear, `product-vision.md` § Confianza). `host` queda sin uso y se podría
  quitar del enum cuando se confirme.

La alternativa es promocionar a `host` en el `create` (un `updateOne` en el
mismo model). Es más simple de consultar, pero trae los problemas de
desincronización de arriba y no resuelve el caso de quien es las dos cosas.

## Fuera de esta especificación

Listar (`GET /api/listings`), ver el detalle, editar y borrar. Van con las
pantallas Explorar y Detalle, que son tareas aparte.
