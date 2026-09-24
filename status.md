# Estado del proyecto

_Última revisión: 25-09-2026._

## Antes de nada: este documento no es la fuente de verdad

`status.md` dice **dónde estamos**, no **qué es el proyecto**. Caduca. Por eso,
antes de proponer o tocar nada, se lee en este orden:

1. **[product-vision.md](product-vision.md)** — qué es micasaestuya, el MVP y lo
   que queda fuera. **Manda sobre cualquier otro documento.**
2. **[ADRs.md](ADRs.md)**, sobre todo el **ADR 006** (el pivote).
3. **El prototipo del MVP** (enlazado en `product-vision.md` § Prototipo): cómo
   se ve y cómo fluye lo que hay que construir.
4. Este documento, para saber por dónde vamos.
5. El `CLAUDE.md` y los `docs/` del repo que se vaya a tocar, y **el código**.

Si algo contradice a `product-vision.md`, gana `product-vision.md`. Si este
documento contradice al código, gana el código: comprueba con `git log`.

### El pivote, en una línea

micasaestuya **ya no es un portal inmobiliario**. Es una plataforma gratuita que
pone en contacto a anfitriones (alojamiento y comida) con viajeros (colaboración
unas horas al día). El trato se cierra por WhatsApp, fuera de la plataforma.

**Cuidado con lo que queda del modelo anterior.** El código y parte de la
documentación de los repos todavía hablan de anuncios, inmuebles, compraventa,
`PropertyType`, `OperationType` o `/post-ad`. Eso es **el modelo viejo**: sirve
como referencia técnica de patrones, nunca como descripción del producto ni como
lista de tareas pendientes.

---

## Dónde estamos

- **Pivote decidido y documentado**: `product-vision.md` y ADR 006. La
  documentación de `docs`, `web` y `api` ya está alineada (ramas
  `docs/align-after-pivot` fusionadas). En `infra`, esa rama figura sin fusionar
  en la copia local: **hay que confirmarlo en GitHub**.
- **Prototipo del MVP hecho**, sobre el sistema de diseño real de `web`: Explorar,
  Detalle de alojamiento (con el botón de WhatsApp), Publicar y Confirmación.
- **Primera pantalla del MVP en `web`** (PR #26): Publicar alojamiento
  (`/publicar-alojamiento`) y Confirmación (`/publicar-alojamiento/publicado`),
  más el header y el footer rediseñados según el prototipo. Detalle:
  - Una sola página de formulario: título, región (`RegionCascade`, obligatoria
    hasta el último nivel), fotos, descripción, tareas de colaboración,
    capacidad y WhatsApp.
  - El borrador vive en el store `listingDraft` (`localStorage`) y las fotos en
    IndexedDB, **solo en el navegador**: todavía no se suben.
  - Identificarse se pide solo al pulsar Publicar, con el `AuthModal` que ya
    existía.
  - **El envío es simulado**: no existe `POST /api/listings`. El único fichero
    que cambia el día que exista es
    `micasaestuya-web/core/services/repository/modules/listing.ts`.
  - `/post-ad` sigue en el código, sin enlazar desde ningún sitio. No se borra:
    se moverá a una carpeta aparte cuando se decida qué se reaprovecha.
- **Especificación de `Listing` para `api`**: [Listing.md](Listing.md) (PR #3).
  Schema, región, fotos reservadas y contrato del endpoint.
- **Roles decididos** ([ADR 007](ADRs.md)): no hay roles de producto y
  `User.role` se quita. Anfitrión es quien tiene al menos un alojamiento. En los
  textos, "viajero".
- **`api` sigue sin nada del MVP**: solo `User` y `Region`.
- **Logo**: wordmark de texto fusionado (PR #25 de `web`).

### Lo que sí se reaprovecha del modelo anterior

No es de inmuebles, es infraestructura que el MVP necesita igual:

- Regiones de Cuba y República Dominicana: el árbol en Redis, `/regions/suggest`,
  `/regions/children` y la cascada provincia/municipio/localidad de `web`
  (`Domain-Vocabulary.md`, `micasaestuya-web/docs/regions.md`). El formulario de
  Publicar ya usa la cascada.
- Locales, i18n y el selector de país/idioma (`LocaleModal`).
- Registro, login y JWT (el `AuthModal`, ya usado en Publicar).
- El sistema de diseño (`micasaestuya-web/docs/design-system.md`) y los
  componentes `Base*`.

### Referencias

Plataformas parecidas para mirar el flujo y la ficha de un alojamiento (no el
modelo de negocio: todas cobran, micasaestuya no): Workaway, Worldpackers (la
más fuerte en Centro y Sudamérica), HelpX y WWOOF (`product-vision.md` § Qué
es).

---

## Lo siguiente, por orden

1. **`api`: `Listing` y `POST /api/listings`** según [Listing.md](Listing.md), y
   enchufarlo en `web` sustituyendo el servicio simulado. En la misma tarea se
   retira `User.role` de `api` y `web` ([ADR 007](ADRs.md)).
2. **Probar en `web` el login y el registro reales** dentro de Publicar. En la
   verificación de la PR #26 solo se simularon.
3. **`web`: Explorar y Detalle**, las pantallas que faltan del prototipo, con su
   endpoint de listado en `api`. Con Detalle vuelve el botón "Ver mi
   alojamiento" de la Confirmación.
4. **Cómo despliega Render** (decisión pequeña, sigue abierta). La
   documentación dice dos cosas: que el Web Service está conectado al repo de
   GitHub y construye desde ahí, y que consume la imagen de
   `ghcr.io/marlonbdez/micasaestuya-api`. Hay que mirarlo en el panel de Render
   y dejar una sola versión en `Infrastructure-and-Deployment.md`, `README.md` y
   `CI-CD.md`.

Fuera del MVP, y no se empieza sin que el uso real lo pida: login social,
confirmar estancias, disponibilidad por fechas, reseñas, pagos
(`product-vision.md` § Qué es el MVP).

---

## Cómo trabajar en este proyecto

Reglas de trabajo acordadas, y no son de cortesía: se llegó a ellas después de
romper cosas por saltárselas.

- **Leer antes de proponer.** La documentación de `micasaestuya-docs`, el
  prototipo y el repo que toque. No asumir nada de memoria ni quedarse con el
  primer documento que se abre.
- **Poco a poco.** Incrementos pequeños y revisables, sin tocar muchas cosas a la
  vez.
- **Consultar antes de cambiar código que funciona.** Aunque parezca una mejora
  obvia. Hubo un caso de "arreglar" las claves `region.levelN` que habría roto
  República Dominicana, porque solo se miraron los ficheros base.
- **Explicar cualquier cambio no pedido**, incluidos los renombrados. Si aparece
  en el diff algo que nadie pidió, va explicado.
- **Preguntar antes de tocar componentes `Base*`** u otro código compartido.
- **De menos a más.** Primera iteración, lo justo necesario, poco código y
  sencillo de mantener. Nada de valores a mano: los enums y los tipos que ya
  existen.
- **Partir de lo que ya existe** en el código real antes de inventar nada nuevo.
- **Código en inglés, URLs traducidas** por locale.
- **Verificar en el navegador**, no dar por hecho que funciona.
- Las explicaciones apuntan a alguien de nivel _mid-junior_: el objetivo es que
  se entienda el porqué, no solo que compile.

## Cómo levantar y comprobar

El stack va con Docker Compose desde el repo `infra` (`docker-compose.yml`,
servicios `nuxt`, `express`, `mongo`, `redis`).

- Web en `http://localhost:3000`, API en `http://localhost:3001/api`.
- **El lint de `api` no se puede correr desde el host**: `api/node_modules` está
  vacío ahí porque las dependencias viven en el volumen de Docker. Va con
  `docker compose exec express npm run lint`.
- En `web` sí: `npm run lint` (ESLint + Prettier) y `npm run test:unit:headless`.
  Prettier revisa **también los `.md`**: un `CLAUDE.md` o un doc de `web` sin
  formatear rompe la CI de cualquier PR.
- Si Nitro falla con `EADDRINUSE .../worker.sock`, es un socket muerto de un
  apagado sucio, no un puerto ocupado:
  `docker compose up -d --force-recreate nuxt`.
- Sembrar Redis: `npm run redis:seed` **borra la base entera** (`flushdb`) antes
  de cargar. No se lanza para "comprobar" nada.

`api` no tiene hooks de git. `web` sí: `pre-commit` corre `lint:fix` y los
tests, y `pre-push` corre `npm run build`. **Al tocar los dos, `api` va
primero**, porque `web` depende de su endpoint.

---

## Deuda anotada

Solo la que sigue valiendo después del pivote.

- `locales/es-cu.json` tiene **la clave `home` repetida dos veces**. Al parsear
  gana la segunda, así que `home.hero.title` y `home.cuba_banner` de la primera
  se pierden. Seguramente se reescriba con la home del MVP; hasta entonces, ojo.
- `modals.locale` conserva `region_label`, `language_label` y `save_button`, que
  no los usa nadie.
- Tests del backend para `/regions/children` y para el filtro por nivel.
- Los tres flags mutables de `useRegionSuggest` (`debounceTimer`,
  `isRequestBlocked`, `skipNextSearch`) más la llamada recursiva de `search()`
  desde su propio `finally`. Es el único sitio donde Code Quality señalaba deuda
  real (`micasaestuya-web/docs/tooling.md` § 1).
- Sembrar Redis emite **634.932 `console.log`** y hace **615.548 `zadd`
  secuenciales**, uno por prefijo. Agruparlos en un `pipeline` de ioredis es lo
  que puede hacer que sembrar staging deje de doler.
- JWT sin expiración ni refresh (`API.md`).
- Código muerto: `web/core/services/repository/modules/Authentication.ts` importa
  un `HttpFactory` que ya no existe; nadie lo usa (el módulo vivo es `auth.ts`).
  En `api`, el schema de `User` tiene un campo `notes` que referencia un modelo
  `Note` inexistente, y `requests/*.rest` prueba unas rutas `/notes` que tampoco
  existen: restos de una plantilla inicial.

- Restos del portal inmobiliario en `web` que ya se ven en pantalla: el
  `<title>` y la descripción de `nuxt.config.ts` ("Alquiler, compra y venta de
  casas"), la opción "Mis anuncios" del menú de usuario del header (lleva a
  `/properties`, con las claves `MY_ADS` y `LOGOUT` sin traducir) y la clave
  `header.advertise_property`, que ya nadie usa.
- Los enlaces del footer (`/about-us`, `/faq`, `/privacy-policy`,
  `/cookie-policy`, `/terms-and-conditions`, `/sitemap`) llevan a páginas que
  no existen y no pasan por `localePath`. "Apóyanos" no está porque todavía no
  hay URL de donaciones.
- Publicar: si el usuario cierra el `AuthModal` sin identificarse y después hace
  login desde el header estando en la misma página, se publica sin volver a
  pulsar. Cerrarlo del todo obliga a tocar `AuthModal`.
- `useDraftPhotos` (genérico) y `useAdPhotos` (el de `/post-ad`) duplican la
  misma lógica. Se resuelve cuando se retire `/post-ad`.
- `pages/login.vue` usa `v-model:checked` en `BaseCheckbox`, que no tiene esa
  prop: el "Recuérdame" de esa página no funciona. Es una página del modelo
  anterior; el login real va por el `AuthModal`.

**Conocido y dejado así a propósito:** `micasaestuya-infra/seed/` tiene su propia
copia de `regions_cu.json` y `regions_do.json` (iguales a las de `api/data/`), un
`regions.json` y un `mongo-init.js` que crea una colección `regions` en Mongo.
La `api` no la usa: la fuente de verdad de las regiones es `api/data/` y Redis
(`Domain-Vocabulary.md`, `micasaestuya-api/CLAUDE.md`). No es un error nuevo que
arreglar al pasar por ahí.

La deuda del flow de `/post-ad` (tests del store `adFlow`, reordenar fotos,
`adFlow.reset()` sin llamar, dos pestañas que se pisan, moneda "USD" en los
textos) **no se arrastra**: pertenece al modelo anterior. Si al implementar el
MVP se reaprovecha esa parte, se recupera del historial de git de este fichero.
