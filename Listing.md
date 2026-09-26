---
title: Listing
---

# Listing — especificación para `api`

> **Estado: implementado** en `api` (`POST /api/listings`, PR #20 de `api`) y
> conectado desde Publicar en `web` (PR #31 de `web`). Si el código y este
> documento no coinciden, manda el código; avísalo aquí.

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

Decisión en el [ADR 009](ADRs.md): las fotos viven en **Cloudflare R2** y en Mongo
solo van las URLs. Hoy **todavía no se suben**: las fotos que el anfitrión elige
se quedan en su navegador (IndexedDB) y no viajan en la petición.

**Reglas del producto**

- De **1 a 7 fotos** por alojamiento.
- Cada foto se optimiza en el navegador antes de subirla: **WebP** (JPEG de reserva
  si el navegador no sabe codificar WebP), lado mayor de **1280 px**, sin EXIF, y una
  **miniatura de 400 px** para Explorar. Estos valores sustituyen a los de
  `useImageResize` (1600 px, JPEG) cuando se implemente.

**Cómo se subirían** (diseño previsto, aún sin código)

```mermaid
sequenceDiagram
  participant N as Navegador
  participant A as api
  participant R as Cloudflare R2
  participant M as Mongo
  N->>N: reduce la foto y hace la miniatura
  N->>A: POST /api/listings/:id/photos (JWT)
  A->>A: ¿el alojamiento es tuyo? ¿caben más fotos?
  A-->>N: URL firmada y temporal
  N->>R: sube el fichero directo
  N->>A: confirma la subida
  A->>M: guarda la URL en photos
```

La `api` no recibe los ficheros: el servidor de Render es pequeño y no debe cargar
con imágenes. Mongo solo guarda `photos: [String]` (URLs públicas), vacío por
defecto: añadir la subida no cambia la forma del documento ni obliga a migrar nada.

**Reglas de seguridad**

- `POST /api/listings` **ignora** `photos` si llega en el body, para que nadie
  pueda meter URLs arbitrarias.
- La `api` solo acepta URLs que apunten a nuestro bucket y a ficheros que ella
  misma autorizó.
- Si se quita una foto o se borra el alojamiento, se borra también el fichero de
  R2 (si no, quedan huérfanos ocupando espacio).

**Por decidir al diseñarlo**

- Cómo limitar el peso y el tipo de fichero en la URL firmada de R2.
- Si el alojamiento se puede publicar sin foto mientras se suben (la regla "mínimo
  1" pide que, al final, tenga al menos una).
- El dominio público de las imágenes: `r2.dev` es para pruebas; en producción
  conviene uno propio.

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
| 400    | Falta un campo o no cumple las reglas | `{ "error": "..." }`, del `errorHandler` común (como `/api/users`)   |
| 401    | Sin token o token inválido            | Lo que ya devuelve el middleware `auth`                             |

`owner` sale de `req.user.id` (lo pone el middleware `auth`); si viene en el
body, se ignora.

**En `web`:** la llamada vive en
`micasaestuya-web/core/services/repository/modules/listing.ts`.

## Publicar y los roles

Decidido en el [ADR 007](ADRs.md): **no hay roles de producto** y `User.role`
se quita.

- Publicar no cambia nada en `User`.
- "Es anfitrión" se deduce de tener al menos un `Listing`
  (`Listing.exists({ owner })`); no se guarda aparte.
- La tarea que implemente `Listing` retira también `role`: del schema de
  `User`, del payload del JWT (`UserModel.login` y `UserModel.create`) y del
  tipo `IUserInfo` de `web`. Los documentos que ya lo tengan en Mongo no
  molestan: el campo se ignora al no estar en el schema.

## Fuera de esta especificación

Listar (`GET /api/listings`), ver el detalle, editar y borrar. Van con las
pantallas Explorar y Detalle, que son tareas aparte.
