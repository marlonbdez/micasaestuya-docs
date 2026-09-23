# Estado del proyecto

**Este documento caduca.** Dice dónde estábamos la última vez, no cómo funciona
nada. Si contradice al código, gana el código: comprueba con `git log`.
El "por qué" de las decisiones vive en los otros documentos de `docs/`.

### Cómo trabajar en este proyecto

Reglas de trabajo acordadas, y no son de cortesía: se llegó a ellas después de
romper cosas por saltárselas.

- **Consultar antes de cambiar código que funciona.** Aunque parezca una mejora
  obvia. Hubo un caso de "arreglar" las claves `region.levelN` que habría roto
  República Dominicana, porque solo se miraron los ficheros base.
- **Explicar cualquier cambio no pedido**, incluidos los renombrados. Si aparece
  en el diff algo que nadie pidió, va explicado.
- **Preguntar antes de tocar componentes `Base*`** u otro código compartido.
- **De menos a más.** Primera iteración, lo justo necesario, poco código y
  sencillo de mantener. Nada de valores a mano: los enums y los tipos que ya
  existen.
- **Código en inglés, URLs traducidas** por locale.
- **Verificar en el navegador**, no dar por hecho que funciona.
- Las explicaciones apuntan a alguien de nivel _mid-junior_: el objetivo es que
  se entienda el porqué, no solo que compile.

### Cómo levantar y comprobar

El stack va con Docker Compose desde el repo `infra` (`docker-compose.yml`,
servicios `nuxt`, `express`, `mongo`, `redis`).

- Web en `http://localhost:3000`, API en `http://localhost:3001/api`.
- **El lint de `api` no se puede correr desde el host**: `api/node_modules` está
  vacío ahí porque las dependencias viven en el volumen de Docker. Va con
  `docker compose exec express npm run lint`.
- En `web` sí: `npm run lint` (ESLint + Prettier) y `npm run test:unit:headless`.
- Si Nitro falla con `EADDRINUSE .../worker.sock`, es un socket muerto de un
  apagado sucio, no un puerto ocupado:
  `docker compose up -d --force-recreate nuxt`.
- Sembrar Redis: `npm run redis:seed` **borra la base entera** (`flushdb`) antes
  de cargar. No se lanza para "comprobar" nada.

### Estado de los repos

Tres repos independientes: `api`, `web`, `infra`. Ahora mismo `api` y `web`
están **commiteados y limpios**.

`api` no tiene hooks de git. `web` sí: `pre-commit` corre `lint:fix` y los
tests, y `pre-push` corre `npm run build`.

**Al tocar los dos, `api` va primero**, porque `web` depende de su endpoint.
Entre un push y otro la web queda rota unos minutos.

### Hecho y verificado en navegador

Pasos 1 y 2 completos, el layout minimalista del flow, la barra de navegación
con `canGoNext`, el filtro `level_type` en el backend, el endpoint
`/regions/children`, la cascada provincia/municipio/localidad con restauración
al volver atrás, el vocabulario de tres palabras de `regions.md`
(`locale` / `region` / `address`), y el informe de Code Quality servido por
ESLint en vez de Code Climate (`tooling.md` § 3).

También el **paso 3** —precio, superficie, habitaciones y baños— y el mensaje de
campo obligatorio bajo la dirección del paso 2. Ahí se cerró el reparto de la
validación: vee-validate y yup explican qué está mal, `canGoNext` sigue siendo
quien habilita el botón (`post-ad-flow.md` § 8).

### Escrito, pendiente de verificar en el navegador

El **paso 4, fotos**. El lint pasa, pero nadie lo ha abierto todavía. Qué mirar:

- Que el indicador marque 04/05 y que "Siguiente" se encienda con una foto.
- Que al salir del paso y volver, las miniaturas se repinten desde IndexedDB.
- Que al quitar una foto desaparezca también de la base: se ve en las DevTools,
  en Application › IndexedDB › `micasaestuya` › `ad-photos`.
- Que una foto vertical de móvil no salga tumbada: es lo que evita
  `imageOrientation` en `useImageResize.ts`.

El **paso 5, descripción**. Solo el textarea de descripción, obligatorio; no
lleva "extras" todavía (ver deuda anotada). Qué mirar:

- Que el indicador marque 05/05 y que "Siguiente" (ahora "Publicar") se
  encienda al escribir algo.
- Que el mensaje de campo obligatorio aparezca al salir del campo vacío, igual
  que en la dirección del paso 2.
- Que el texto sobreviva a recargar la página (se persiste en el borrador de
  `localStorage`, como el resto de campos de texto).

Nuevo componente base: `BaseTextarea.vue`, calcado de `BaseInput.vue` pero con
`<textarea>`. No existía ninguno para texto largo.

### Lo siguiente, por orden

1. **"Extras" del paso 5.** Queda pendiente de definir: no hay vocabulario de
   dominio para amenidades/extras. `IProperty.amenities: string[]` no vale de
   referencia, es residuo del prototipo de IA (`post-ad-flow.md` § 7).

2. **La pantalla de publicar**, con el login embebido y el envío al backend, que
   hoy no existe. Ahí es donde el anuncio pasa a Mongo y donde toca decidir si
   la dirección se guarda desnormalizada —lo habitual en clasificados— o
   referenciada.

3. **La subida de las fotos.** Hoy se quedan en el navegador: la `api` no tiene
   endpoint de ficheros ni sitio donde guardarlos, así que un anuncio publicado
   todavía no llevaría imágenes.

### Deuda anotada

Nada de esto bloquea el paso 5.

**Tests**

- Tests del store `adFlow`: `persist`, `hydrate`, JSON corrupto, `reset`,
  `canGoNext`.
- Tests del backend para `/children` y para el filtro por nivel.
- Del paso 4, lo que es lógica pura y se prueba fácil: el `case 'photos'` de
  `canGoNext` y la reconciliación de ids sin fichero de `useAdPhotos.load()`.

**Bugs pequeños, con su diagnóstico ya hecho**

- `locales/es-cu.json` tiene **la clave `home` repetida dos veces**. Al parsear
  gana la segunda, así que `home.hero.title` y `home.cuba_banner` de la primera
  se están perdiendo ahora mismo. Hay que decidir qué textos valen.
- `modals.locale` conserva `region_label`, `language_label` y `save_button`, que
  no los usa nadie.
- Las fotos del paso 4 no se pueden reordenar ni elegir cuál es la principal:
  manda el orden en que se añadieron. La primera se marca como "Principal".
- `adFlow.reset()` vacía el borrador y las fotos, pero **no lo llama nadie
  todavía**: hasta que exista la pantalla de publicar, un anuncio terminado se
  queda en el navegador para siempre.
- **Dos pestañas del flow se pisan.** Cada una tiene su copia del borrador en
  memoria y la última que escribe gana. Con texto costaba unos bytes; con fotos
  cuesta megas, porque los ids que se pierden dejan los ficheros huérfanos en
  IndexedDB sin que nadie los borre nunca. Se arregla escuchando el evento
  `storage` del navegador en `hydrate`, y eso toca los cinco pasos: mejor con el
  flow terminado.
- La moneda está escrita a mano como "USD" en las traducciones. Los mercados son
  Cuba y República Dominicana y no hay sistema de monedas.

**Refactors con motivo**

- Los tres flags mutables de `useRegionSuggest` (`debounceTimer`,
  `isRequestBlocked`, `skipNextSearch`) más la llamada recursiva de `search()`
  desde su propio `finally`. Es el único sitio donde Code Quality señalaba deuda
  real (`tooling.md` § 2).
- Sembrar Redis emite **634.932 `console.log`** y hace **615.548 `zadd`
  secuenciales**, uno por prefijo. Agruparlos en un `pipeline` de ioredis es lo
  que puede hacer que sembrar staging deje de doler.
- El residuo del prototipo de IA, detallado en `post-ad-flow.md` § 7.

---
