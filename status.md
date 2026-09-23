# Estado del proyecto

_Última revisión: 23-09-2026._

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
  documentación de los cuatro repos (este, y los `CLAUDE.md`, README,
  CONTRIBUTING y `docs/` de `web`, `api` e `infra`) se alineó con el pivote en
  las ramas `docs/align-after-pivot`.
- **Prototipo del MVP hecho**, sobre el sistema de diseño real de `web`: Explorar,
  Detalle de alojamiento (con el botón de WhatsApp), Publicar y Confirmación.
- **El código sigue siendo el del modelo anterior.** No se ha escrito todavía
  nada del MVP:
  - `api` solo tiene `User` y `Region`. No existe `Listing`.
  - `web` tiene el flow de `/post-ad` (5 pasos, pensado para inmuebles). Qué se
    reaprovecha y qué se reescribe se decide al implementar el MVP;
    `micasaestuya-web/docs/post-ad-flow.md` ya lo avisa.
- **Primer paso hacia el prototipo en `web`**: el logo pasa de PNG a wordmark de
  texto (`feat/logo-text-wordmark`, PR #22). El lint de ese PR no falla por el
  logo: el `CLAUDE.md` de `main` no pasaba Prettier (se arregla en
  `docs/align-after-pivot` de `web`).

### Lo que sí se reaprovecha del modelo anterior

No es de inmuebles, es infraestructura que el MVP necesita igual:

- Regiones de Cuba y República Dominicana: el árbol en Redis, `/regions/suggest`,
  `/regions/children` y la cascada provincia/municipio/localidad de `web`
  (`Domain-Vocabulary.md`, `micasaestuya-web/docs/regions.md`).
- Locales, i18n y el selector de país/idioma (`LocaleModal`).
- Registro, login y JWT.
- El sistema de diseño (`micasaestuya-web/docs/design-system.md`) y los
  componentes `Base*`.

---

## Lo siguiente, por orden

1. **Fusionar las ramas `docs/align-after-pivot`** de los cuatro repos, y
   después el PR #22 del logo (su CI pasa en cuanto `main` de `web` tenga el
   `CLAUDE.md` formateado).
2. **Dos decisiones abiertas**, pequeñas, antes de escribir el MVP:
   - **Roles.** El schema de `User` ya tiene `role: guest | host | admin`. En el
     producto hay anfitrión y huésped (el viajero); falta decidir cómo se
     reflejan en `role` y, de paso, si el texto usa "huésped" o "viajero".
   - **Cómo despliega Render.** La documentación dice dos cosas: que el Web
     Service está conectado al repo de GitHub y construye desde ahí, y que
     consume la imagen de `ghcr.io/marlonbdez/micasaestuya-api`. Hay que mirarlo
     en el panel de Render y dejar una sola versión en
     `Infrastructure-and-Deployment.md`, `README.md` y `CI-CD.md`.
3. **Implementar el MVP**, poco a poco y con `api` primero, porque `web` depende
   de sus endpoints:
   - `api`: el modelo `Listing` (qué ofrece el anfitrión, qué tareas pide,
     cuántas personas caben, contacto de WhatsApp, ubicación con `region`) y sus
     rutas para publicar y listar. Los campos salen de la pantalla Publicar del
     prototipo y se acuerdan antes de escribir código.
   - `web`: las pantallas del prototipo, una a una.

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
