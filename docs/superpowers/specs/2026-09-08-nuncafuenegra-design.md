# nuncafuenegra.com — Diseño

**Fecha:** 2026-09-08
**Estado:** aprobado, pendiente de plan de implementación

---

## 1. Tesis

El color negro de una aceituna es **tiempo** o es **química**, y la lata no te dice cuál.

La inmensa mayoría de las «aceitunas negras» que se venden en lata son aceitunas
recogidas **verdes**, tratadas con álcali, oxidadas por inyección de aire y fijadas
con una sal de hierro. La aceituna negra de verdad es la que maduró en el árbol
varios meses más y se curó en sal o salmuera.

Marco editorial, y no es negociable en ninguna línea del sitio:

> **No te envenenan. Te engañan.**

El sitio no argumenta que el producto industrial sea peligroso. Argumenta que
**no es lo que su nombre dice que es**, que la diferencia es verificable por
cualquiera en su propia despensa, y que el lector merece saberlo.

---

## 2. Objetivos

Cuatro capas, en este orden. La utilidad va al final, nunca al principio: la
narrativa se gana el derecho a dar consejos.

1. **Entender el proceso** — los dos caminos, explicados con rigor.
2. **Saber mirar la etiqueta** — el lector sale con una habilidad, no con una lista.
3. **Querer compartirlo** — la pieza tiene que funcionar como objeto viral.
4. **Acabado de portfolio** — justifica gastar en detalle de diseño y movimiento.

### No objetivos

- No es una tienda, ni un comparador, ni un directorio de marcas.
- No hay CMS, ni backend, ni base de datos, ni analítica de terceros invasiva.
- No hay modo oscuro. Un solo tema, fijo, cálido.
- No hay versión en inglés en la v1 (el copy se guarda aparte del markup para no
  cerrar la puerta).

---

## 3. Público y tono

**Público:** hispanohablante adulto, curioso, no experto. Alguien que ha comido
aceitunas negras toda su vida y nunca se ha preguntado por qué son negras.

**Tono:** divulgación seria y cálida. Ni tono de denuncia histérica ni tono de
paper. Frases cortas. Concreción antes que adjetivos. El dato hace el trabajo
que en otros sitios hace la indignación.

**Filo editorial:** se señala **la práctica**, nunca las marcas.

- Sí: explicar que es legal, que está permitido, y que está mal etiquetado de facto.
- Sí: recomendar **variedades** (Empeltre / negra de Aragón, Cuquillo, Kalamata).
- No: nombrar marcas, ni fotografiar envases reales, ni reproducir identidades
  comerciales. La etiqueta de ejemplo del sitio es **genérica y compuesta**, y se
  presenta explícitamente como ejemplo.

Razón: nombrar marcas exige verificar y mantener al día formulaciones que cambian,
envejece pésimamente en un sitio estático, y expone a reclamaciones sin añadir
nada al argumento.

---

## 4. Arquitectura de contenido

### 4.1 La narrativa (`/`)

Documento único, diez pantallas. **Sin menú de cabecera**: un header con
navegación mata la pantalla 1. Los enlaces a las páginas de respuesta viven en la
sección 10 y en un pie discreto.

| # | Sección | Función |
|---|---------|---------|
| 1 | **Apertura** | Fondo casi vacío. Una aceituna negra perfecta, uniforme, brillante. Titular: «Esta aceituna nunca estuvo negra.» Nada más. |
| 2 | **El color es una promesa** | Qué cree el lector que compra cuando compra «negras». |
| 3 | **El calendario del olivo** | Verde en septiembre → violeta → granate en enero. El color como paso del tiempo. *(acuarela)* |
| 4 | **La bifurcación** | El scroll se parte en dos columnas que nunca vuelven a juntarse. Diagrama SVG. Punto de no retorno del sitio. |
| 5 | **Camino fábrica** | Recogida en verde → baños de álcali → inyección de aire por tandas → fijación con sal de hierro (E-579) → deshuesado → esterilización en autoclave. Ritmo mecánico, tiempos en horas, todo en monoespaciada. |
| 6 | **Camino árbol** | Meses más en la rama → salmuera, sal seca, hierbas → fermentación. Tiempos en meses, todo en serif. *(acuarela)* |
| 7 | **La prueba** | El decodificador: lista de ingredientes genérica donde el E-579 se ilumina. «Tu lata te lo dice, en cuerpo 6.» |
| 8 | **Reconocerlas sin etiqueta** | Negra mate uniforme, blanda, deshuesada **vs** arrugada, tornasolada, violeta-marrón, amarga. Comparación en CSS/SVG, no en foto. |
| 9 | **Qué comprar** | Variedades y cómo pedirlas. Nunca marcas. |
| 10 | **Honestidad y fuentes** | *No es veneno: es otra cosa.* Qué se pierde, qué no pasa nada, por qué es legal. Fuentes citadas + crédito de ilustraciones IA + enlaces a las páginas de respuesta. |

**La sección 10 no es relleno.** Admitir en voz alta lo que el sitio *no* está
diciendo es lo que le da autoridad a las nueve anteriores. Un sitio que solo acusa
se lee como panfleto; uno que marca sus propios límites se lee como divulgación.

**Volumen de texto:** 1.400–1.800 palabras reales en la narrativa. Las pantallas
no pueden ser solo titular: cada una lleva prosa de verdad, tanto por el lector
como por el rastreador.

### 4.2 Páginas de respuesta

Cuatro páginas de texto sobrio, sin scroll narrativo, sin animación. Cada una
responde **una** pregunta y enlaza hacia la narrativa. Son el motor de tráfico
orgánico; la narrativa es el motor de conversión y de compartición.

| Ruta | Pregunta que responde | Consultas objetivo |
|------|----------------------|--------------------|
| `/aceitunas-negras-oxidadas/` | ¿Cómo se fabrica una aceituna negra? | aceitunas negras oxidadas · cómo se hacen las aceitunas negras · por qué son negras |
| `/e-579-gluconato-ferroso/` | ¿Qué es el E-579 y qué hace ahí? | e579 · gluconato ferroso · qué es el e-579 · aditivo aceitunas |
| `/variedades-aceituna-negra-natural/` | ¿Cuáles son negras de verdad? | aceituna negra de Aragón · empeltre · cuquillo · kalamata · aceituna negra natural |
| `/aceitunas-verdes-y-negras/` | ¿En qué se diferencian de verdad? | diferencia entre aceitunas verdes y negras · son la misma aceituna |

800–1.200 palabras cada una. Contenido **propio**, no resumen de la narrativa:
duplicar canibaliza.

---

## 5. Identidad visual

### 5.1 La idea que la sostiene

**La tipografía y la paleta no acompañan al argumento: lo ejecutan.**

- El **camino del árbol** se compone en serif — humano, lento, con italic real.
- El **camino de la fábrica** se compone en **monoespaciada** — la letra del
  albarán, del análisis, de la letra pequeña regulatoria.
- Y el golpe: **en toda la paleta no existe un negro puro**, salvo en el lado de la
  fábrica. `#000` aparece *únicamente* ahí, y rodeado de ocres y granates se ve
  muerto. El lector siente el argumento antes de leerlo.

### 5.2 Paleta

Un solo tema, `color-scheme: light only`. Toda cálida: prohibidos los grises y
los `rgba()` de negro para texto secundario (viran a gris sucio sobre el papel).

```
--papel      #EDE4D4   arena clara, con grano sutil
--tinta      #2B2318   marrón muy oscuro (nunca negro)
--tinta-70   #5C4F3C   secundario
--tinta-45   #8A7B63   terciario, aparato crítico
--linea      #C4B393
--oliva      #6E7444   epígrafes, numeración
--ocre       #B8792F   la sal, la luz de enero
--terracota  #9C5232   destacados, entradillas
--granate    #6B2B3E   EL ACENTO. La aceituna negra de verdad. El E-579.
--negro      #000000   SOLO camino fábrica. Prohibido en el resto del sitio.
```

### 5.3 Tipografía

Todas variables, **autoalojadas** y subseteadas a `latin` + `latin-ext`
(imprescindible: `ñ`, `¿`, `¡`, vocales acentuadas).

| Voz | Fuente | Uso |
|-----|--------|-----|
| Display | **Fraunces** — ejes `opsz`, `SOFT`, `WONK` | Titulares. `opsz` 144, `SOFT` 45, `WONK` 1. Cálida y orgánica con un desajuste deliberado. |
| Lectura | **Newsreader** — eje `opsz` | Cuerpo a 20px, medida 62–64 caracteres, italic real para las voces. |
| Aparato / fábrica | **IBM Plex Mono** 400/500 | Numeración de secciones, datos de proceso, tiempos, y la lista de ingredientes. |

Escala de contraste extremo, no escala modular: display hasta
`clamp(2.4rem, 7.8vw, 5.8rem)` contra cuerpo de 20px. El contraste brutal entre
display y cuerpo es lo que separa lo editorial de lo corporativo.

Carga: `preload` de Fraunces y Newsreader, `font-display: swap`, y métricas de
fallback (`size-adjust`, `ascent-override`) para que el intercambio no mueva nada.
**CLS objetivo: 0.**

### 5.4 Ilustración

8–10 acuarelas generadas con IA, dirección **mediterránea moderna**: aguadas
amplias y húmedas, bordes sangrados, sin línea de tinta, luz natural.

**Reparto de trabajo, y es una regla:**

- **La acuarela lleva la emoción** — el olivar, la rama, las manos, la salmuera, la nave.
- **SVG y CSS llevan el argumento** — la transformación de color, el diagrama de los
  dos caminos, la línea de proceso, el decodificador, el comparador.

Un dato no se dibuja en acuarela. Y al contrario: una ilustración no prueba un
color, así que **el rigor no lo carga la imagen**, lo cargan el diagrama, la
etiqueta reproducida como texto real y las fuentes citadas.

**Integración con el fondo:** las piezas llegarán sobre papel blanco. Se sirven en
**AVIF sobre blanco con `mix-blend-mode: multiply`**. El blanco desaparece, el
pigmento se asienta sobre el arena como si estuviera pintado ahí, y se mantiene el
formato comprimido (un PNG con alpha pesaría 4–5× y los bordes aguados se recortan
mal). Efecto lateral deseado: las aguadas se tiñen del arena, así que **la paleta
del sitio se impone a la de la IA** — que es exactamente lo que evita el aspecto
de «esto lo ha hecho una máquina».

**Divulgación:** línea de crédito en la sección 10. Un sitio que denuncia la
apariencia artificial y se ilustra con IA sin decirlo es munición para un lector
hostil. La acuarela lee como interpretación, no como prueba, y el crédito lo deja
por escrito.

#### Ancla de estilo para los prompts

El fallo típico de un sitio ilustrado con IA es que las piezas no parecen de la
misma mano. Se evita con un ancla literal, idéntica en las diez, y variando **solo**
la escena:

> `loose modern Mediterranean watercolour, wet-on-wet washes with bleeding edges,
> no ink outline, no linework, warm palette of olive green ochre terracotta and
> deep garnet, generous white paper margins, natural daylight, painterly and
> restrained, editorial illustration, no text, no lettering, no people's faces`

Escenas (una por pieza): rama con aceitunas verdes · rama en viraje violeta ·
rama madura granate y arrugada · olivar en enero con luz baja · manos recogiendo
en cesta · aceitunas en salmuera dentro de un tarro de vidrio · sal gruesa y
hierbas · nave industrial con depósitos y tuberías · lata abierta sobre mesa ·
suelo de olivar con red.

---

## 6. Movimiento y accesibilidad

- **Scroll-driven puro en CSS**: `animation-timeline: view()` y `scroll()`. **0 KB de JS.**
- Envuelto en `@supports (animation-timeline: view())`. Sin soporte, el sitio
  muestra los **estados finales estáticos** y se lee perfectamente.
- `prefers-reduced-motion: reduce` desactiva todo el movimiento. No negociable.
- **Regla de oro:** ninguna animación puede esconder texto del DOM. Prohibido
  `display: none` y `content-visibility: hidden` sobre contenido. Solo `opacity`,
  `transform` y `color` — el texto está siempre presente y siempre accesible para
  lector de pantalla y para rastreador.
- Contraste AA como mínimo en todo texto sobre `--papel`.
- La bifurcación de la sección 4 es una cuadrícula de dos columnas que en móvil
  se apila con un rótulo explícito por camino: la comparación no puede depender
  de la disposición lateral.
- Jerarquía semántica real: un `h1`, `h2` por sección, `section` con
  `aria-labelledby`.

---

## 7. Arquitectura técnica

**Astro 5**, prerenderizado estático, cero integraciones de framework. CSS vanilla
con `@layer` y custom properties. Sin Tailwind. Sin runtime de cliente.

```
nuncafuenegra/
├─ astro.config.mjs          # site, @astrojs/sitemap
├─ public/
│  ├─ robots.txt
│  └─ fonts/                 # woff2 variables subseteados
├─ src/
│  ├─ assets/acuarelas/      # originales; los procesa astro:assets
│  ├─ components/
│  │  ├─ narrativa/          # 01Apertura.astro … 10Fuentes.astro
│  │  ├─ figuras/            # Bifurcacion, LineaProceso, Decodificador, ComparadorColor (SVG inline)
│  │  └─ ui/                 # Acuarela.astro, Dato.astro, Fuente.astro, Nota.astro
│  ├─ content/
│  │  ├─ narrativa.ts        # copy de las 10 pantallas, separado del markup
│  │  ├─ fuentes.ts          # bibliografía con id, cita, url, fecha de consulta
│  │  └─ respuestas/*.md     # las 4 páginas de respuesta
│  ├─ layouts/
│  │  ├─ Narrativa.astro
│  │  └─ Respuesta.astro
│  ├─ pages/
│  │  ├─ index.astro
│  │  ├─ aceitunas-negras-oxidadas.astro
│  │  ├─ e-579-gluconato-ferroso.astro
│  │  ├─ variedades-aceituna-negra-natural.astro
│  │  └─ aceitunas-verdes-y-negras.astro
│  └─ styles/
│     ├─ tokens.css          # la paleta y la escala
│     ├─ base.css
│     └─ narrativa.css
└─ docs/superpowers/specs/
```

**Decisiones de fondo:**

- **Copy separado del markup** (`content/`). Permite revisión editorial sin tocar
  layout, y deja la puerta abierta a una versión en inglés sin reescribir componentes.
- **`fuentes.ts` como fuente única de verdad** de la bibliografía. Cada afirmación
  del copy referencia un `id`; la sección 10 y las páginas de respuesta renderizan
  la lista desde ahí. Así es imposible que una cita se quede huérfana.
- **`<Picture>` de `astro:assets`** para todas las acuarelas: AVIF + WebP, `srcset`,
  `width`/`height` explícitos siempre, `loading="lazy"` salvo la de la pantalla 1.
- **CSS crítico inline** en la narrativa; el resto diferido.
- **Un componente por pantalla.** Diez ficheros pequeños en vez de un `index.astro`
  de 900 líneas.

**Despliegue:** Cloudflare Pages (estático, CDN global, gratis). Cabeceras de
caché inmutable para fuentes e imágenes con hash.

---

## 8. Presupuesto de rendimiento

Son límites, no aspiraciones. Si una decisión de diseño rompe uno, se revisa la
decisión de diseño.

| Métrica | Límite |
|---------|--------|
| JavaScript enviado | **0 KB** |
| HTML + CSS crítico + fuentes | < 100 KB |
| Página completa, con acuarelas | < 500 KB |
| LCP (4G simulada) | < 1,2 s |
| CLS | **0** |
| Peticiones en el primer render | ≤ 6 |

---

## 9. SEO

**Expectativa realista, dicha sin adornos:** para `aceitunas negras` a secas no se
va a rankear — ese resultado es supermercado y receta, intención de compra. El
terreno ganable es el racimo de preguntas de la tabla 4.2, donde la competencia
son blogs mediocres y el tráfico es exactamente el del sitio.

Implementación:

- Prerenderizado completo. Nada de contenido dependiente de JS.
- `<title>` y `<meta description>` propios y escritos a mano en las cinco páginas.
- JSON-LD: `Article` en la narrativa, `FAQPage` en las páginas de respuesta.
- `og:image` **diseñada** — una acuarela compuesta con el titular en Fraunces, no
  un recorte. Es la mitad de la compartibilidad.
- `sitemap.xml` vía `@astrojs/sitemap`, `robots.txt`, `canonical`, `lang="es"`.
- Enlazado interno: respuestas → narrativa (la narrativa es el destino), y
  narrativa → respuestas solo desde la sección 10 y el pie.
- Los enlaces (backlinks) los tiene que traer la compartición. El diseño de la
  narrativa **es** la estrategia de enlaces; no hay otra.

---

## 10. Verificación de hechos — bloqueante

**Ninguna afirmación de esta lista se escribe en copy definitivo antes de tener
fuente citable en `fuentes.ts`.** El sitio renuncia a la foto como prueba, así que
la fuente es toda la prueba que hay. Esto se hace *antes* de redactar, no después.

| Afirmación | Estado | Dónde buscar |
|---|---|---|
| Las negras oxidadas se elaboran a partir de fruto verde o en envero | alta confianza, falta cita | Codex STAN 66-1981; norma de calidad española de aceituna de mesa |
| Secuencia del proceso: baños de álcali + aireación + fijación con sal de hierro + esterilización | alta confianza, falta cita | bibliografía técnica de elaboración de aceituna de mesa |
| **Tiempos concretos del proceso** (las «18 h» de la maqueta son un marcador, no un dato) | **sin verificar** | igual que arriba — sin cita no se publica ninguna cifra |
| E-579 (gluconato ferroso) y E-585 (lactato ferroso) autorizados en la UE específicamente para aceitunas negras oxidadas | alta confianza, falta la referencia exacta | Reg. (CE) 1333/2008, anexo II |
| Los aditivos deben figurar en la lista de ingredientes | alta confianza | Reg. (UE) 1169/2011 |
| **Qué obliga exactamente la norma sobre denominar «negras» a las oxidadas** — el punto crítico de todo el sitio | **sin verificar** | norma de calidad española vigente; verificar si sigue en vigor el RD que se cite |
| Pérdida de polifenoles frente a la curación por fermentación | dirección clara, faltan cifras y fuente | literatura científica revisada |
| Acrilamida en negras oxidadas | real, tratar con cuidado | dictamen de EFSA sobre acrilamida |
| Variedades españolas curadas negras de verdad (Empeltre / negra de Aragón, Cuquillo) y cuáles se destinan a oxidación (Hojiblanca, Manzanilla, Cacereña) | media, falta cita | denominaciones de origen y fuentes sectoriales |

Sobre la acrilamida: es un dato real y pertinente, pero el sitio se ha
comprometido con «no te envenenan, te engañan». Se menciona en la sección 10, con
fuente, y **con su contexto** — nunca como titular ni como amenaza. Convertirlo en
susto rompería el marco editorial y, además, sería peor periodismo.

---

## 11. Riesgos

| Riesgo | Mitigación |
|---|---|
| La ilustración IA se usa contra el argumento del sitio | Crédito explícito en la sección 10; la acuarela nunca ocupa el lugar de la prueba |
| Sin fuentes, el sitio es una opinión bonita | La sección 10 es bloqueante, y `fuentes.ts` obliga a que cada afirmación tenga id |
| Las diez acuarelas no parecen de la misma mano | Ancla de estilo literal e idéntica + `multiply` sobre el arena, que las unifica |
| `animation-timeline` sin soporte en algún navegador | `@supports` con estados finales estáticos; el sitio se lee igual sin una sola animación |
| Diez pantallas a pantalla completa con poco texto → contenido flaco para Google | Mínimo de 1.400 palabras en la narrativa, más las cuatro páginas de respuesta |
| Alcance creciente hacia comparador o directorio de marcas | Está en «no objetivos». Si vuelve, es un proyecto nuevo |

---

## 12. Decisiones abiertas

1. **Requisito pendiente del usuario.** Queda un requisito que se mencionó pero no
   se llegó a formular. Se incorpora como enmienda a este documento cuando se
   concrete, antes de escribir el plan de implementación si llega a tiempo.

Todo lo demás está cerrado: nombre, estructura, stack, paleta, tipografía,
tratamiento de imagen, filo editorial, presupuesto y SEO.
