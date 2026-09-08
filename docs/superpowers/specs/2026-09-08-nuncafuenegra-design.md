# nuncafuenegra.com — Diseño

**Fecha:** 2026-09-08
**Revisión:** 2 — reencuadre editorial tras verificación de hechos (2026-09-08)
**Estado:** aprobado en forma y estructura; el marco editorial cambia respecto a la revisión 1

---

## 0. Aviso de la revisión 2

La revisión 1 daba por bueno el relato popular: que las aceitunas negras de lata
son aceitunas verdes **teñidas**, y que la lata te lo esconde. La verificación
demuestra que **eso es falso en dos puntos**, y que la historia real es más
precisa, más citable y —afortunadamente— más interesante.

Lo que cambia está en §1 y §10. Lo que **no** cambia: estructura de diez
pantallas, stack, paleta, tipografía, tratamiento de imagen, SEO y presupuesto.

---

## 1. Tesis

### Lo que es falso y el sitio no dirá

- **No las tiñen.** No se añade ningún colorante. El negro es pigmento de la
  propia aceituna: sus compuestos fenólicos se oxidan en medio alcalino y forman
  pigmentos de tipo melanina. Decir «teñidas» es incorrecto y hunde la
  credibilidad del sitio en la primera frase.
- **No es un fraude de etiquetado.** La etiqueta cumple la norma. El producto es
  una categoría legal reconocida, con su definición y su nombre.
- **No es un problema de salud.** No se insinuará que lo sea.

### Lo que sí es cierto, y es la tesis

El fruto se recoge **sin madurar**. Se le ennegrece a la fuerza en medio
alcalino. Y luego se le añade una sal de hierro **porque el resultado honesto no
es negro, es marrón oscuro, y el marrón oscuro no vende**.

Y el remate, que es la columna vertebral del sitio y está en el BOE:

> La lata está **obligada** a decirte el color.
> **No está obligada** a decirte el proceso.

El Real Decreto 679/2016 obliga a que la denominación incluya el color según su
artículo 4 —donde «Negras» significa oxidadas y «Negras naturales» significa
maduradas en el árbol— pero incluir el proceso de elaboración es **voluntario**
(art. 12.3.b).

Marco editorial, reformulado:

> **No te engañan con un tinte. Te engañan con una palabra.**

La palabra es **«naturales»**. Su ausencia es el dato. Todo lo demás —el E-579, la
textura, el negro uniforme— es confirmación.

### Consecuencia editorial

El sitio deja de ser una denuncia y pasa a ser algo mejor: **un curso de lectura
de etiquetas de tres minutos, envuelto en una historia bonita**. Mantiene todo su
filo, porque el filo ahora está en el BOE en vez de en una intuición.

El titular de la pantalla 1 sobrevive intacto, y ahora es literalmente cierto:

> **«Esta aceituna nunca estuvo negra.»**

---

## 2. Objetivos

Cuatro capas, en este orden. La utilidad va al final: la narrativa se gana el
derecho a dar consejos.

1. **Entender el proceso** — los dos caminos, con rigor.
2. **Saber mirar la etiqueta** — el lector sale con una habilidad, no con una lista.
3. **Querer compartirlo** — la pieza funciona como objeto viral.
4. **Acabado de portfolio** — justifica el gasto en diseño y movimiento.

### No objetivos

- No es tienda, comparador ni directorio de marcas.
- No hay CMS, backend, base de datos ni analítica invasiva.
- No hay modo oscuro. Un solo tema, cálido, fijo.
- No hay versión en inglés en la v1 (el copy se guarda aparte para no cerrar la puerta).
- **No hay alarmismo sanitario.** Ni sobre el hierro, ni sobre la acrilamida, ni
  sobre los aditivos.

---

## 3. Público y tono

**Público:** hispanohablante adulto, curioso, no experto. Alguien que ha comido
aceitunas negras toda su vida y nunca se ha preguntado por qué son negras.

**Tono:** divulgación seria y cálida. El dato hace el trabajo que en otros sitios
hace la indignación. Frases cortas, concreción antes que adjetivos.

**Filo editorial:** se señala **la práctica y el vacío de la norma**, nunca las marcas.

- Sí: explicar que es legal, que la etiqueta cumple, y que **la norma no obliga a
  declarar el proceso**.
- Sí: recomendar **variedades**.
- No: marcas, envases reales, identidades comerciales. La etiqueta de ejemplo es
  **genérica y compuesta**, presentada explícitamente como ejemplo.
- **No: la palabra «teñidas», ni «colorante», ni «fraude».** Prohibidas en todo el copy.

---

## 4. Arquitectura de contenido

### 4.1 La narrativa (`/`)

Documento único, diez pantallas. **Sin menú de cabecera.** Los enlaces a las
páginas de respuesta viven en la sección 10 y en un pie discreto.

| # | Sección | Función |
|---|---------|---------|
| 1 | **Apertura** | Casi vacío. Una aceituna negra perfecta y uniforme. «Esta aceituna nunca estuvo negra.» Nada más. |
| 2 | **El color es una promesa** | Qué cree el lector que compra. Y el desmontaje del bulo que él mismo puede haber oído: **no, no las tiñen** — lo que pasa es peor de explicar y más interesante. |
| 3 | **El calendario del olivo** | Verde en septiembre → envero → negra natural en pleno diciembre. El color como paso del tiempo. *(acuarela)* |
| 4 | **La bifurcación** | El scroll se parte en dos columnas que no vuelven a juntarse. Diagrama SVG. |
| 5 | **Camino fábrica** | Recogida sin madurar → salmuera → tratamientos alcalinos sucesivos con aireación entre ellos → **fijación con sal de hierro** → deshuesado → esterilización térmica. Monoespaciada, ritmo mecánico. Con el dato que lo explica todo: sin el hierro, el resultado es **marrón oscuro**. |
| 6 | **Camino árbol** | Meses más en la rama → salmuera, sal seca, hierbas → fermentación. Serif, tiempos en meses. *(acuarela)* |
| 7 | **La prueba** | El decodificador, ahora con **dos** pruebas: primero la **denominación** (busca la palabra «naturales»), después la **lista de ingredientes** (E-579 o E-585). La primera es la buena; la segunda confirma. |
| 8 | **Reconocerlas sin etiqueta** | Negro uniforme, blanda, dulce-neutra **vs** color desigual, arrugada, ácida-amarga. Comparación en CSS/SVG. |
| 9 | **Qué comprar** | Variedades y cómo pedirlas. Nunca marcas. |
| 10 | **Lo que no te estoy diciendo** | La sección de honestidad, reforzada: el contraargumento del CSIC citado y **enlazado**, la aclaración de que no es fraude ni riesgo sanitario, la acrilamida con su contexto, y las fuentes. |

**Volumen:** 1.400–1.800 palabras reales. Cada pantalla lleva prosa, no solo titular.

**Sobre la pantalla 2 y la 10.** El sitio ahora hace algo poco habitual: **desmonta
el bulo que le beneficiaría**. Eso no es un peaje, es el activo principal — es lo
que separa esto de los cientos de artículos que repiten «van teñidas». Un lector
que llega creyendo el bulo y sale sabiendo la verdad más fina es un lector que
comparte.

### 4.2 Páginas de respuesta

Cuatro páginas sobrias, sin scroll narrativo ni animación. Motor de tráfico
orgánico; la narrativa es el motor de compartición.

| Ruta | Pregunta | Consultas objetivo |
|------|----------|--------------------|
| `/aceitunas-negras-oxidadas/` | ¿Cómo se fabrica una aceituna negra? | aceitunas negras oxidadas · cómo se hacen · estilo californiano |
| `/e-579-gluconato-ferroso/` | ¿Qué es el E-579 y qué hace ahí? | e579 · gluconato ferroso · lactato ferroso e585 |
| `/aceitunas-negras-naturales/` | ¿Cuáles son negras de verdad y cómo se llaman? | aceitunas negras naturales · negra de Aragón · empeltre · kalamata |
| `/aceitunas-verdes-y-negras/` | ¿En qué se diferencian de verdad? | diferencia entre aceitunas verdes y negras · son la misma aceituna |

800–1.200 palabras cada una, contenido **propio**. Duplicar canibaliza.

---

## 5. Identidad visual

*(sin cambios respecto a la revisión 1)*

### 5.1 La idea que la sostiene

**La tipografía y la paleta no acompañan al argumento: lo ejecutan.**

- El **camino del árbol** en serif — humano, lento, con italic real.
- El **camino de la fábrica** en **monoespaciada** — la letra del albarán y de la
  letra pequeña regulatoria.
- Y el golpe: **en toda la paleta no existe un negro puro**, salvo en el lado de la
  fábrica. `#000` aparece *solo* ahí y, rodeado de ocres y granates, se ve muerto.

Esto encaja aún mejor con la tesis de la revisión 2: el negro absoluto y uniforme
**es precisamente el artificio**. El color honesto de una aceituna madura es
granate, violeta o castaño oscuro — nunca `#000`. La paleta del sitio es el
argumento del sitio.

### 5.2 Paleta

`color-scheme: light only`. Toda cálida: prohibidos los grises y los `rgba()` de
negro para texto (viran a gris sucio sobre el papel).

```
--papel      #EDE4D4   arena clara, con grano sutil
--tinta      #2B2318   marrón muy oscuro (nunca negro)
--tinta-70   #5C4F3C   secundario
--tinta-45   #8A7B63   terciario, aparato crítico
--linea      #C4B393
--oliva      #6E7444   epígrafes, numeración
--ocre       #B8792F   la sal, la luz de enero
--terracota  #9C5232   destacados, entradillas
--granate    #6B2B3E   EL ACENTO. La aceituna negra de verdad.
--negro      #000000   SOLO camino fábrica. Prohibido en el resto.
```

### 5.3 Tipografía

Variables, **autoalojadas** y subseteadas a `latin` + `latin-ext` (`ñ`, `¿`, `¡`,
acentos).

| Voz | Fuente | Uso |
|-----|--------|-----|
| Display | **Fraunces** (`opsz`, `SOFT`, `WONK`) | Titulares. `opsz` 144, `SOFT` 45, `WONK` 1. |
| Lectura | **Newsreader** (`opsz`) | Cuerpo 20px, medida 62–64 caracteres, italic real. |
| Aparato / fábrica | **IBM Plex Mono** 400/500 | Numeración, datos de proceso, lista de ingredientes. |

Contraste extremo, no escala modular: display hasta `clamp(2.4rem, 7.8vw, 5.8rem)`
contra cuerpo de 20px.

Carga: `preload` de Fraunces y Newsreader, `font-display: swap`, métricas de
fallback (`size-adjust`, `ascent-override`). **CLS objetivo: 0.**

### 5.4 Ilustración

8–10 acuarelas IA, dirección **mediterránea moderna**: aguadas amplias y húmedas,
bordes sangrados, sin línea de tinta, luz natural.

**Reparto de trabajo, y es una regla:**

- **La acuarela lleva la emoción** — el olivar, la rama, las manos, la salmuera, la nave.
- **SVG y CSS llevan el argumento** — la transformación de color, la bifurcación,
  la línea de proceso, el decodificador, el comparador.

Un dato no se dibuja en acuarela. Y al contrario: una ilustración no prueba nada,
así que **el rigor lo cargan el diagrama, la etiqueta reproducida como texto y las
fuentes citadas**.

**Integración con el fondo:** las piezas llegarán sobre papel blanco. Se sirven en
**AVIF sobre blanco con `mix-blend-mode: multiply`**. El blanco desaparece, el
pigmento se asienta sobre el arena, y se mantiene el formato comprimido (un PNG
con alpha pesaría 4–5× y los bordes aguados se recortan mal). Efecto lateral
deseado: **la paleta del sitio se impone a la de la IA**, que es lo que evita el
aspecto de «esto lo ha hecho una máquina».

**Divulgación:** línea de crédito en la sección 10. La acuarela lee como
interpretación, no como prueba, y el crédito lo deja por escrito.

#### Ancla de estilo para los prompts

Idéntica en las diez piezas; varía **solo** la escena:

> `loose modern Mediterranean watercolour, wet-on-wet washes with bleeding edges,
> no ink outline, no linework, warm palette of olive green ochre terracotta and
> deep garnet, generous white paper margins, natural daylight, painterly and
> restrained, editorial illustration, no text, no lettering, no people's faces`

Escenas: rama con aceitunas verdes · rama en envero · rama madura granate y
arrugada · olivar en diciembre con luz baja · manos recogiendo en cesta ·
aceitunas en salmuera en tarro de vidrio · sal gruesa y hierbas · nave industrial
con depósitos y tuberías · lata abierta sobre mesa · suelo de olivar con red.

---

## 6. Movimiento y accesibilidad

- **Scroll-driven puro en CSS**: `animation-timeline: view()` y `scroll()`. **0 KB de JS.**
- Envuelto en `@supports (animation-timeline: view())`. Sin soporte, **estados
  finales estáticos** y el sitio se lee igual.
- `prefers-reduced-motion: reduce` desactiva todo el movimiento. No negociable.
- **Regla de oro:** ninguna animación esconde texto del DOM. Prohibido
  `display: none` y `content-visibility: hidden` sobre contenido. Solo `opacity`,
  `transform` y `color`.
- Contraste AA mínimo en todo texto sobre `--papel`.
- La bifurcación de la §4 es una cuadrícula de dos columnas; en móvil se apila con
  rótulo explícito por camino. La comparación no puede depender de la disposición lateral.
- Semántica real: un `h1`, `h2` por sección, `section` con `aria-labelledby`.

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
│  │  ├─ narrativa/          # 01Apertura.astro … 10Honestidad.astro
│  │  ├─ figuras/            # Bifurcacion, LineaProceso, Decodificador, ComparadorColor (SVG inline)
│  │  └─ ui/                 # Acuarela.astro, Dato.astro, Cita.astro, Nota.astro
│  ├─ content/
│  │  ├─ narrativa.ts        # copy de las 10 pantallas, separado del markup
│  │  ├─ fuentes.ts          # bibliografía: id, cita, url, fecha de consulta
│  │  └─ respuestas/*.md
│  ├─ layouts/
│  │  ├─ Narrativa.astro
│  │  └─ Respuesta.astro
│  ├─ pages/
│  │  ├─ index.astro
│  │  ├─ aceitunas-negras-oxidadas.astro
│  │  ├─ e-579-gluconato-ferroso.astro
│  │  ├─ aceitunas-negras-naturales.astro
│  │  └─ aceitunas-verdes-y-negras.astro
│  └─ styles/
│     ├─ tokens.css  base.css  narrativa.css
└─ docs/superpowers/specs/
```

**Decisiones de fondo:**

- **Copy separado del markup** (`content/`): revisión editorial sin tocar layout, y
  puerta abierta al inglés.
- **`fuentes.ts` como fuente única de verdad.** Cada afirmación del copy referencia
  un `id`. La §10 y las páginas de respuesta renderizan la lista desde ahí, así es
  imposible que una cita quede huérfana. **Con la revisión 2 esto pasa de bueno a
  imprescindible**: el sitio se sostiene sobre citas verificables, no sobre tono.
- **`<Picture>` de `astro:assets`**: AVIF + WebP, `srcset`, `width`/`height`
  explícitos, `loading="lazy"` salvo la pantalla 1.
- **CSS crítico inline** en la narrativa; el resto diferido.
- **Un componente por pantalla.** Diez ficheros pequeños, no un `index.astro` de 900 líneas.

**Despliegue:** Cloudflare Pages. Caché inmutable para fuentes e imágenes con hash.

---

## 8. Presupuesto de rendimiento

Límites, no aspiraciones. Si una decisión de diseño rompe uno, se revisa la
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

**Expectativa realista:** para `aceitunas negras` a secas no se va a rankear — ese
resultado es supermercado y receta, intención de compra. El terreno ganable es el
racimo de preguntas de §4.2, donde la competencia son artículos que repiten el
bulo de las «teñidas» y el sitio tiene una ventaja real: **es correcto**.

- Prerenderizado completo. Nada de contenido dependiente de JS.
- `<title>` y `<meta description>` propios, escritos a mano, en las cinco páginas.
- JSON-LD: `Article` en la narrativa, `FAQPage` en las respuestas.
- `og:image` **diseñada** — acuarela compuesta con el titular en Fraunces, no un recorte.
- `sitemap.xml` (`@astrojs/sitemap`), `robots.txt`, `canonical`, `lang="es"`.
- Enlazado interno: respuestas → narrativa; narrativa → respuestas solo desde §10 y pie.
- Los enlaces entrantes los trae la compartición. El diseño de la narrativa **es** la
  estrategia de enlaces.

---

## 10. Hechos verificados

Verificación realizada el 2026-09-08. Cada fila es citable. Las que no lo son
están marcadas y **no se escriben en copy**.

### 10.1 Confirmado con fuente primaria

| Hecho | Fuente |
|---|---|
| **«Negras: son las obtenidas de frutos que no estando totalmente maduros, han sido oscurecidos mediante oxidación.»** | RD 679/2016, art. 4.4 *(verbatim)* |
| **«Negras naturales: son las obtenidas de frutos recogidos en plena madurez o poco antes de ella, pudiendo presentar […] color negro rojizo, negro violáceo, violeta, negro verdoso o castaño oscuro.»** | RD 679/2016, art. 4.3 *(verbatim)* |
| **«Oxidación: es el proceso por el cual las aceitunas de los tipos verde y de color cambiante, que en una fase previa se conservan en salmuera, fermentadas o no, son ennegrecidas por oxidación en medio alcalino.»** | RD 679/2016, art. 5.4 *(verbatim)* |
| La denominación debe indicar **«el color de la aceituna según el artículo 4. Esta mención no será obligatoria en los envases transparentes.»** | RD 679/2016, art. 12.2.a.2.º *(verbatim)* |
| **El proceso de elaboración es mención VOLUNTARIA:** «Voluntariamente se podrán incluir las siguientes menciones: […] b) El proceso de elaboración al que han sido sometidas conforme al artículo 5.» | RD 679/2016, art. 12.3.b *(verbatim)* — **la columna vertebral del sitio** |
| El RD 679/2016 derogó el RD 1230/2001 | RD 679/2016, disp. derogatoria única.b |
| E-579 (gluconato ferroso) y E-585 (lactato ferroso) autorizados para aceitunas ennegrecidas por oxidación, máx. **150 mg/kg expresado en Fe** | Reg. (CE) 1333/2008, anexo II (vía Reg. (UE) 1129/2011) |
| Los aditivos deben declararse en la lista de ingredientes | Reg. (UE) 1169/2011 |

### 10.2 Confirmado con fuente experta

| Hecho | Fuente |
|---|---|
| **No se añade colorante.** El negro son pigmentos de tipo melanina formados al oxidarse los compuestos fenólicos **de la propia aceituna** en medio alcalino | Javier Sánchez Perona (CSIC, Instituto de la Grasa) y Marta Berlanga Del Pozo (Univ. Pablo de Olavide), *The Conversation*, 2025-03-06 |
| **La sal de hierro no colorea: uniformiza.** Forma complejos con los fenoles propios de la aceituna para dar un negro homogéneo e intenso | ídem |
| **Sin la sal de hierro el resultado es «marrón muy oscuro».** El paso es *opcional* y se adopta ampliamente **por atractivo comercial** | ídem — **el dato más elocuente del sitio** |
| Diferencia organoléptica: negras naturales **ácidas, saladas y amargas**; oxidadas **neutro-alcalinas**, poco saladas, sin amargor | ídem |
| Proceso: NaOH diluido (reduce amargor y ablanda) + inyección de aire para oxidar; esterilización térmica final | ídem + Revista Alimentaria |
| Acrilamida en negras oxidadas: **31,5–744,0 ng/g** en el fruto y **59,2–1697 ng/g** en el líquido. Se genera **en la esterilización térmica**, no en la oxidación. Estilo californiano presenta los niveles más altos; **no se detecta en estilo español**. EFSA las incluye entre los alimentos a vigilar | Daniel Martín Vertedor (CICYTEX), *Revista Alimentaria* |
| Reparto varietal español: Hojiblanca ~46 %, Manzanilla ~36 %, Gordal ~7 %, Manzanilla Cacereña ~4 %. La Cacereña se destina comúnmente a negra oxidada estilo californiano | MAPA / Oleo Revista |
| Negra de Aragón (variedad **Empeltre**): negra natural, curada en seco, arrugada, sin proceso químico | fuentes sectoriales aragonesas |

### 10.3 NO verificado — prohibido escribirlo

| Afirmación | Situación |
|---|---|
| **Pérdida de polifenoles de las oxidadas frente a las naturales** | **No se ha encontrado cifra comparativa sólida.** Los autores del CSIC afirman que **ambos tipos** contienen compuestos fenólicos con actividad antioxidante y que la preocupación nutricional principal es **el sodio**, no el proceso. **Se retira del copy** hasta tener un estudio comparativo con cifras. La revisión 1 lo daba por bueno; era una suposición. |
| **Tiempos concretos del proceso** (las «18 h» de la maqueta tipográfica) | **Marcador inventado, no dato.** No se ha localizado fuente con números de baños, concentraciones ni duraciones. Sin cita, no se publica ninguna cifra: el copy hablará de «tratamientos sucesivos» y «horas», no de números. |
| Que «la mayoría» de las negras de lata sean oxidadas | Plausible y repetido por prensa, pero sin dato de cuota de mercado verificado. Se dirá «la mayor parte de las que se venden en lata» solo si se encuentra la cifra; si no, se reformula sin cuantificar. |

### 10.4 Refutado — el sitio dirá lo contrario

| Afirmación popular | Realidad |
|---|---|
| «Las aceitunas negras van **teñidas**» | Falso. No hay colorante. El pigmento es de la propia aceituna. |
| «El color lo pone una fábrica» | Impreciso. La fábrica **acelera y fija** un pigmento que la aceituna genera. |
| «Es un fraude de etiquetado» | Falso. La etiqueta cumple la norma. El problema es que **la norma no obliga a declarar el proceso** (art. 12.3.b). |
| «Son peligrosas» | Sin respaldo, y fuera del marco editorial. |

### 10.5 El contraargumento, citado y enlazado

La §10 de la narrativa **cita y enlaza** el artículo de Sánchez Perona y Berlanga
Del Pozo, «Todas las aceitunas negras de mesa son de verdad», y le da la razón en
lo que la tiene: la química, la legalidad y la ausencia de riesgo.

Y explica dónde el sitio sigue discrepando: en que **el consumidor que compra
«aceitunas negras» cree estar comprando fruto madurado en el árbol**, y la norma
permite no aclarárselo. Eso no es un problema de química. Es un problema de
vocabulario, y por tanto es legítimo hablar de él.

Steel-mannear al experto no debilita el sitio: es lo único que lo hace
inatacable.

---

## 11. Riesgos

| Riesgo | Mitigación |
|---|---|
| Repetir el bulo de las «teñidas» por inercia al redactar | «Teñidas», «colorante» y «fraude» son **palabras prohibidas** (§3). Revisión de copy contra esa lista antes de publicar |
| Que un experto desmonte el sitio | Ya está desmontado el bulo dentro del propio sitio (§4.1 pantalla 2 y §10.5), con el contraargumento citado y enlazado |
| La ilustración IA se usa contra el argumento | Crédito explícito en §10; la acuarela nunca ocupa el lugar de la prueba |
| Colar cifras sin fuente (polifenoles, tiempos) | §10.3 es una lista de prohibiciones, y `fuentes.ts` obliga a que toda afirmación tenga `id` |
| Deriva al alarmismo con la acrilamida | Va en §10, con su contexto, y con la frase de que se genera en la esterilización y no se detecta en estilo español. Nunca en titular |
| `animation-timeline` sin soporte | `@supports` con estados finales estáticos |
| Contenido flaco para Google | Mínimo 1.400 palabras en la narrativa + cuatro páginas de respuesta |
| Alcance creciente hacia comparador o marcas | Está en «no objetivos». Si vuelve, es un proyecto nuevo |

---

## 12. Decisiones abiertas

1. **Requisito pendiente del usuario.** Queda un requisito mencionado pero no
   formulado. Se incorpora como enmienda antes de escribir el plan de
   implementación si llega a tiempo.

Todo lo demás está cerrado: nombre, estructura, stack, paleta, tipografía,
tratamiento de imagen, marco editorial, presupuesto y SEO.

---

## 13. Fuentes

- **Real Decreto 679/2016**, de 16 de diciembre, norma de calidad de las aceitunas
  de mesa. Texto consolidado.
  https://www.boe.es/buscar/act.php?id=BOE-A-2016-11953
- **Reglamento (UE) n.º 1129/2011**, que modifica el anexo II del Reglamento (CE)
  n.º 1333/2008 (lista de aditivos de la Unión).
  https://www.boe.es/doue/2011/295/L00001-00177.pdf
- **Reglamento (UE) n.º 1169/2011**, información alimentaria facilitada al consumidor.
- **Codex Alimentarius CXS 66**, norma para las aceitunas de mesa.
  https://www.fao.org/input/download/standards/243/CXS_066s.pdf
- Sánchez Perona, J. y Berlanga Del Pozo, M. (2025). **«Todas las aceitunas negras
  de mesa son de verdad»**. *The Conversation* España, 6 de marzo.
  https://theconversation.com/todas-las-aceitunas-negras-de-mesa-son-de-verdad-249617
- Martín Vertedor, D. **«¿Las aceitunas negras oxidadas al estilo californiano
  contienen acrilamida?»**. *Revista Alimentaria* (CICYTEX).
  https://revistaalimentaria.es/agricultura/materias-primas/las-aceitunas-negras-oxidadas-al-estilo-californiano-contienen-acrilamida
- **MAPA**, Aceituna de mesa (datos de variedades y producción).
  https://www.mapa.gob.es/es/agricultura/temas/producciones-agricolas/aceite-oliva-y-aceituna-mesa/aceituna

**Pendiente de localizar:** estudio comparativo de polifenoles entre negras
oxidadas y negras naturales con cifras; y fuente técnica con tiempos y
concentraciones del proceso de oxidación.
