# Visión de producto

**Este documento sustituye la idea original de micasaestuya como portal de compraventa/alquiler de inmuebles.** Ver [ADRs](ADRs.md) (ADR 006) para el porqué del cambio. El resto de la documentación técnica (wiki, `CLAUDE.md` de cada repo, `docs/` de este repo) todavía describe en parte el modelo anterior y se está reescribiendo para reflejar esto — si algo contradice a este documento, gana este documento.

## Qué es

Una plataforma que conecta a personas que ofrecen alojamiento y comida con personas dispuestas a colaborar unas horas al día en tareas domésticas u otras, a cambio. Sin dinero de por medio entre anfitrión y viajero — el intercambio es alojamiento + comida por colaboración.

Ya existen plataformas parecidas (Workaway, Worldpackers, HelpX, WWOOF), de pago para el viajero y centradas en Europa/Oceanía, o —en el caso de Worldpackers— ya fuerte en Centro y Sudamérica. micasaestuya arranca en el Caribe (Cuba y República Dominicana) con vocación de crecer a cualquier país del mundo.

## A quién sirve

Todavía no se sabe con precisión, y está bien que sea así por ahora: "personas que buscan libertad" es la intuición de partida, no un perfil cerrado. Se irá afinando con uso real en vez de inventarlo en un documento.

- **Anfitrión** — quien ofrece alojamiento (normalmente también comida) a cambio de ayuda.
- **Viajero** — quien busca alojamiento y comida a cambio de colaborar.

## Cómo funciona

1. El anfitrión publica su alojamiento: qué ofrece, qué tareas necesita, cuántas personas puede recibir a la vez.
2. El viajero busca y ve los alojamientos disponibles.
3. Si le interesa uno, contacta al anfitrión por WhatsApp (o el medio que el anfitrión indique).
4. Anfitrión y viajero hablan, negocian y cierran el trato **por su cuenta, fuera de la plataforma** — videollamada, WhatsApp, lo que prefieran.

micasaestuya no media en ese acuerdo, no lo verifica ni lo garantiza. Mensaje explícito a los usuarios: **nosotros solo os ponemos en contacto.**

## Modelo económico

Gratuito. Sin comisión, sin cuota de acceso, sin planes de pago. Se sostiene con donaciones voluntarias (enlace tipo Buy Me a Coffee) destinadas únicamente a cubrir desarrollo y hosting — no es un modelo de ingresos, es altruista y de crecimiento personal, tanto para quien lo usa como para quien lo construye.

Si algún día hace falta financiación adicional para escalar, la vía a explorar primero es apoyo de fundaciones u organizaciones afines (turismo sostenible, intercambio cultural) — no cobrar a los usuarios.

## Confianza y seguridad

Con presupuesto cero, la seguridad no puede apoyarse en verificación cara (documentos de identidad, comprobación de antecedentes). Se construye con piezas baratas que suman:

- **Login social** (LinkedIn, redes) en vez de solo email — sube el listón sin coste.
- **Reputación acumulada** — reseñas ligadas a estancias reales, no a cualquiera.
- **Botón de reportar/bloquear** perfiles, con moderación humana básica.
- **Edad mínima 18 años.**
- **Términos de uso explícitos**: la plataforma no verifica identidades, no media en el acuerdo entre las partes, no garantiza nada de lo que ocurra durante la estancia.

Esto no elimina el riesgo — lo declara con claridad desde el principio, que es la única defensa real cuando no hay presupuesto para más.

## Principios de diseño y desarrollo

- **Menos es más.** Cada función se justifica por una necesidad real observada, no por parecer buena idea. Ante la duda, no se construye.
- **Filosofía Toyota — sobriedad y fiabilidad, sin florituras.** El objetivo no es una interfaz vistosa, es una herramienta simple que funciona siempre y no da sorpresas.
- **Hecho para servir.** El proyecto existe para conectar a personas, no para crecer por crecer ni para generar ingresos.
- **Software libre.** El código vive en GitHub, abierto a quien quiera leerlo o contribuir, aunque no se espera que nadie lo haga hasta que el proyecto funcione y demuestre utilidad.

## Alcance geográfico

Lanzamiento: Cuba y República Dominicana (ya existe árbol de regiones para ambos). Vocación de crecer a cualquier país desde el diseño — no se codifica nada que asuma "solo Caribe" salvo donde sea estrictamente necesario (idioma, referencias de moneda en los textos).

## Qué es el MVP (y qué no)

**El MVP es esto, y nada más:**

- Un anfitrión puede publicar su alojamiento (qué ofrece, qué tareas pide, cuántas personas caben a la vez).
- Un viajero puede ver los alojamientos publicados.
- El viajero puede contactar al anfitrión por WhatsApp.
- Ahí termina la responsabilidad de la plataforma — el acuerdo lo cierran ellos, fuera de la app.

**Explícitamente fuera del MVP** (se añade después, solo si el uso real lo pide):

- Login social como requisito de confianza.
- "Confirmar estancia" dentro de la app (registro del acuerdo, bloqueo de plazas por fecha).
- Gestión automática de disponibilidad por fechas y capacidad múltiple.
- Reseñas y reputación acumulada.
- Cualquier tipo de pago, suscripción o cuota.

## Próximos pasos

1. Actualizar la documentación técnica existente (wiki y `CLAUDE.md` de cada repo) para que deje de describir un portal inmobiliario.
2. Prototipo navegable de la interfaz antes de escribir código, para validar diseño visual y de flujo.
3. Implementación del MVP.
