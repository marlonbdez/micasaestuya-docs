# Estado del proyecto

_Última revisión: 26-09-2026._

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
  documentación de `docs`, `web` y `api` está alineada. En `infra`, la rama
  `docs/align-after-pivot` **nunca se fusionó**: `main` sigue en `e456005`.
- **Prototipo del MVP hecho**, sobre el sistema de diseño real de `web`: Explorar,
  Detalle de alojamiento (con el botón de WhatsApp), Publicar y Confirmación.
- **Publicar un alojamiento funciona de punta a punta** (web #26, #31; api #20):
  - `web`: `/publicar-alojamiento` (un solo formulario: título, región con
    `RegionCascade` hasta el último nivel, fotos, descripción, tareas,
    capacidad y WhatsApp) y `/publicar-alojamiento/publicado`. Borrador en el
    store `listingDraft` (`localStorage`) y fotos en IndexedDB. Identificarse se
    pide solo al pulsar Publicar (`AuthModal`).
  - `api`: `Listing` y `POST /api/listings` con auth, según
    [Listing.md](Listing.md). La región se valida contra el árbol de regiones.
    Límite de peticiones (`express-rate-limit`) en login, registro y publicar.
  - **Las fotos todavía no se suben**: se quedan en el navegador.
  - **Falta probarlo con un login real** (solo se ha probado con sesión
    simulada).
- **Sin roles** ([ADR 007](ADRs.md)): `User.role` retirado de `api` y `web`.
  Anfitrión es quien tiene al menos un alojamiento. En los textos, "viajero".
- **`web` sin restos visibles del portal**: título de la web, menú de usuario
  ("Mis alojamientos (próximamente)", deshabilitada) y páginas del footer
  (quiénes somos, FAQ, mapa web y legales; **las legales son un borrador** sin
  revisión legal) (web #30).
- **Dependencias**: `main` de `web` se rompió al fusionar actualizaciones
  mayores de Dependabot; se revirtió (web #27) y Dependabot de `web` ya no
  propone versiones mayores. **`api` todavía no tiene esa configuración** y
  tiene 15 PR de Dependabot abiertas, casi todas mayores: no fusionarlas.
- **Render construye `api` desde el repo de GitHub** (confirmado en el panel),
  no desde la imagen de `ghcr.io`. La documentación aún dice lo contrario.
- `/post-ad` sigue en el código, sin enlazar. No se borra: se moverá a una
  carpeta aparte cuando se decida qué se reaprovecha.

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

1. **Dependabot en `api`**: la misma configuración que `web` (sin versiones
   mayores, menores agrupadas) y cerrar las PR de versiones mayores.
2. **Probar el login y el registro reales** dentro de Publicar, en local.
3. **Corregir la documentación de despliegue**: Render construye desde GitHub.
   Dejarlo en `Infrastructure-and-Deployment.md`, `README.md`, `CI-CD.md` y
   `micasaestuya-api/CLAUDE.md` ("lo consume Render").
4. **Cabos sueltos**:
   - Sentry graba el 100 % de las sesiones con `maskAllText: false` y
     `blockAllMedia: false` (`web/sentry.client.config.ts`): ve textos como el
     WhatsApp. Bajar el muestreo y enmascarar.
   - `api/utils/config.js` tiene una contraseña de Mongo por defecto escrita en
     el código de un repo público.
   - "Compartir en redes sociales" del footer apunta a `#`.
   - El círculo con la inicial del usuario en el header apenas se ve.
   - La rama `docs/align-after-pivot` de `infra`: revisarla y fusionarla o
     descartarla.
5. **`web` + `api`: Explorar y Detalle**, las pantallas que faltan del
   prototipo, con `GET /api/listings` (listar y ver uno). Con Detalle vuelven
   "Ver mi alojamiento" en la Confirmación y "Mis alojamientos" en el menú.
6. **Más adelante, planificado**: migrar `web` a Nuxt 4 (Pinia, Vitest,
   `nuxt-icons`, ESLint 9), que es lo que pedían las versiones mayores.

Fuera del MVP, y no se empieza sin que el uso real lo pida: login social,
confirmar estancias, disponibilidad por fechas, reseñas, pagos
(`product-vision.md` § Qué es el MVP).

---

## Optimizaciones para una PR propia

Ideas para que el código siga siendo simple, ligero y fiable sin depender de que
alguien se acuerde. Ninguna está empezada: se eligen y se hacen en una PR pequeña
cada una, o agrupadas si tocan lo mismo. El criterio general está en
[Code-Conventions.md](Code-Conventions.md) § Criterio general.

**Automatizar lo que hoy es una regla escrita**

1. **Comprobar los iconos solos.** Un script en el hook o en la CI que falle si un
   SVG de `assets/icons/` tiene más de un `<path>`, trae colores o pesa de más, salvo
   las banderas y los usados como `background-image`
   (`micasaestuya-web/docs/design-system.md` § Iconos).
2. **Detectar lo que sobra** con una herramienta como `knip` (ficheros, exports y
   dependencias sin uso). Ya conocidos: `HomeSearchVacaciones` y `TheHero`, que no
   se pintan en la home, y los restos de la deuda anotada.
3. **Presupuesto de peso** del bundle y de las fuentes en la CI, para notar cuándo
   algo engorda.
4. **Accesibilidad más allá de la home.** `pa11y-ci` ya corre en la CI, pero solo
   sobre la home: ampliarlo a Publicar y al menú de usuario abierto, y a Explorar y
   Detalle cuando existan.
5. **Plantilla de PR** con una lista corta: claro y oscuro, móvil y escritorio, con
   y sin sesión, y lo que no se pudo probar.

**Deuda concreta que salió al construir el menú**

6. **Términos del registro obligatorios de verdad.** `SignUp.vue` los valida con
   `bool().required()`, que deja pasar `false` (`micasaestuya-web/docs/gotchas.md` § 11).
7. **Los e2e de la CI van contra la API real, y el límite de peticiones los tumba.**
   El job de e2e apunta a `https://micasaestuya-api.onrender.com/api`: los tests de
   auth **crean usuarios en la base de datos de producción** y comparten la IP del
   runner de GitHub. `authLimiter` (20 peticiones por IP cada 15 minutos en login y
   registro) devuelve `429` en cuanto hay dos ejecuciones seguidas (la de la PR y la
   del push): visto en `web#33`, donde fallaron «logs in successfully» y «tries to
   sign up with an existing email». En local pasa lo mismo tras dos o tres
   ejecuciones, porque `express` corre en `development` y el límite solo se salta con
   `NODE_ENV=test` (se arregla con `docker compose restart express`).
   Propuesta: e2e contra una API efímera en la CI (servicios `mongo` y `redis`, y la
   `api` arrancada con `NODE_ENV=test`, que ya se salta el límite), y en local un
   opt-out explícito (`RATE_LIMIT=off`) en `api/utils/rateLimit.js` y en el
   `docker-compose.yml` de `infra`. El límite no se desactiva nunca por defecto.
8. **Iconos recoloreables también en los formularios.** Pasar `check`,
   `error-circle`, `location`, `search` y `chevron-down` de `background-image` a
   `mask` con una variable (toca `BaseInput`, `BaseCheckbox`, `BaseSelect`,
   `BaseTextarea` y `BaseFileInput`); permitiría borrar `chevron-down-white`.
9. **Borrar el modelo anterior** cuando se decida qué se reaprovecha: `/post-ad`,
   el store `adFlow` y sus iconos (`apartments`, `buildings`, `garage`,
   `landscape`).

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

## Para retomar en una sesión nueva con Claude

Lo aprendido trabajando que no está en otro sitio:

- **Leer primero**, en este orden: `product-vision.md`, `ADRs.md`, este
  documento, y el `CLAUDE.md` y `docs/` del repo que se toque.
- **Git**: rama por tarea desde `main` actualizado (`git pull` antes), PR en
  GitHub y el usuario fusiona. Un commit por punto cuando la PR junta varios.
  Nada de commits sin que el usuario lo pida.
- **Orden al tocar `api` y `web`**: `api` primero y se fusiona antes, porque
  Render despliega `api` al fusionar en `main` y `web` depende del endpoint.
- **Verificar antes de dar algo por hecho**: lint y tests en local
  (`web`: `npm run lint`, `npm run test:unit:headless`; `api`: dentro del
  contenedor, `docker compose exec express npm run lint` y `npm test`), y el
  flujo en el navegador contra `localhost:3000`.
- **Dependencias**: no fusionar actualizaciones mayores de Dependabot sin
  planificarlas; se hacen a mano, una a una. Instalar paquetes de `api` con
  `docker compose exec express npm install <paquete>`.
- **Claude no crea cuentas ni escribe contraseñas**: las pruebas con login real
  las hace el usuario, o se le da un usuario de prueba que ya exista.
- **El límite de peticiones también aplica en local**: si una prueba lo agota,
  `docker compose restart express` lo reinicia.

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
- `web` y `api` sin tests de extremo a extremo del flujo de Publicar con login
  real.
- Solo `POST /api/listings` y `/api/users/login|create` tienen límite de
  peticiones; el resto de rutas con auth, no.

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
