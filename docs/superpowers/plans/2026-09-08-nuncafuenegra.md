# nuncafuenegra.com Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Construir un sitio estático en español de una sola página narrativa (más cuatro páginas de respuesta para SEO) que explique por qué las «aceitunas negras» de lata son fruto sin madurar ennegrecido por oxidación, y no «aceitunas negras naturales».

**Architecture:** Astro 5 prerenderizado, cero JavaScript enviado al cliente. Todo el movimiento con animaciones CSS ligadas al scroll (`animation-timeline`). El copy vive en módulos TypeScript separados del markup, y cada afirmación referencia por `id` una entrada de una bibliografía tipada. Los límites del spec (presupuesto de peso, paleta, vocabulario prohibido, citas obligatorias) se verifican con tests automáticos sobre `dist/`, de forma que romper el spec rompe el build.

**Tech Stack:** Astro 5 · TypeScript · CSS vanilla con `@layer` y custom properties (sin Tailwind) · `astro:assets` (sharp) para AVIF/WebP · `@astrojs/sitemap` · Vitest para asserts sobre `dist/` · Playwright + `@axe-core/playwright` para accesibilidad · Cloudflare Pages para el despliegue.

**Spec:** `docs/superpowers/specs/2026-09-08-nuncafuenegra-design.md` (revisión 2)

## Global Constraints

Todas las tareas heredan implícitamente esta sección. Los valores están copiados verbatim del spec.

**Presupuesto de rendimiento (spec §8) — son límites, no aspiraciones:**
- JavaScript enviado: **0 KB**
- HTML + CSS crítico + fuentes: **< 100 KB**
- Página completa, con acuarelas: **< 500 KB**
- LCP (4G simulada): **< 1,2 s**
- CLS: **0**
- Peticiones en el primer render: **≤ 6**

**Paleta (spec §5.2) — `color-scheme: light only`, sin modo oscuro:**
```
--papel      #EDE4D4
--tinta      #2B2318
--tinta-70   #5C4F3C
--tinta-45   #8A7B63
--linea      #C4B393
--oliva      #6E7444
--ocre       #B8792F
--terracota  #9C5232
--granate    #6B2B3E
--negro      #000000   SOLO camino fábrica. Prohibido en el resto.
```
Prohibidos los grises y los `rgba()` de negro para texto: viran a gris sucio sobre el papel. Los secundarios usan `--tinta-70` y `--tinta-45`.

**Tipografía (spec §5.3):** Fraunces (display, `opsz` 144 / `SOFT` 45 / `WONK` 1), Newsreader (cuerpo, 20px, medida 62–64 caracteres), IBM Plex Mono 400/500 (aparato, datos de proceso, lista de ingredientes). Todas variables, **autoalojadas** y subseteadas a `latin` + `latin-ext`. Prohibido cargar fuentes de un CDN externo.

**Vocabulario prohibido en todo el copy (spec §3 y §10.4):** `teñidas`, `teñir`, `colorante`, `fraude`. No las tiñen: el pigmento es de la propia aceituna.

**Afirmaciones prohibidas (spec §10.3):** pérdida de polifenoles frente a las naturales; cualquier cifra de tiempos o concentraciones del proceso; cuantificar qué proporción del mercado es oxidada.

**Regla de oro de accesibilidad (spec §6):** ninguna animación puede esconder texto del DOM. Prohibidos `display: none` y `content-visibility: hidden` sobre contenido. Solo `opacity`, `transform` y `color`. `prefers-reduced-motion: reduce` desactiva todo el movimiento.

**Documentación IA-first (spec §13):** el repo debe poder retomarse con otra IA
sin contexto previo. `AGENTS.md` es el punto de entrada; `docs/NO-TOCAR.md` lista
lo que un agente intentará «arreglar» y no debe. Todo en español. Los ficheros
`src/content/fuentes.ts`, `src/content/narrativa.ts`, `src/styles/tokens.css` y
`src/components/ui/Acuarela.astro` empiezan con un comentario de cabecera que dice
qué regla protegen y qué test salta al romperla. Los tests son la documentación
ejecutable: una regla escrita sin test es una sugerencia.

**Regla editorial (spec §3):** se señala la práctica y el vacío de la norma, **nunca las marcas**. La etiqueta de ejemplo es genérica y compuesta.

## Prerequisitos externos

Las **10 acuarelas** (spec §5.4) las genera el usuario con IA; no son parte de este plan. La Tarea 6 crea `scripts/acuarelas-provisionales.mjs`, que genera sustitutas de color plano con las dimensiones y nombres definitivos, de modo que el build y los tests pasan antes de que existan las definitivas. Sustituir los ficheros de `src/assets/acuarelas/` no requiere ningún cambio de código.

Nombres definitivos, en este orden:
`01-aceituna-negra.jpg` · `02-rama-verde.jpg` · `03-rama-envero.jpg` · `04-rama-madura.jpg` · `05-olivar-diciembre.jpg` · `06-manos-cesta.jpg` · `07-salmuera-tarro.jpg` · `08-sal-hierbas.jpg` · `09-nave-industrial.jpg` · `10-lata-mesa.jpg`

---

### Task 1: Andamiaje Astro y guardián del presupuesto

**Files:**
- Create: `package.json`
- Create: `astro.config.mjs`
- Create: `tsconfig.json`
- Create: `.gitignore`
- Create: `src/pages/index.astro`
- Create: `tests/helpers/dist.ts`
- Create: `tests/presupuesto.test.ts`
- Create: `vitest.config.ts`

**Interfaces:**
- Consumes: nada.
- Produces: `tests/helpers/dist.ts` exporta `DIST` (ruta absoluta a `dist/`), `leerDist(rel: string): string`, `listarDist(ext: string): string[]`, `pesoDist(rel: string): number` y `htmlDeDist(): {ruta: string, html: string}[]`. Todas las tareas posteriores usan estos helpers en sus tests.

- [ ] **Step 1: Write the failing test**

`tests/helpers/dist.ts`:
```ts
import { readFileSync, statSync, readdirSync } from 'node:fs'
import { join, relative, sep } from 'node:path'

export const DIST = join(process.cwd(), 'dist')

function caminar(dir: string): string[] {
  return readdirSync(dir, { withFileTypes: true }).flatMap((e) =>
    e.isDirectory() ? caminar(join(dir, e.name)) : [join(dir, e.name)],
  )
}

/** Rutas relativas a dist/ con la extensión dada, en POSIX. */
export function listarDist(ext: string): string[] {
  return caminar(DIST)
    .filter((f) => f.endsWith(ext))
    .map((f) => relative(DIST, f).split(sep).join('/'))
}

export function leerDist(rel: string): string {
  return readFileSync(join(DIST, rel), 'utf8')
}

/** Peso en bytes de un fichero de dist/. */
export function pesoDist(rel: string): number {
  return statSync(join(DIST, rel)).size
}

export function htmlDeDist(): { ruta: string; html: string }[] {
  return listarDist('.html').map((ruta) => ({ ruta, html: leerDist(ruta) }))
}
```

`tests/presupuesto.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { listarDist, htmlDeDist } from './helpers/dist'

describe('presupuesto de rendimiento (spec §8)', () => {
  it('no envía ni un byte de JavaScript', () => {
    expect(listarDist('.js')).toEqual([])
    expect(listarDist('.mjs')).toEqual([])
  })

  it('ningún HTML contiene etiquetas <script>', () => {
    for (const { ruta, html } of htmlDeDist()) {
      expect(html, `${ruta} contiene <script>`).not.toMatch(/<script/i)
    }
  })

  it('el build produce al menos la página de inicio', () => {
    expect(listarDist('.html')).toContain('index.html')
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npx vitest run tests/presupuesto.test.ts`
Expected: FAIL — `ENOENT: no such file or directory, scandir '.../dist'` (aún no hay build).

- [ ] **Step 3: Write minimal implementation**

`package.json`:
```json
{
  "name": "nuncafuenegra",
  "type": "module",
  "private": true,
  "engines": { "node": ">=20.11" },
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro preview",
    "test": "astro build && vitest run",
    "test:unit": "vitest run"
  },
  "dependencies": {
    "astro": "^5.0.0",
    "@astrojs/sitemap": "^3.2.0",
    "sharp": "^0.33.0"
  },
  "devDependencies": {
    "vitest": "^2.1.0",
    "typescript": "^5.6.0"
  }
}
```

`astro.config.mjs`:
```js
import { defineConfig } from 'astro/config'
import sitemap from '@astrojs/sitemap'

export default defineConfig({
  site: 'https://nuncafuenegra.com',
  integrations: [sitemap()],
  build: { inlineStylesheets: 'always' },
  compressHTML: true,
})
```

`tsconfig.json`:
```json
{
  "extends": "astro/tsconfigs/strict",
  "include": [".astro/types.d.ts", "**/*"],
  "exclude": ["dist"]
}
```

`.gitignore`:
```
node_modules/
dist/
.astro/
.DS_Store
test-results/
playwright-report/
```

`vitest.config.ts`:
```ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: { include: ['tests/**/*.test.ts'], environment: 'node' },
})
```

`src/pages/index.astro`:
```astro
---
const titulo = 'Esta aceituna nunca estuvo negra'
---
<html lang="es">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>{titulo}</title>
  </head>
  <body>
    <h1>{titulo}</h1>
  </body>
</html>
```

- [ ] **Step 4: Run test to verify it passes**

Run: `npm install && npm test`
Expected: PASS — 3 tests.

- [ ] **Step 5: Commit**

```bash
git add package.json package-lock.json astro.config.mjs tsconfig.json .gitignore vitest.config.ts src/pages/index.astro tests/
git commit -m "feat: andamiaje astro con guardián de presupuesto de cero javascript"
```

---

### Task 1.5: Documentación IA-first (base)

Va aquí, y no al final, porque el repo tiene que ser retomable desde el segundo
commit. Necesita el Vitest de la Tarea 1.

**Files:**
- Create: `AGENTS.md`
- Create: `CLAUDE.md`
- Create: `docs/NO-TOCAR.md`
- Create: `docs/GLOSARIO.md`
- Create: `tests/documentacion.test.ts`

**Interfaces:**
- Consumes: nada.
- Produces: `AGENTS.md` como punto de entrada único. Toda tarea posterior que cree
  una regla nueva la añade a `docs/NO-TOCAR.md` si un agente pudiera deshacerla.

- [ ] **Step 1: Write the failing test**

`tests/documentacion.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { existsSync, readFileSync } from 'node:fs'
import { join } from 'node:path'

const raiz = (rel: string) => join(process.cwd(), rel)
const leer = (rel: string) => readFileSync(raiz(rel), 'utf8')

const OBLIGATORIOS = [
  'AGENTS.md',
  'CLAUDE.md',
  'docs/NO-TOCAR.md',
  'docs/GLOSARIO.md',
  'docs/superpowers/specs/2026-09-08-nuncafuenegra-design.md',
  'docs/superpowers/plans/2026-09-08-nuncafuenegra.md',
]

describe('documentación IA-first (spec §13)', () => {
  it('existen todos los documentos de entrada', () => {
    for (const f of OBLIGATORIOS) expect(existsSync(raiz(f)), `falta ${f}`).toBe(true)
  })

  it('CLAUDE.md solo apunta a AGENTS.md, sin duplicar contenido', () => {
    const c = leer('CLAUDE.md')
    expect(c).toMatch(/AGENTS\.md/)
    expect(c.length, 'CLAUDE.md duplica contenido en vez de apuntar').toBeLessThan(400)
  })

  it('toda ruta citada en AGENTS.md existe de verdad', () => {
    const rutas = [...leer('AGENTS.md').matchAll(/`([^`\n]+)`/g)]
      .map((m) => m[1])
      .filter((t) => /^[\w./-]+\.(md|ts|css|astro|json|mjs|txt)$/.test(t) || t.endsWith('/'))
    expect(rutas.length, 'AGENTS.md no cita ninguna ruta').toBeGreaterThan(3)
    for (const r of rutas)
      expect(existsSync(raiz(r.replace(/\/$/, ''))), `AGENTS.md cita ${r}, que no existe`).toBe(true)
  })

  it('AGENTS.md enuncia las reglas irrompibles', () => {
    const a = leer('AGENTS.md')
    for (const regla of [/te[ñn]idas/i, /0 KB/i, /naturales/, /#000/])
      expect(a, `AGENTS.md no menciona ${regla}`).toMatch(regla)
  })

  it('NO-TOCAR.md da un motivo por cada prohibición', () => {
    const lineas = leer('docs/NO-TOCAR.md')
      .split('\n')
      .filter((l) => l.trim().startsWith('- **'))
    expect(lineas.length, 'muy pocas prohibiciones').toBeGreaterThanOrEqual(6)
    for (const l of lineas)
      expect(l, `sin motivo: ${l.slice(0, 50)}`).toMatch(/Por qué:/i)
  })

  it('el glosario define los términos del dominio', () => {
    const g = leer('docs/GLOSARIO.md')
    for (const t of ['envero', 'aderezo', 'aliñado', 'oxidación', 'negras naturales'])
      expect(g.toLowerCase(), `falta ${t}`).toContain(t.toLowerCase())
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npx vitest run tests/documentacion.test.ts`
Expected: FAIL — «falta AGENTS.md».

- [ ] **Step 3: Write minimal implementation**

`AGENTS.md`:
```markdown
# nuncafuenegra.com

Sitio estático en español que explica por qué las «aceitunas negras» de lata son
fruto recogido sin madurar y ennegrecido por oxidación, y no «aceitunas negras
naturales».

**Si eres un agente y acabas de abrir este repo, lee esto entero antes de tocar
nada.** No es burocracia: hay cinco decisiones que parecen errores y no lo son, y
si las «arreglas» rompes el proyecto.

## Las cinco reglas irrompibles

1. **No se dice «teñidas».** Las aceitunas oxidadas NO llevan colorante. El negro
   es pigmento de la propia aceituna, que se oxida en medio alcalino. La sal de
   hierro (E-579) no colorea: uniformiza. Escribir «teñidas», «colorante» o
   «fraude» es factualmente falso y hay un test que lo impide.
2. **Cero JavaScript.** `0 KB` enviados al cliente. Nada de islas, ni de un
   `<script>` «pequeñito». El movimiento va con `animation-timeline` en CSS.
3. **`#000` solo en el camino de la fábrica.** El negro puro es el artificio del
   que habla el sitio; usarlo en cualquier otro sitio destruye el argumento
   visual. El resto de la paleta es cálida y no tiene ni un gris.
4. **La palabra clave es «naturales».** La tesis del sitio es legal, no química:
   la norma obliga a declarar el color y NO obliga a declarar el proceso
   (RD 679/2016, art. 12.3.b). La prueba para el lector es la ausencia de la
   palabra «naturales» en la denominación, no el E-579.
5. **Ninguna afirmación sin fuente.** Toda afirmación de hecho referencia un `id`
   de la bibliografía. Si no hay fuente, no se escribe. Hay afirmaciones
   explícitamente prohibidas por no estar verificadas.

## Antes de escribir una línea

| Documento | Qué contiene |
|---|---|
| `docs/NO-TOCAR.md` | Lo que vas a intentar arreglar y no debes, con su motivo. **Empieza por aquí.** |
| `docs/GLOSARIO.md` | Envero, aderezo, aliñado, oxidación. Si no sabes qué es el envero, no puedes escribir el copy. |
| `docs/superpowers/specs/2026-09-08-nuncafuenegra-design.md` | El diseño completo. §10 son los hechos verificados y §13 explica por qué existe esta documentación. El mapa de ficheros está en §7. |
| `docs/superpowers/plans/2026-09-08-nuncafuenegra.md` | El plan de implementación, tarea a tarea, con casillas de progreso. **El estado del trabajo son esas casillas.** |

## Cómo se arranca

```bash
npm install
npm run acuarelas   # sustitutas provisionales, si aún no hay ilustraciones reales
npm run dev
npm test            # build + los guardianes del spec
```

Configuración en `package.json` y `astro.config.mjs`.

## El error que vas a cometer

Vas a leer el copy, pensar que «no las tiñen» es un matiz irrelevante y
reescribirlo como «las tiñen» porque suena mejor. Ese cambio es la razón de que
exista este fichero. La versión pegadiza es falsa, la desmontaron investigadores
del CSIC, y el sitio se sostiene precisamente sobre ser el único que lo cuenta
bien. Está explicado en `docs/DECISIONES.md`.
```

`CLAUDE.md`:
```markdown
# CLAUDE.md

Las instrucciones de este repositorio están en `AGENTS.md`. Léelo antes de tocar
nada: contiene cinco reglas que parecen errores y no lo son.
```

`docs/NO-TOCAR.md`:
```markdown
# No tocar

Cada línea es algo que un agente sin contexto intentará mejorar. No lo son:
mejoras, digo. Son decisiones.

- **No añadas modo oscuro.** Por qué: el sitio tiene una identidad de papel
  cálido, con `color-scheme: light only` declarado a propósito. Un tema oscuro
  destruiría la paleta, que ES el argumento del sitio (spec §5.1).
- **No metas Tailwind, React, Vue ni ningún framework.** Por qué: el presupuesto
  es de 0 KB de JavaScript y el propietario del proyecto no quiere Tailwind. Hay
  un test que falla si aparece un solo `.js` en `dist/`.
- **No quites el `mix-blend-mode: multiply` de las acuarelas.** Por qué: no es un
  apaño. Las ilustraciones llegan sobre papel blanco y el `multiply` hace
  desaparecer ese blanco sin recurrir a PNG con alpha, que pesarían cuatro veces
  más. Además tiñe las aguadas con el arena del fondo y unifica diez piezas
  desiguales (spec §5.4).
- **No escribas «teñidas», «colorante» ni «fraude».** Por qué: es falso. No hay
  colorante, y el etiquetado cumple la norma. Investigadores del CSIC lo
  desmontaron públicamente y el valor de este sitio es ser el que lo cuenta bien.
- **No resucites la afirmación de los polifenoles.** Por qué: no existe cifra
  comparativa verificada entre oxidadas y naturales. Se buscó y no se encontró.
  Los propios autores del CSIC señalan que ambos tipos tienen compuestos
  fenólicos y que lo relevante es el sodio (spec §10.3).
- **No inventes tiempos ni temperaturas del proceso.** Por qué: no se ha
  localizado fuente fiable. El copy dice «tratamientos sucesivos», sin números,
  a propósito. Un test busca patrones de duración en la pantalla de fábrica.
- **No nombres marcas ni fotografíes envases reales.** Por qué: envejece mal en un
  sitio estático, exige verificar formulaciones que cambian, y expone a
  reclamaciones sin añadir nada al argumento. La etiqueta de ejemplo es genérica
  y compuesta (spec §3).
- **No añadas un menú de navegación a la narrativa.** Por qué: un header con
  navegación mata la pantalla de apertura. Los enlaces a las páginas de respuesta
  viven en la sección 10 y en el pie (spec §4.1).
- **No uses `display: none`, `visibility: hidden` ni `content-visibility: hidden`
  sobre contenido.** Por qué: ninguna animación puede esconder texto del DOM, ni
  para el lector de pantalla ni para el rastreador. Solo `opacity`, `transform` y
  `color` (spec §6).
- **No borres ni relajes los tests de `tests/`.** Por qué: no son control de
  calidad, son el mecanismo por el que estas decisiones sobreviven a sesiones
  futuras que no han leído el spec.
```

`docs/GLOSARIO.md`:
```markdown
# Glosario

Vocabulario del dominio. Los cuatro primeros términos son los que más se usan mal.

**Envero.** El periodo en que la aceituna cambia de color, de verde a rosado, rosa
vino o castaño, antes de la madurez completa. Suele caer entre octubre y
noviembre. Una aceituna «de color cambiante» es la recogida durante el envero.

**Aderezo.** Proceso en el que las aceitunas reciben un tratamiento alcalino para
quitarles el amargor y después se acondicionan en salmuera, donde fermentan
parcial o totalmente. No es lo mismo que la oxidación.

**Aliñado.** Añadir a la salmuera condimentos o especias, y eventualmente
vinagre: ajo, hinojo, tomillo, cáscara de naranja. Es lo que la gente llama
«aceitunas aliñadas».

**Oxidación.** Proceso por el que aceitunas verdes o de color cambiante, que antes
se conservan en salmuera, se ennegrecen por oxidación en medio alcalino. Es el
proceso que produce las «aceitunas negras» de lata.

**Negras naturales.** Categoría legal: aceitunas obtenidas de frutos recogidos en
plena madurez o poco antes. Su color puede ser negro rojizo, negro violáceo,
violeta, negro verdoso o castaño oscuro.

**Negras.** Categoría legal distinta: aceitunas obtenidas de frutos que, sin estar
totalmente maduros, han sido oscurecidos mediante oxidación. Es la que llena las
latas. La única diferencia de nombre con la anterior es la palabra «naturales».

**Estilo californiano.** Nombre habitual en la industria para la aceituna negra
oxidada. «Estilo español» se refiere a la aceituna verde aderezada y fermentada.

**Gluconato ferroso (E-579) / lactato ferroso (E-585).** Sales de hierro
autorizadas como estabilizantes del color, solo en aceitunas ennegrecidas por
oxidación, con un máximo de 150 mg/kg expresado en hierro. **No son colorantes:**
forman complejos con los compuestos fenólicos de la propia aceituna y convierten
un marrón muy oscuro en un negro uniforme.

**Compuestos fenólicos.** Los responsables del amargor de la aceituna y, al
oxidarse, del color oscuro. El pigmento negro sale de aquí, no de un aditivo.

**Salmuera.** Disolución de sal en agua donde la aceituna se conserva y fermenta.

**Empeltre.** Variedad aragonesa, la «negra de Aragón». Se cura en seco hasta
arrugarse. Es negra natural, sin proceso químico.
```

- [ ] **Step 4: Run test to verify it passes**

Run: `npm test`
Expected: PASS — 9 tests (3 de presupuesto + 6 de documentación).

- [ ] **Step 5: Commit**

```bash
git add AGENTS.md CLAUDE.md docs/NO-TOCAR.md docs/GLOSARIO.md tests/documentacion.test.ts
git commit -m "docs: punto de entrada para agentes con reglas irrompibles y glosario"
```

---

### Task 2: Tokens de diseño y guardián de la paleta

**Files:**
- Create: `src/styles/tokens.css`
- Create: `src/styles/base.css`
- Create: `tests/paleta.test.ts`
- Modify: `src/pages/index.astro`

**Interfaces:**
- Consumes: `tests/helpers/dist.ts` de la Tarea 1.
- Produces: las custom properties de `tokens.css` (`--papel`, `--tinta`, `--tinta-70`, `--tinta-45`, `--linea`, `--oliva`, `--ocre`, `--terracota`, `--granate`, `--negro`) y las utilidades `.mono` y `.medida` de `base.css`. Todas las pantallas posteriores usan estos nombres y no declaran colores literales.

- [ ] **Step 1: Write the failing test**

`tests/paleta.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { readFileSync, readdirSync } from 'node:fs'
import { join } from 'node:path'

const ESTILOS = join(process.cwd(), 'src/styles')
const css = () =>
  readdirSync(ESTILOS)
    .filter((f) => f.endsWith('.css'))
    .map((f) => ({ f, texto: readFileSync(join(ESTILOS, f), 'utf8') }))

const PALETA = [
  '#EDE4D4', '#2B2318', '#5C4F3C', '#8A7B63',
  '#C4B393', '#6E7444', '#B8792F', '#9C5232', '#6B2B3E',
]

describe('paleta (spec §5.2)', () => {
  it('tokens.css declara los diez tokens con sus valores exactos', () => {
    const t = readFileSync(join(ESTILOS, 'tokens.css'), 'utf8')
    for (const hex of PALETA) expect(t).toContain(hex)
    expect(t).toContain('--negro: #000000')
  })

  it('declara color-scheme light only: no hay modo oscuro', () => {
    const t = readFileSync(join(ESTILOS, 'tokens.css'), 'utf8')
    expect(t).toMatch(/color-scheme:\s*light only/)
    for (const { f, texto } of css())
      expect(texto, `${f} tiene un bloque prefers-color-scheme`).not.toMatch(
        /prefers-color-scheme/,
      )
  })

  it('el negro puro solo aparece al declarar --negro', () => {
    for (const { f, texto } of css()) {
      const lineas = texto
        .split('\n')
        .filter((l) => /#000\b|#000000\b|:\s*black\b/i.test(l))
        .filter((l) => !l.includes('--negro: #000000'))
      expect(lineas, `${f} usa negro puro fuera del token`).toEqual([])
    }
  })

  it('no usa rgba de negro ni grises para texto', () => {
    for (const { f, texto } of css()) {
      expect(texto, `${f} usa rgba de negro`).not.toMatch(/rgba\(\s*0\s*,\s*0\s*,\s*0/)
      expect(texto, `${f} usa un gris`).not.toMatch(/#(?:[89ab])\1\1\b/i)
    }
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npx vitest run tests/paleta.test.ts`
Expected: FAIL — `ENOENT ... src/styles/tokens.css`.

- [ ] **Step 3: Write minimal implementation**

`src/styles/tokens.css`:
```css
@layer tokens {
  :root {
    color-scheme: light only;

    --papel: #EDE4D4;
    --tinta: #2B2318;
    --tinta-70: #5C4F3C;
    --tinta-45: #8A7B63;
    --linea: #C4B393;
    --oliva: #6E7444;
    --ocre: #B8792F;
    --terracota: #9C5232;
    --granate: #6B2B3E;
    /* Solo camino fábrica. Prohibido en el resto del sitio. */
    --negro: #000000;

    --medida: 63ch;
    --paso: clamp(1.5rem, 4vw, 3rem);
  }
}
```

`src/styles/base.css`:
```css
@layer base {
  *, *::before, *::after { box-sizing: border-box; }

  html { -webkit-text-size-adjust: 100%; }

  body {
    margin: 0;
    background: var(--papel);
    color: var(--tinta);
    font-family: 'Newsreader', Georgia, serif;
    font-size: 20px;
    line-height: 1.6;
  }

  .mono {
    font-family: 'IBM Plex Mono', ui-monospace, monospace;
    font-size: 0.68rem;
    letter-spacing: 0.22em;
    text-transform: uppercase;
    color: var(--oliva);
  }

  .medida { max-width: var(--medida); }

  img { max-width: 100%; height: auto; }
}
```

`src/pages/index.astro` — reemplazar el contenido por:
```astro
---
import '../styles/tokens.css'
import '../styles/base.css'
const titulo = 'Esta aceituna nunca estuvo negra'
---
<html lang="es">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>{titulo}</title>
  </head>
  <body>
    <h1>{titulo}</h1>
  </body>
</html>
```

- [ ] **Step 4: Run test to verify it passes**

Run: `npm test`
Expected: PASS — 7 tests.

- [ ] **Step 5: Commit**

```bash
git add src/styles/ src/pages/index.astro tests/paleta.test.ts
git commit -m "feat: tokens de la paleta cálida con guardián del negro puro"
```

---

### Task 3: Fuentes autoalojadas sin salto de maquetación

**Files:**
- Create: `public/fonts/` (los `.woff2` subseteados)
- Create: `src/styles/tipografia.css`
- Create: `tests/tipografia.test.ts`
- Modify: `src/pages/index.astro`

**Interfaces:**
- Consumes: los tokens de la Tarea 2.
- Produces: las familias `'Fraunces'`, `'Newsreader'` y `'IBM Plex Mono'` disponibles por CSS, más las clases `.display`, `.cuerpo` y `.aparato`. Las pantallas posteriores usan estas clases.

- [ ] **Step 1: Write the failing test**

`tests/tipografia.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { readFileSync, existsSync } from 'node:fs'
import { join } from 'node:path'
import { htmlDeDist, listarDist, pesoDist } from './helpers/dist'

const tipografia = () =>
  readFileSync(join(process.cwd(), 'src/styles/tipografia.css'), 'utf8')

const FUENTES = [
  'fraunces-latin-ext.woff2',
  'newsreader-latin-ext.woff2',
  'plex-mono-latin-ext.woff2',
]

describe('tipografía (spec §5.3)', () => {
  it('las tres fuentes están autoalojadas en public/fonts', () => {
    for (const f of FUENTES)
      expect(existsSync(join(process.cwd(), 'public/fonts', f)), f).toBe(true)
  })

  it('no carga fuentes de ningún CDN externo', () => {
    expect(tipografia()).not.toMatch(/https?:\/\//)
    for (const { ruta, html } of htmlDeDist())
      expect(html, `${ruta} enlaza fuentes externas`).not.toMatch(
        /fonts\.googleapis|fonts\.gstatic|use\.typekit/,
      )
  })

  it('cada @font-face usa swap y declara métricas de fallback', () => {
    const bloques = tipografia().match(/@font-face\s*\{[^}]*\}/g) ?? []
    expect(bloques.length).toBeGreaterThanOrEqual(3)
    for (const b of bloques) {
      expect(b, 'falta font-display: swap').toMatch(/font-display:\s*swap/)
      expect(b, 'falta size-adjust').toMatch(/size-adjust:/)
    }
  })

  it('la portada precarga las dos fuentes críticas', () => {
    const html = htmlDeDist().find((h) => h.ruta === 'index.html')!.html
    expect(html).toMatch(/rel="preload"[^>]*fraunces[^>]*as="font"/i)
    expect(html).toMatch(/rel="preload"[^>]*newsreader[^>]*as="font"/i)
  })

  it('las fuentes suman menos de 80 KB', () => {
    const total = listarDist('.woff2').reduce((n, f) => n + pesoDist(f), 0)
    expect(total).toBeLessThan(80 * 1024)
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npx vitest run tests/tipografia.test.ts`
Expected: FAIL — `ENOENT ... src/styles/tipografia.css`.

- [ ] **Step 3: Write minimal implementation**

Descargar las fuentes variables y subsetearlas. Instalar la herramienta una sola vez:

```bash
pip install "fonttools[woff]" brotli
mkdir -p public/fonts vendor-fonts
# Descargar los .ttf variables desde los repositorios oficiales de Google Fonts:
curl -sL -o vendor-fonts/Fraunces.ttf   "https://raw.githubusercontent.com/google/fonts/main/ofl/fraunces/Fraunces%5BSOFT%2CWONK%2Copsz%2Cwght%5D.ttf"
curl -sL -o vendor-fonts/Newsreader.ttf "https://raw.githubusercontent.com/google/fonts/main/ofl/newsreader/Newsreader%5Bopsz%2Cwght%5D.ttf"
curl -sL -o vendor-fonts/PlexMono.ttf   "https://raw.githubusercontent.com/google/fonts/main/ofl/ibmplexmono/IBMPlexMono-Regular.ttf"
```

Subsetear a `latin` + `latin-ext` conservando los ejes variables. El rango incluye `ñ`, `¿`, `¡` y las vocales acentuadas:

```bash
RANGO="U+0020-007E,U+00A0-00FF,U+0100-017F,U+02BB-02BC,U+2010-2027,U+20AC"
for par in "Fraunces:fraunces" "Newsreader:newsreader" "PlexMono:plex-mono"; do
  origen="${par%%:*}"; destino="${par##*:}"
  pyftsubset "vendor-fonts/${origen}.ttf" \
    --unicodes="$RANGO" \
    --layout-features='kern,liga,onum,tnum' \
    --flavor=woff2 \
    --output-file="public/fonts/${destino}-latin-ext.woff2"
done
ls -la public/fonts/
```

`src/styles/tipografia.css`. Los valores de `size-adjust`, `ascent-override` y
`descent-override` alinean la métrica del fallback con la de la fuente real para
que el intercambio no mueva nada (CLS 0):

```css
@layer tipografia {
  @font-face {
    font-family: 'Fraunces';
    src: url('/fonts/fraunces-latin-ext.woff2') format('woff2-variations');
    font-weight: 100 900;
    font-style: normal;
    font-display: swap;
    size-adjust: 100%;
    ascent-override: 96%;
    descent-override: 24%;
    unicode-range: U+0020-007E, U+00A0-00FF, U+0100-017F, U+2010-2027, U+20AC;
  }

  @font-face {
    font-family: 'Newsreader';
    src: url('/fonts/newsreader-latin-ext.woff2') format('woff2-variations');
    font-weight: 200 800;
    font-style: normal;
    font-display: swap;
    size-adjust: 100%;
    ascent-override: 94%;
    descent-override: 26%;
    unicode-range: U+0020-007E, U+00A0-00FF, U+0100-017F, U+2010-2027, U+20AC;
  }

  @font-face {
    font-family: 'IBM Plex Mono';
    src: url('/fonts/plex-mono-latin-ext.woff2') format('woff2');
    font-weight: 400 500;
    font-style: normal;
    font-display: swap;
    size-adjust: 100%;
    ascent-override: 92%;
    descent-override: 28%;
    unicode-range: U+0020-007E, U+00A0-00FF, U+0100-017F, U+2010-2027, U+20AC;
  }

  .display {
    font-family: 'Fraunces', Georgia, serif;
    font-weight: 400;
    font-variation-settings: 'opsz' 144, 'SOFT' 45, 'WONK' 1;
    font-size: clamp(2.4rem, 7.8vw, 5.8rem);
    line-height: 1;
    letter-spacing: -0.03em;
    margin: 0;
  }

  .cuerpo {
    font-family: 'Newsreader', Georgia, serif;
    font-variation-settings: 'opsz' 20;
    font-size: 1.24rem;
    line-height: 1.66;
  }

  .aparato {
    font-family: 'IBM Plex Mono', ui-monospace, monospace;
    font-size: 0.86rem;
    line-height: 1.75;
  }
}
```

Añadir a `src/pages/index.astro`, en el `<head>` y antes del `<title>`:
```astro
    <link rel="preload" href="/fonts/fraunces-latin-ext.woff2" as="font" type="font/woff2" crossorigin />
    <link rel="preload" href="/fonts/newsreader-latin-ext.woff2" as="font" type="font/woff2" crossorigin />
```
y en el frontmatter, tras los otros imports:
```astro
import '../styles/tipografia.css'
```

- [ ] **Step 4: Run test to verify it passes**

Run: `npm test`
Expected: PASS — 12 tests. Si el test de peso falla, reducir `--layout-features` a `'kern,liga'`.

- [ ] **Step 5: Commit**

```bash
git add public/fonts src/styles/tipografia.css src/pages/index.astro tests/tipografia.test.ts
git commit -m "feat: fuentes autoalojadas subseteadas con métricas de fallback"
```

---

### Task 4: Bibliografía tipada y guardianes del copy

Esta es la tarea que hace que el spec sea ejecutable. Convierte las prohibiciones
de §10.3 y §3 en tests, y hace imposible que una afirmación quede sin cita.

**Files:**
- Create: `src/content/fuentes.ts`
- Create: `src/content/narrativa.ts`
- Create: `tests/copy.test.ts`

**Interfaces:**
- Consumes: nada.
- Produces:
  - `src/content/fuentes.ts` exporta `type FuenteId` y `const FUENTES: Record<FuenteId, Fuente>` con `Fuente = { cita: string; url: string; consultada: string }`.
  - `src/content/narrativa.ts` exporta `type Pantalla = { n: number; id: string; titulo: string; entradilla?: string; prosa: string[]; fuentes?: FuenteId[] }` y `const PANTALLAS: Pantalla[]` con las diez pantallas. Las Tareas 7 a 12 importan `PANTALLAS` y seleccionan por `n`.

- [ ] **Step 1: Write the failing test**

`tests/copy.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { FUENTES } from '../src/content/fuentes'
import { PANTALLAS } from '../src/content/narrativa'

const todoElTexto = () =>
  PANTALLAS.flatMap((p) => [p.titulo, p.entradilla ?? '', ...p.prosa]).join('\n')

describe('vocabulario prohibido (spec §3, §10.4)', () => {
  it('no dice teñidas, teñir, colorante ni fraude', () => {
    for (const palabra of [/teñid/i, /teñir/i, /colorante/i, /fraude/i])
      expect(todoElTexto(), `usa ${palabra}`).not.toMatch(palabra)
  })

  it('no afirma pérdida de polifenoles', () => {
    expect(todoElTexto()).not.toMatch(/polifenol/i)
  })
})

describe('bibliografía (spec §7, §10)', () => {
  it('toda fuente referenciada existe', () => {
    for (const p of PANTALLAS)
      for (const id of p.fuentes ?? [])
        expect(FUENTES[id], `pantalla ${p.n} cita "${id}", que no existe`).toBeDefined()
  })

  it('toda fuente tiene cita, url y fecha de consulta', () => {
    for (const [id, f] of Object.entries(FUENTES)) {
      expect(f.cita.length, id).toBeGreaterThan(10)
      expect(f.url, id).toMatch(/^https:\/\//)
      expect(f.consultada, id).toMatch(/^\d{4}-\d{2}-\d{2}$/)
    }
  })

  it('las pantallas 5, 7 y 10 van citadas: son las que afirman hechos', () => {
    for (const n of [5, 7, 10]) {
      const p = PANTALLAS.find((x) => x.n === n)!
      expect(p.fuentes?.length, `pantalla ${n} sin fuentes`).toBeGreaterThan(0)
    }
  })
})

describe('estructura y volumen (spec §4.1)', () => {
  it('tiene diez pantallas numeradas del 1 al 10', () => {
    expect(PANTALLAS.map((p) => p.n)).toEqual([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
  })

  it('supera las 1.400 palabras reales', () => {
    const palabras = todoElTexto().trim().split(/\s+/).length
    expect(palabras).toBeGreaterThanOrEqual(1400)
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npx vitest run tests/copy.test.ts`
Expected: FAIL — no resuelve `../src/content/fuentes`.

- [ ] **Step 3: Write minimal implementation**

`src/content/fuentes.ts`:
```ts
export type Fuente = { cita: string; url: string; consultada: string }

export const FUENTES = {
  'rd-679-2016': {
    cita: 'Real Decreto 679/2016, de 16 de diciembre, por el que se establece la norma de calidad de las aceitunas de mesa. Texto consolidado.',
    url: 'https://www.boe.es/buscar/act.php?id=BOE-A-2016-11953',
    consultada: '2026-09-08',
  },
  'aditivos-1129-2011': {
    cita: 'Reglamento (UE) n.º 1129/2011, que modifica el anexo II del Reglamento (CE) n.º 1333/2008 para establecer la lista de aditivos alimentarios de la Unión.',
    url: 'https://www.boe.es/doue/2011/295/L00001-00177.pdf',
    consultada: '2026-09-08',
  },
  'codex-66': {
    cita: 'Codex Alimentarius CXS 66, Norma para las aceitunas de mesa.',
    url: 'https://www.fao.org/input/download/standards/243/CXS_066s.pdf',
    consultada: '2026-09-08',
  },
  'csic-perona': {
    cita: 'Sánchez Perona, J. y Berlanga Del Pozo, M. (2025). «Todas las aceitunas negras de mesa son de verdad». The Conversation España, 6 de marzo.',
    url: 'https://theconversation.com/todas-las-aceitunas-negras-de-mesa-son-de-verdad-249617',
    consultada: '2026-09-08',
  },
  'cicytex-acrilamida': {
    cita: 'Martín Vertedor, D. «¿Las aceitunas negras oxidadas al estilo californiano contienen acrilamida?». Revista Alimentaria (CICYTEX).',
    url: 'https://revistaalimentaria.es/agricultura/materias-primas/las-aceitunas-negras-oxidadas-al-estilo-californiano-contienen-acrilamida',
    consultada: '2026-09-08',
  },
  'mapa-aceituna': {
    cita: 'Ministerio de Agricultura, Pesca y Alimentación. Aceituna de mesa: variedades y producción.',
    url: 'https://www.mapa.gob.es/es/agricultura/temas/producciones-agricolas/aceite-oliva-y-aceituna-mesa/aceituna',
    consultada: '2026-09-08',
  },
} as const satisfies Record<string, Fuente>

export type FuenteId = keyof typeof FUENTES
```

`src/content/narrativa.ts`. El copy es definitivo, no un borrador: respeta el
vocabulario prohibido y no contiene ninguna cifra de proceso.

```ts
import type { FuenteId } from './fuentes'

export type Pantalla = {
  n: number
  id: string
  titulo: string
  entradilla?: string
  prosa: string[]
  fuentes?: FuenteId[]
}

export const PANTALLAS: Pantalla[] = [
  {
    n: 1,
    id: 'apertura',
    titulo: 'Esta aceituna nunca estuvo negra.',
    prosa: [
      'Negra uniforme, brillante, blanda, sin hueso. Exactamente igual que la de al lado, y que las cuarenta de la lata. Ninguna aceituna madura así.',
    ],
  },
  {
    n: 2,
    id: 'promesa',
    titulo: 'El color es una promesa',
    entradilla: 'Cuando compras «aceitunas negras», crees que compras tiempo.',
    prosa: [
      'Crees que compras un fruto que se quedó en la rama hasta diciembre, que pasó del verde al violeta y del violeta al granate mientras el resto del olivar se vendimiaba. Eso es lo que la palabra «negra» promete: madurez.',
      'Y ahora la parte incómoda, porque a lo mejor has oído la versión de internet: no, no las tiñen. No hay colorante. Quien te diga eso te está contando un cuento más fácil que la verdad, y encima falso.',
      'Lo que pasa es más raro y más interesante. El negro es de la aceituna. Es su propio pigmento. Lo que le han quitado es el tiempo.',
    ],
    fuentes: ['csic-perona'],
  },
  {
    n: 3,
    id: 'calendario',
    titulo: 'El calendario del olivo',
    entradilla: 'El color de una aceituna es una fecha.',
    prosa: [
      'En septiembre es verde y dura, del verde al amarillo paja. Entre octubre y noviembre entra en envero: aparecen los rosados, el rosa vino, el castaño. Y solo en plena madurez llega el negro de verdad, que casi nunca es negro.',
      'La norma española lo describe con una precisión que ya lo dice todo: negro rojizo, negro violáceo, violeta, negro verdoso o castaño oscuro. Cinco maneras de no ser negro.',
      'Y ninguna madura al mismo ritmo que su vecina. Un olivar en diciembre es un degradado, no un color.',
    ],
    fuentes: ['rd-679-2016'],
  },
  {
    n: 4,
    id: 'bifurcacion',
    titulo: 'Aquí el camino se parte en dos',
    entradilla: 'El mismo fruto. Dos destinos que no se vuelven a encontrar.',
    prosa: [
      'A la izquierda, la aceituna se queda en el árbol. A la derecha, se recoge antes de madurar y el color se consigue en una nave. Las dos acabarán negras en un envase. Solo una de ellas maduró.',
    ],
  },
  {
    n: 5,
    id: 'fabrica',
    titulo: 'El camino de la fábrica',
    entradilla: 'Recoger verde no es un descuido: es el plan.',
    prosa: [
      'El fruto se recoge sin madurar y se conserva en salmuera. Después recibe tratamientos alcalinos sucesivos, y entre uno y otro se le inyecta aire. La sosa le quita el amargor y le ablanda la carne; el aire hace el resto. Los compuestos fenólicos de la propia aceituna se oxidan y forman pigmentos oscuros. Nadie ha añadido color: se ha provocado.',
      'Y aquí está el detalle que lo explica todo. El resultado honesto de ese proceso no es negro. Es marrón muy oscuro. Así que al final se añade una sal de hierro (gluconato ferroso, E-579, o lactato ferroso, E-585) que se acopla a esos mismos fenoles y convierte el marrón en un negro uniforme e intenso.',
      'Ese paso es opcional. Los investigadores del CSIC que mejor lo han explicado lo dicen sin rodeos: se hace porque es lo que el mercado espera. Es decir: se añade un aditivo porque el color real no es lo bastante negro para venderse como negro.',
      'Luego viene el deshuesado, el envasado y la esterilización térmica. Y a la lata.',
    ],
    fuentes: ['rd-679-2016', 'csic-perona', 'aditivos-1129-2011'],
  },
  {
    n: 6,
    id: 'arbol',
    titulo: 'El camino del árbol',
    entradilla: 'No hay proceso. Hay espera.',
    prosa: [
      'La aceituna se queda colgando. Cambia de color sola, a su ritmo y sin ponerse de acuerdo con la de al lado. Se recoge en plena madurez, o poco antes.',
      'Y después, casi nada: salmuera, donde fermenta despacio; o sal seca, que la deshidrata y la arruga; y a veces hierbas, ajo, hinojo, tomillo, cáscara de naranja. La norma llama a eso «aliñado», y consiste literalmente en añadir condimentos o especias.',
      'El precio de esa espera se nota en la boca. La negra natural es ácida, salada y amarga. Sabe a algo. La oxidada es suave, casi neutra, poco salada y sin amargor, y por eso gusta a tanta gente: no molesta.',
      'No son la misma aceituna peor o mejor hecha. Son dos productos distintos. El problema es que solo uno de los dos lleva el nombre del otro.',
    ],
    fuentes: ['rd-679-2016', 'csic-perona'],
  },
  {
    n: 7,
    id: 'prueba',
    titulo: 'La prueba está en el envase',
    entradilla: 'Dos comprobaciones. La primera te vale sola.',
    prosa: [
      'La norma española obliga a que la denominación diga el color, y define dos tipos distintos: «negras», que son las oscurecidas mediante oxidación, y «negras naturales», que son las recogidas en plena madurez. Dos categorías, dos nombres, una sola palabra de diferencia.',
      'Así que la primera comprobación es la cara del envase, y es la buena: busca la palabra «naturales». Si no está, no lo son. No hace falta saber química.',
      'La segunda es la lista de ingredientes: si aparece gluconato ferroso (E-579) o lactato ferroso (E-585), es oxidada. Esto solo confirma lo que ya sabías por la primera.',
      'Y ahora lo que casi nadie sabe, que está en el artículo 12 de la misma norma: declarar el proceso de elaboración es voluntario. Voluntario. La lata está obligada a decirte el color y no está obligada a decirte cómo lo consiguió. La mención del color tampoco es obligatoria si el envase es transparente, porque entonces ya lo ves.',
      'Nadie está incumpliendo nada. Ahí está el asunto.',
    ],
    fuentes: ['rd-679-2016', 'aditivos-1129-2011'],
  },
  {
    n: 8,
    id: 'reconocerlas',
    titulo: 'Reconocerlas sin leer nada',
    entradilla: 'Cuando ya las tienes en el plato.',
    prosa: [
      'La oxidada es negra mate y de un negro idéntico en todas, de piel lisa y tensa, blanda al morder, dulzona y sin amargor, y casi siempre deshuesada. Su virtud es la uniformidad, y la uniformidad es la firma de la fábrica.',
      'La natural no se pone de acuerdo consigo misma: dentro del mismo tarro hay granates, violetas, castaños. Muchas están arrugadas. La piel tiene brillos tornasolados. Al morder es firme, y aparecen la sal, la acidez y un amargor de fondo que no se va.',
      'Si todas son exactamente del mismo color, ya sabes de dónde vienen.',
    ],
  },
  {
    n: 9,
    id: 'que-comprar',
    titulo: 'Qué pedir, y cómo pedirlo',
    entradilla: 'No hace falta memorizar marcas. Basta con dos palabras y unos nombres.',
    prosa: [
      'Las dos palabras son «negras naturales». Dichas así, en la tienda o leídas en el envase.',
      'Y luego los nombres propios. La negra de Aragón, de variedad Empeltre, curada en seco hasta quedar arrugada, dulce y mantecosa. La kalamata griega, morada, firme y punzante. Y en general cualquier aceituna que te vendan por su variedad y no por su color, porque quien nombra la variedad no tiene nada que esconder.',
      'La norma permite indicar la variedad de forma voluntaria, precedida de la palabra «variedad». Que alguien se moleste en ponerlo ya es una señal.',
      'Un aviso para que esto no se lea como una lista de buenos y malos: casi la mitad de la aceituna de mesa española es Hojiblanca, y otra buena parte Manzanilla. Son variedades excelentes, y muchas acaban oxidadas porque es lo que se vende. El problema no es la aceituna ni quien la cultiva.',
    ],
    fuentes: ['rd-679-2016', 'mapa-aceituna'],
  },
  {
    n: 10,
    id: 'honestidad',
    titulo: 'Lo que no te estoy diciendo',
    entradilla: 'Esta sección es la que hace que puedas creerte las nueve anteriores.',
    prosa: [
      'No te están envenenando. La sal de hierro está autorizada, con límite máximo, y una ración normal de aceitunas se queda muy lejos de cualquier cifra preocupante. Si algo hay que vigilar en una aceituna es la sal, y eso vale para las dos.',
      'No es un fraude. La etiqueta cumple. Y hay quien sostiene, con razones, que llamar «falsa» a la aceituna oxidada es injusto: el proceso tiene una base científica sólida, el pigmento es de la propia aceituna y el producto es una categoría legal reconocida con su propio sabor y sus defensores. Lo explican Javier Sánchez Perona, del Instituto de la Grasa del CSIC, y Marta Berlanga Del Pozo, en un artículo que se titula precisamente «Todas las aceitunas negras de mesa son de verdad». Merece la pena leerlo, y tienen razón en casi todo.',
      'Donde este sitio sigue discrepando es en otra cosa, y no es química: quien compra «aceitunas negras» cree estar comprando fruta madurada en el árbol, y la norma permite no aclarárselo. Ese hueco existe, es legal, y es del que hablamos aquí.',
      'Un dato que no voy a convertir en titular: en las negras oxidadas se ha medido acrilamida, y se forma en la esterilización térmica, no en la oxidación. En las elaboradas al estilo español no se detecta. La EFSA las tiene entre los alimentos a vigilar. Es información, no una alarma, y quien quiera el detalle tiene el estudio enlazado abajo.',
      'Y dos cosas que este sitio no sabe y por eso no dice: no he encontrado un estudio comparativo serio con cifras sobre el contenido de polifenoles de unas frente a otras, ni una fuente fiable con los tiempos exactos del proceso industrial. Cuando aparezcan, se añaden aquí.',
      'Las ilustraciones de esta página están generadas con inteligencia artificial. Son interpretaciones, no pruebas. Las pruebas son el Boletín Oficial del Estado y los enlaces que vienen a continuación.',
    ],
    fuentes: ['csic-perona', 'cicytex-acrilamida', 'aditivos-1129-2011', 'rd-679-2016', 'codex-66'],
  },
]
```

- [ ] **Step 4: Run test to verify it passes**

Run: `npx vitest run tests/copy.test.ts`
Expected: PASS — 7 tests. Si el recuento de palabras no llega a 1.400, ampliar la prosa de las pantallas 5, 7 y 10, que son las que admiten más detalle sin perder ritmo.

- [ ] **Step 5: Commit**

```bash
git add src/content tests/copy.test.ts
git commit -m "feat: copy de la narrativa y bibliografía tipada con guardianes de vocabulario"
```

---

### Task 5: Layout de la narrativa con SEO y datos estructurados

**Files:**
- Create: `src/layouts/Narrativa.astro`
- Create: `src/styles/narrativa.css`
- Create: `tests/seo.test.ts`
- Modify: `src/pages/index.astro`

**Interfaces:**
- Consumes: los estilos de las Tareas 2 y 3.
- Produces: `src/layouts/Narrativa.astro` con props `{ titulo: string; descripcion: string; canonical: string }` y un `<slot />`. Las Tareas 7 a 12 insertan secciones en ese slot.

- [ ] **Step 1: Write the failing test**

`tests/seo.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { htmlDeDist, leerDist } from './helpers/dist'

const portada = () => leerDist('index.html')

describe('SEO de la narrativa (spec §9)', () => {
  it('tiene lang es, título y descripción propios', () => {
    const h = portada()
    expect(h).toMatch(/<html[^>]*lang="es"/)
    expect(h).toMatch(/<title>[^<]{15,70}<\/title>/)
    expect(h).toMatch(/<meta name="description" content="[^"]{50,160}"/)
  })

  it('tiene canonical y tarjetas sociales', () => {
    const h = portada()
    expect(h).toMatch(/rel="canonical"/)
    expect(h).toMatch(/property="og:title"/)
    expect(h).toMatch(/property="og:description"/)
    expect(h).toMatch(/property="og:image"/)
    expect(h).toMatch(/name="twitter:card" content="summary_large_image"/)
  })

  it('lleva un JSON-LD de tipo Article que parsea', () => {
    const m = portada().match(
      /<script type="application\/ld\+json">([\s\S]*?)<\/script>/,
    )
    expect(m, 'no hay JSON-LD').not.toBeNull()
    const datos = JSON.parse(m![1])
    expect(datos['@type']).toBe('Article')
    expect(datos.headline.length).toBeGreaterThan(10)
  })

  it('cada página tiene exactamente un h1', () => {
    for (const { ruta, html } of htmlDeDist()) {
      const h1 = html.match(/<h1[\s>]/g) ?? []
      expect(h1.length, `${ruta} tiene ${h1.length} h1`).toBe(1)
    }
  })

  it('la narrativa no lleva menú de cabecera (spec §4.1)', () => {
    expect(portada()).not.toMatch(/<header[^>]*>[\s\S]*<nav/)
  })
})
```

Nota sobre el test del JSON-LD: `application/ld+json` no cuenta como JavaScript
ejecutable, pero el guardián de la Tarea 1 busca `<script`. Hay que refinar ese
test en el paso 3 de esta tarea.

- [ ] **Step 2: Run test to verify it fails**

Run: `npm test`
Expected: FAIL — `tests/seo.test.ts` falla en «no hay JSON-LD» y en la descripción.

- [ ] **Step 3: Write minimal implementation**

Primero, refinar el guardián de la Tarea 1 para que permita datos estructurados
pero siga prohibiendo JavaScript. En `tests/presupuesto.test.ts`, reemplazar el
test «ningún HTML contiene etiquetas `<script>`» por:

```ts
  it('ningún HTML contiene scripts ejecutables', () => {
    for (const { ruta, html } of htmlDeDist()) {
      const scripts = html.match(/<script\b[^>]*>/gi) ?? []
      for (const s of scripts)
        expect(s, `${ruta} tiene un script ejecutable`).toMatch(
          /type="application\/ld\+json"/,
        )
    }
  })
```

`src/layouts/Narrativa.astro`:
```astro
---
import '../styles/tokens.css'
import '../styles/base.css'
import '../styles/tipografia.css'
import '../styles/narrativa.css'

interface Props {
  titulo: string
  descripcion: string
  canonical: string
}
const { titulo, descripcion, canonical } = Astro.props

const jsonLd = {
  '@context': 'https://schema.org',
  '@type': 'Article',
  headline: titulo,
  description: descripcion,
  inLanguage: 'es',
  mainEntityOfPage: canonical,
  image: new URL('/og.jpg', Astro.site).href,
}
---
<html lang="es">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="color-scheme" content="light only" />
    <link rel="preload" href="/fonts/fraunces-latin-ext.woff2" as="font" type="font/woff2" crossorigin />
    <link rel="preload" href="/fonts/newsreader-latin-ext.woff2" as="font" type="font/woff2" crossorigin />
    <title>{titulo}</title>
    <meta name="description" content={descripcion} />
    <link rel="canonical" href={canonical} />
    <meta property="og:type" content="article" />
    <meta property="og:title" content={titulo} />
    <meta property="og:description" content={descripcion} />
    <meta property="og:image" content={new URL('/og.jpg', Astro.site).href} />
    <meta property="og:locale" content="es_ES" />
    <meta name="twitter:card" content="summary_large_image" />
    <script type="application/ld+json" set:html={JSON.stringify(jsonLd)} />
  </head>
  <body>
    <main><slot /></main>
  </body>
</html>
```

`src/styles/narrativa.css`:
```css
@layer narrativa {
  .pantalla {
    padding-block: clamp(4rem, 12vh, 9rem);
    padding-inline: 6vw;
    max-width: 1180px;
    margin-inline: auto;
  }

  .pantalla > .mono { display: block; margin-bottom: 1.6rem; }

  .entradilla {
    font-size: clamp(1.25rem, 2.4vw, 1.7rem);
    line-height: 1.42;
    max-width: 32ch;
    color: var(--terracota);
    margin: 1.6rem 0 2.4rem;
  }

  .prosa > p { max-width: var(--medida); margin: 0 0 1.4em; }
  .prosa em { color: var(--granate); font-style: italic; }
}
```

`src/pages/index.astro` — reemplazar todo el fichero:
```astro
---
import Narrativa from '../layouts/Narrativa.astro'

const titulo = 'Esta aceituna nunca estuvo negra'
const descripcion =
  'Las aceitunas negras de lata se recogen sin madurar y se ennegrecen por oxidación. La norma obliga a decirte el color, pero no el proceso. Aprende a leerlo en diez pantallas.'
---
<Narrativa titulo={titulo} descripcion={descripcion} canonical="https://nuncafuenegra.com/">
  <section class="pantalla" aria-labelledby="t-apertura">
    <h1 id="t-apertura" class="display">{titulo}.</h1>
  </section>
</Narrativa>
```

- [ ] **Step 4: Run test to verify it passes**

Run: `npm test`
Expected: PASS — 17 tests.

- [ ] **Step 5: Commit**

```bash
git add src/layouts src/styles/narrativa.css src/pages/index.astro tests/seo.test.ts tests/presupuesto.test.ts
git commit -m "feat: layout de la narrativa con metadatos y datos estructurados"
```

---

### Task 6: Componente de acuarela y sustitutas provisionales

**Files:**
- Create: `scripts/acuarelas-provisionales.mjs`
- Create: `src/assets/acuarelas/` (10 `.jpg` generados)
- Create: `src/components/ui/Acuarela.astro`
- Create: `tests/imagenes.test.ts`
- Modify: `package.json`

**Interfaces:**
- Consumes: los tokens de la Tarea 2.
- Produces: `src/components/ui/Acuarela.astro` con props `{ src: ImageMetadata; alt: string; prioritaria?: boolean }`. Las Tareas 7 a 12 lo usan importando la imagen desde `src/assets/acuarelas/`.

- [ ] **Step 1: Write the failing test**

`tests/imagenes.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { existsSync } from 'node:fs'
import { join } from 'node:path'
import { htmlDeDist, listarDist } from './helpers/dist'

const ACUARELAS = [
  '01-aceituna-negra', '02-rama-verde', '03-rama-envero', '04-rama-madura',
  '05-olivar-diciembre', '06-manos-cesta', '07-salmuera-tarro',
  '08-sal-hierbas', '09-nave-industrial', '10-lata-mesa',
]

describe('imágenes (spec §5.4, §8)', () => {
  it('existen los diez ficheros de acuarela con su nombre definitivo', () => {
    for (const n of ACUARELAS)
      expect(
        existsSync(join(process.cwd(), 'src/assets/acuarelas', `${n}.jpg`)),
        `falta ${n}.jpg`,
      ).toBe(true)
  })

  it('el build genera variantes AVIF', () => {
    expect(listarDist('.avif').length).toBeGreaterThan(0)
  })

  it('toda img lleva width y height explícitos: CLS 0', () => {
    for (const { ruta, html } of htmlDeDist())
      for (const img of html.match(/<img\b[^>]*>/g) ?? []) {
        expect(img, `${ruta}: img sin width`).toMatch(/\swidth="\d+"/)
        expect(img, `${ruta}: img sin height`).toMatch(/\sheight="\d+"/)
        expect(img, `${ruta}: img sin alt`).toMatch(/\salt="/)
      }
  })

  it('solo la primera imagen de la portada carga con prioridad', () => {
    const html = htmlDeDist().find((h) => h.ruta === 'index.html')!.html
    const imgs = html.match(/<img\b[^>]*>/g) ?? []
    if (imgs.length > 1)
      for (const img of imgs.slice(1))
        expect(img, 'imagen no diferida').toMatch(/loading="lazy"/)
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npx vitest run tests/imagenes.test.ts`
Expected: FAIL — «falta 01-aceituna-negra.jpg».

- [ ] **Step 3: Write minimal implementation**

`scripts/acuarelas-provisionales.mjs`. Genera sustitutas con las dimensiones y
nombres definitivos, en los colores de la paleta, para que el build y los tests
funcionen antes de que existan las acuarelas reales:

```js
import sharp from 'sharp'
import { mkdirSync, existsSync } from 'node:fs'
import { join } from 'node:path'

const DESTINO = join(process.cwd(), 'src/assets/acuarelas')
mkdirSync(DESTINO, { recursive: true })

// Fondo blanco a propósito: las definitivas llegarán sobre papel blanco y se
// integran con mix-blend-mode: multiply (spec §5.4).
const PIEZAS = [
  ['01-aceituna-negra', 1600, 1200, '#2B2318'],
  ['02-rama-verde', 1600, 1067, '#6E7444'],
  ['03-rama-envero', 1600, 1067, '#9C5232'],
  ['04-rama-madura', 1600, 1067, '#6B2B3E'],
  ['05-olivar-diciembre', 2000, 1125, '#B8792F'],
  ['06-manos-cesta', 1600, 1067, '#8A7B63'],
  ['07-salmuera-tarro', 1200, 1500, '#6E7444'],
  ['08-sal-hierbas', 1600, 1067, '#C4B393'],
  ['09-nave-industrial', 2000, 1125, '#5C4F3C'],
  ['10-lata-mesa', 1600, 1067, '#9C5232'],
]

for (const [nombre, w, h, color] of PIEZAS) {
  const ruta = join(DESTINO, `${nombre}.jpg`)
  if (existsSync(ruta)) {
    console.log(`· ${nombre}.jpg ya existe, no lo toco`)
    continue
  }
  const margen = Math.round(Math.min(w, h) * 0.12)
  await sharp({
    create: { width: w, height: h, channels: 3, background: '#ffffff' },
  })
    .composite([
      {
        input: await sharp({
          create: {
            width: w - margen * 2,
            height: h - margen * 2,
            channels: 3,
            background: color,
          },
        })
          .jpeg()
          .toBuffer(),
        top: margen,
        left: margen,
      },
    ])
    .jpeg({ quality: 82 })
    .toFile(ruta)
  console.log(`✓ ${nombre}.jpg (${w}×${h})`)
}
```

Añadir a los scripts de `package.json`:
```json
    "acuarelas": "node scripts/acuarelas-provisionales.mjs",
```

`src/components/ui/Acuarela.astro`. El `multiply` hace desaparecer el papel
blanco y asienta el pigmento sobre el arena (spec §5.4):

```astro
---
import { Picture } from 'astro:assets'

interface Props {
  src: ImageMetadata
  alt: string
  prioritaria?: boolean
}
const { src, alt, prioritaria = false } = Astro.props
---
<figure class="acuarela">
  <Picture
    src={src}
    alt={alt}
    formats={['avif', 'webp']}
    widths={[640, 960, 1440, 1920]}
    sizes="(max-width: 900px) 100vw, 1180px"
    loading={prioritaria ? 'eager' : 'lazy'}
    decoding={prioritaria ? 'sync' : 'async'}
  />
</figure>

<style>
  .acuarela { margin: 0; }
  .acuarela :global(img) {
    display: block;
    width: 100%;
    height: auto;
    mix-blend-mode: multiply;
  }
</style>
```

- [ ] **Step 4: Run test to verify it passes**

Run: `npm run acuarelas && npm test`
Expected: PASS — 21 tests. El test de AVIF requiere que alguna pantalla use ya el
componente; si aún no hay ninguna, añadir temporalmente la acuarela 01 a la
pantalla de apertura de `src/pages/index.astro` con `prioritaria`.

- [ ] **Step 5: Commit**

```bash
git add scripts src/assets src/components package.json tests/imagenes.test.ts src/pages/index.astro
git commit -m "feat: componente de acuarela en avif con sustitutas provisionales"
```

---

### Task 7: Pantallas 1 a 3 — apertura, promesa y calendario

**Files:**
- Create: `src/components/narrativa/01Apertura.astro`
- Create: `src/components/narrativa/02Promesa.astro`
- Create: `src/components/narrativa/03Calendario.astro`
- Create: `src/components/ui/Prosa.astro`
- Create: `tests/narrativa.test.ts`
- Modify: `src/pages/index.astro`

**Interfaces:**
- Consumes: `PANTALLAS` (Tarea 4), `Narrativa.astro` (Tarea 5), `Acuarela.astro` (Tarea 6).
- Produces: `src/components/ui/Prosa.astro` con props `{ parrafos: string[] }`, usado por todas las pantallas siguientes. Cada componente de pantalla exporta por defecto una `<section class="pantalla" id="...">` sin props.

- [ ] **Step 1: Write the failing test**

`tests/narrativa.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { leerDist } from './helpers/dist'
import { PANTALLAS } from '../src/content/narrativa'

const portada = () => leerDist('index.html')

describe('pantallas de la narrativa (spec §4.1)', () => {
  it('cada pantalla implementada tiene su section con id y aria-labelledby', () => {
    const h = portada()
    for (const p of PANTALLAS.filter((x) => x.n <= 3)) {
      expect(h, `falta la section de ${p.id}`).toMatch(
        new RegExp(`<section[^>]*id="${p.id}"`),
      )
      expect(h, `${p.id} sin aria-labelledby`).toMatch(
        new RegExp(`<section[^>]*id="${p.id}"[^>]*aria-labelledby="t-${p.id}"`),
      )
    }
  })

  it('la prosa de las pantallas 1 a 3 está en el HTML', () => {
    const h = portada()
    for (const p of PANTALLAS.filter((x) => x.n <= 3))
      for (const parrafo of p.prosa) {
        const trozo = parrafo.slice(0, 40).replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
        expect(h, `no aparece "${trozo}"`).toMatch(new RegExp(trozo))
      }
  })

  it('la apertura es el único h1; las demás pantallas usan h2', () => {
    const h = portada()
    expect((h.match(/<h1[\s>]/g) ?? []).length).toBe(1)
    expect((h.match(/<h2[\s>]/g) ?? []).length).toBeGreaterThanOrEqual(2)
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npm test`
Expected: FAIL — «falta la section de promesa».

- [ ] **Step 3: Write minimal implementation**

`src/components/ui/Prosa.astro`:
```astro
---
interface Props { parrafos: string[] }
const { parrafos } = Astro.props
---
<div class="prosa cuerpo">
  {parrafos.map((p) => <p>{p}</p>)}
</div>
```

`src/components/narrativa/01Apertura.astro`:
```astro
---
import { PANTALLAS } from '../../content/narrativa'
import Prosa from '../ui/Prosa.astro'
import Acuarela from '../ui/Acuarela.astro'
import aceituna from '../../assets/acuarelas/01-aceituna-negra.jpg'

const p = PANTALLAS.find((x) => x.n === 1)!
---
<section class="pantalla apertura" id={p.id} aria-labelledby={`t-${p.id}`}>
  <span class="mono">01 / Apertura</span>
  <h1 id={`t-${p.id}`} class="display">{p.titulo}</h1>
  <Acuarela src={aceituna} alt="Acuarela de una aceituna negra de superficie uniforme y brillante" prioritaria />
  <Prosa parrafos={p.prosa} />
</section>

<style>
  .apertura { min-height: 92svh; display: grid; align-content: center; }
  .apertura :global(.acuarela) { max-width: 22rem; margin-block: 2.5rem; }
</style>
```

`src/components/narrativa/02Promesa.astro`:
```astro
---
import { PANTALLAS } from '../../content/narrativa'
import Prosa from '../ui/Prosa.astro'

const p = PANTALLAS.find((x) => x.n === 2)!
---
<section class="pantalla" id={p.id} aria-labelledby={`t-${p.id}`}>
  <span class="mono">02 / El color es una promesa</span>
  <h2 id={`t-${p.id}`} class="display">{p.titulo}</h2>
  <p class="entradilla">{p.entradilla}</p>
  <Prosa parrafos={p.prosa} />
</section>
```

`src/components/narrativa/03Calendario.astro`. El degradado de fechas se dibuja
con CSS y los tokens de la paleta, no con una imagen:
```astro
---
import { PANTALLAS } from '../../content/narrativa'
import Prosa from '../ui/Prosa.astro'
import Acuarela from '../ui/Acuarela.astro'
import verde from '../../assets/acuarelas/02-rama-verde.jpg'
import envero from '../../assets/acuarelas/03-rama-envero.jpg'
import madura from '../../assets/acuarelas/04-rama-madura.jpg'

const p = PANTALLAS.find((x) => x.n === 3)!
const MESES = [
  { mes: 'Septiembre', color: 'var(--oliva)', nota: 'verde al amarillo paja' },
  { mes: 'Octubre', color: 'var(--ocre)', nota: 'empieza el envero' },
  { mes: 'Noviembre', color: 'var(--terracota)', nota: 'rosa vino, castaño' },
  { mes: 'Diciembre', color: 'var(--granate)', nota: 'plena madurez' },
]
---
<section class="pantalla" id={p.id} aria-labelledby={`t-${p.id}`}>
  <span class="mono">03 / El calendario del olivo</span>
  <h2 id={`t-${p.id}`} class="display">{p.titulo}</h2>
  <p class="entradilla">{p.entradilla}</p>

  <ol class="calendario">
    {MESES.map((m) => (
      <li>
        <span class="punto" style={`--c:${m.color}`} aria-hidden="true"></span>
        <b class="aparato">{m.mes}</b>
        <span class="nota">{m.nota}</span>
      </li>
    ))}
  </ol>

  <div class="ramas">
    <Acuarela src={verde} alt="Acuarela de una rama con aceitunas verdes en septiembre" />
    <Acuarela src={envero} alt="Acuarela de una rama en envero, con aceitunas rosadas y castañas" />
    <Acuarela src={madura} alt="Acuarela de una rama con aceitunas maduras de color granate y arrugadas" />
  </div>

  <Prosa parrafos={p.prosa} />
</section>

<style>
  .calendario {
    list-style: none;
    padding: 0;
    margin: 2.5rem 0;
    display: grid;
    gap: 1.1rem;
  }
  .calendario li { display: flex; align-items: baseline; gap: 0.9rem; }
  .punto {
    width: 0.8rem; height: 0.8rem; border-radius: 50%;
    background: var(--c); flex: none; translate: 0 -0.1rem;
  }
  .calendario b { min-width: 9ch; }
  .nota { color: var(--tinta-70); }

  .ramas {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 1.5rem;
    margin-block: 2.5rem 3rem;
  }
  @media (max-width: 700px) { .ramas { grid-template-columns: 1fr; } }
</style>
```

`src/pages/index.astro` — reemplazar el `<section>` provisional:
```astro
---
import Narrativa from '../layouts/Narrativa.astro'
import Apertura from '../components/narrativa/01Apertura.astro'
import Promesa from '../components/narrativa/02Promesa.astro'
import Calendario from '../components/narrativa/03Calendario.astro'

const titulo = 'Esta aceituna nunca estuvo negra'
const descripcion =
  'Las aceitunas negras de lata se recogen sin madurar y se ennegrecen por oxidación. La norma obliga a decirte el color, pero no el proceso. Aprende a leerlo en diez pantallas.'
---
<Narrativa titulo={titulo} descripcion={descripcion} canonical="https://nuncafuenegra.com/">
  <Apertura />
  <Promesa />
  <Calendario />
</Narrativa>
```

- [ ] **Step 4: Run test to verify it passes**

Run: `npm test`
Expected: PASS — 24 tests.

- [ ] **Step 5: Commit**

```bash
git add src/components src/pages/index.astro tests/narrativa.test.ts
git commit -m "feat: pantallas de apertura, promesa y calendario del olivo"
```

---

### Task 8: Pantalla 4 — la bifurcación

Es el punto de no retorno del sitio: dos columnas que no se vuelven a juntar.

**Files:**
- Create: `src/components/figuras/Bifurcacion.astro`
- Create: `src/components/narrativa/04Bifurcacion.astro`
- Create: `tests/bifurcacion.test.ts`
- Modify: `src/pages/index.astro`

**Interfaces:**
- Consumes: `PANTALLAS` (Tarea 4), `Prosa.astro` (Tarea 7).
- Produces: las clases `.camino-arbol` y `.camino-fabrica`, que las Tareas 9 a 12 reutilizan. **`.camino-fabrica` es el único ámbito donde se permite `var(--negro)`.**

- [ ] **Step 1: Write the failing test**

`tests/bifurcacion.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { readFileSync } from 'node:fs'
import { join } from 'node:path'
import { leerDist } from './helpers/dist'

const fuente = (rel: string) => readFileSync(join(process.cwd(), rel), 'utf8')

describe('la bifurcación (spec §4.1 pantalla 4, §6)', () => {
  it('está en el HTML con los dos caminos rotulados', () => {
    const h = leerDist('index.html')
    expect(h).toMatch(/<section[^>]*id="bifurcacion"/)
    expect(h).toMatch(/camino-arbol/)
    expect(h).toMatch(/camino-fabrica/)
  })

  it('el SVG es decorativo y no aporta información sola', () => {
    const h = leerDist('index.html')
    const svg = h.match(/<svg\b[^>]*>/g) ?? []
    expect(svg.length).toBeGreaterThan(0)
    for (const s of svg) expect(s).toMatch(/aria-hidden="true"/)
  })

  it('en móvil las columnas se apilan con rótulo explícito por camino', () => {
    const css = fuente('src/components/narrativa/04Bifurcacion.astro')
    expect(css).toMatch(/@media[^{]*max-width[^{]*\{[\s\S]*grid-template-columns:\s*1fr/)
    expect(css).toMatch(/rotulo/)
  })

  it('var(--negro) solo se usa dentro del ámbito .camino-fabrica', () => {
    for (const rel of [
      'src/components/figuras/Bifurcacion.astro',
      'src/components/narrativa/04Bifurcacion.astro',
    ]) {
      const texto = fuente(rel)
      const bloques = texto.match(/<style>[\s\S]*?<\/style>/g) ?? []
      for (const b of bloques)
        for (const regla of b.split('}'))
          if (regla.includes('var(--negro)'))
            expect(regla, `${rel}: --negro fuera de fábrica`).toMatch(
              /\.camino-fabrica/,
            )
    }
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npm test`
Expected: FAIL — no existe la section `bifurcacion`.

- [ ] **Step 3: Write minimal implementation**

`src/components/figuras/Bifurcacion.astro`. El trazo que se parte en dos:
```astro
---
---
<svg class="fig-bifurcacion" viewBox="0 0 400 220" aria-hidden="true" focusable="false">
  <path d="M200 0 V70" />
  <path d="M200 70 C200 130 60 120 60 220" class="rama-arbol" />
  <path d="M200 70 C200 130 340 120 340 220" class="rama-fabrica" />
  <circle cx="200" cy="70" r="5" class="nudo" />
</svg>

<style>
  .fig-bifurcacion { width: 100%; height: auto; max-width: 26rem; margin-inline: auto; display: block; }
  .fig-bifurcacion path { fill: none; stroke-width: 2; }
  .fig-bifurcacion path:first-of-type { stroke: var(--tinta-45); }
  .rama-arbol { stroke: var(--granate); }
  .rama-fabrica { stroke: var(--tinta-45); stroke-dasharray: 6 5; }
  .nudo { fill: var(--tinta); }
</style>
```

`src/components/narrativa/04Bifurcacion.astro`:
```astro
---
import { PANTALLAS } from '../../content/narrativa'
import Prosa from '../ui/Prosa.astro'
import Bifurcacion from '../figuras/Bifurcacion.astro'

const p = PANTALLAS.find((x) => x.n === 4)!
---
<section class="pantalla" id={p.id} aria-labelledby={`t-${p.id}`}>
  <span class="mono">04 / La bifurcación</span>
  <h2 id={`t-${p.id}`} class="display">{p.titulo}</h2>
  <p class="entradilla">{p.entradilla}</p>

  <Bifurcacion />

  <div class="caminos">
    <div class="camino camino-arbol">
      <span class="rotulo mono">Camino del árbol</span>
      <p class="cuerpo">Se queda en la rama. Madura sola, a su ritmo, y ninguna igual que la de al lado.</p>
    </div>
    <div class="camino camino-fabrica">
      <span class="rotulo aparato">CAMINO DE LA FÁBRICA</span>
      <p class="aparato">Se recoge sin madurar. El color se consigue después, y sale idéntico en todas.</p>
    </div>
  </div>

  <Prosa parrafos={p.prosa} />
</section>

<style>
  .caminos {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 0;
    margin-block: 2.5rem 3rem;
    border-block: 1px solid var(--linea);
  }
  .camino { padding: 1.8rem 1.6rem; }
  .camino-arbol { border-right: 1px solid var(--linea); }
  .camino-arbol .rotulo { color: var(--granate); }
  .camino-arbol p { max-width: 28ch; margin: 0.9rem 0 0; }

  .camino-fabrica .rotulo { color: var(--negro); letter-spacing: 0.2em; }
  .camino-fabrica p { max-width: 34ch; margin: 0.9rem 0 0; color: var(--tinta-70); }

  @media (max-width: 780px) {
    .caminos { grid-template-columns: 1fr; }
    .camino-arbol { border-right: 0; border-bottom: 1px solid var(--linea); }
  }
</style>
```

Añadir en `src/pages/index.astro`: el import
`import Bifurcacion from '../components/narrativa/04Bifurcacion.astro'` y
`<Bifurcacion />` tras `<Calendario />`.

- [ ] **Step 4: Run test to verify it passes**

Run: `npm test`
Expected: PASS — 28 tests.

- [ ] **Step 5: Commit**

```bash
git add src/components src/pages/index.astro tests/bifurcacion.test.ts
git commit -m "feat: pantalla de la bifurcación con los dos caminos"
```

---

### Task 9: Pantallas 5 y 6 — los dos caminos, con la tipografía como argumento

La fábrica se compone en monoespaciada; el árbol en serif. Es la idea del §5.1.

**Files:**
- Create: `src/components/figuras/LineaProceso.astro`
- Create: `src/components/narrativa/05Fabrica.astro`
- Create: `src/components/narrativa/06Arbol.astro`
- Create: `tests/caminos.test.ts`
- Modify: `src/pages/index.astro`

**Interfaces:**
- Consumes: `PANTALLAS`, `Prosa.astro`, `Acuarela.astro`, las clases `.camino-*` de la Tarea 8.
- Produces: `LineaProceso.astro` con props `{ pasos: { paso: string; nota: string }[] }`.

- [ ] **Step 1: Write the failing test**

`tests/caminos.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { readFileSync } from 'node:fs'
import { join } from 'node:path'
import { leerDist } from './helpers/dist'

const fuente = (rel: string) => readFileSync(join(process.cwd(), rel), 'utf8')

describe('los dos caminos (spec §4.1, §5.1)', () => {
  it('ambas pantallas están en el HTML', () => {
    const h = leerDist('index.html')
    expect(h).toMatch(/<section[^>]*id="fabrica"/)
    expect(h).toMatch(/<section[^>]*id="arbol"/)
  })

  it('la fábrica se compone en monoespaciada y el árbol en serif', () => {
    expect(fuente('src/components/narrativa/05Fabrica.astro')).toMatch(
      /class="[^"]*aparato/,
    )
    expect(fuente('src/components/narrativa/06Arbol.astro')).toMatch(
      /class="[^"]*cuerpo/,
    )
  })

  it('la pantalla de fábrica dice que sin el hierro el resultado es marrón', () => {
    expect(leerDist('index.html')).toMatch(/marrón muy oscuro/)
  })

  it('no cita ninguna cifra de tiempos del proceso (spec §10.3)', () => {
    const h = leerDist('index.html')
    const seccion = h.slice(h.indexOf('id="fabrica"'), h.indexOf('id="arbol"'))
    expect(seccion, 'aparece una duración concreta').not.toMatch(
      /\d+\s*(?:h|horas|min|minutos|días|ºC|°C)\b/,
    )
  })

  it('la línea de proceso es una lista ordenada accesible', () => {
    expect(fuente('src/components/figuras/LineaProceso.astro')).toMatch(/<ol/)
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npm test`
Expected: FAIL — no existe la section `fabrica`.

- [ ] **Step 3: Write minimal implementation**

`src/components/figuras/LineaProceso.astro`:
```astro
---
interface Props { pasos: { paso: string; nota: string }[] }
const { pasos } = Astro.props
---
<ol class="linea-proceso">
  {pasos.map((p, i) => (
    <li>
      <span class="num aparato">{String(i + 1).padStart(2, '0')}</span>
      <div>
        <b class="aparato">{p.paso}</b>
        <span class="nota aparato">{p.nota}</span>
      </div>
    </li>
  ))}
</ol>

<style>
  .linea-proceso {
    list-style: none;
    padding: 0;
    margin: 2.2rem 0 3rem;
    display: grid;
    gap: 0;
    border-top: 1px solid var(--linea);
  }
  .linea-proceso li {
    display: flex;
    gap: 1.2rem;
    padding: 1rem 0;
    border-bottom: 1px solid var(--linea);
  }
  .num { color: var(--tinta-45); min-width: 3ch; }
  .linea-proceso b { display: block; font-weight: 500; }
  .nota { display: block; color: var(--tinta-70); margin-top: 0.25rem; }
</style>
```

`src/components/narrativa/05Fabrica.astro`:
```astro
---
import { PANTALLAS } from '../../content/narrativa'
import Prosa from '../ui/Prosa.astro'
import Acuarela from '../ui/Acuarela.astro'
import LineaProceso from '../figuras/LineaProceso.astro'
import nave from '../../assets/acuarelas/09-nave-industrial.jpg'

const p = PANTALLAS.find((x) => x.n === 5)!
const PASOS = [
  { paso: 'Recogida sin madurar', nota: 'el fruto todavía no ha cambiado de color' },
  { paso: 'Conservación en salmuera', nota: 'fermentada o no' },
  { paso: 'Tratamientos alcalinos sucesivos', nota: 'quitan el amargor y ablandan la carne' },
  { paso: 'Inyección de aire entre tratamientos', nota: 'oxida los fenoles de la propia aceituna' },
  { paso: 'Fijación con sal de hierro', nota: 'E-579 o E-585; convierte el marrón en negro uniforme' },
  { paso: 'Deshuesado y envasado', nota: '' },
  { paso: 'Esterilización térmica', nota: 'el producto no es estable sin ella' },
]
---
<section class="pantalla camino-fabrica" id={p.id} aria-labelledby={`t-${p.id}`}>
  <span class="mono">05 / El camino de la fábrica</span>
  <h2 id={`t-${p.id}`} class="display">{p.titulo}</h2>
  <p class="entradilla">{p.entradilla}</p>

  <LineaProceso pasos={PASOS} />
  <Acuarela src={nave} alt="Acuarela de una nave industrial con depósitos y tuberías" />
  <Prosa parrafos={p.prosa} />
</section>

<style>
  .camino-fabrica .display { color: var(--negro); }
  .camino-fabrica .entradilla { color: var(--tinta-70); }
</style>
```

`src/components/narrativa/06Arbol.astro`:
```astro
---
import { PANTALLAS } from '../../content/narrativa'
import Prosa from '../ui/Prosa.astro'
import Acuarela from '../ui/Acuarela.astro'
import salmuera from '../../assets/acuarelas/07-salmuera-tarro.jpg'
import sal from '../../assets/acuarelas/08-sal-hierbas.jpg'

const p = PANTALLAS.find((x) => x.n === 6)!
const ESPERA = [
  { hito: 'Se queda en la rama', tiempo: 'meses' },
  { hito: 'Cambia de color sola', tiempo: 'a su ritmo' },
  { hito: 'Salmuera, o sal seca', tiempo: 'fermenta despacio' },
  { hito: 'Hierbas, si se quiere', tiempo: 'el «aliñado» de la norma' },
]
---
<section class="pantalla camino-arbol" id={p.id} aria-labelledby={`t-${p.id}`}>
  <span class="mono">06 / El camino del árbol</span>
  <h2 id={`t-${p.id}`} class="display">{p.titulo}</h2>
  <p class="entradilla">{p.entradilla}</p>

  <dl class="espera cuerpo">
    {ESPERA.map((e) => (
      <div class="fila">
        <dt>{e.hito}</dt>
        <dd>{e.tiempo}</dd>
      </div>
    ))}
  </dl>

  <div class="par">
    <Acuarela src={salmuera} alt="Acuarela de aceitunas en salmuera dentro de un tarro de vidrio" />
    <Acuarela src={sal} alt="Acuarela de sal gruesa y hierbas aromáticas" />
  </div>

  <Prosa parrafos={p.prosa} />
</section>

<style>
  .espera { margin: 2.2rem 0 3rem; border-top: 1px solid var(--linea); }
  .espera .fila {
    display: flex; justify-content: space-between; gap: 2rem;
    padding: 1rem 0; border-bottom: 1px solid var(--linea);
  }
  .espera dt { font-style: italic; }
  .espera dd { margin: 0; color: var(--granate); }

  .par { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 1.5rem; margin-bottom: 3rem; }
  @media (max-width: 700px) { .par { grid-template-columns: 1fr; } }
</style>
```

Añadir en `src/pages/index.astro` los imports de `05Fabrica.astro` y
`06Arbol.astro` y sus componentes `<Fabrica />` y `<Arbol />` tras `<Bifurcacion />`.

- [ ] **Step 4: Run test to verify it passes**

Run: `npm test`
Expected: PASS — 33 tests.

- [ ] **Step 5: Commit**

```bash
git add src/components src/pages/index.astro tests/caminos.test.ts
git commit -m "feat: pantallas de fábrica y árbol con la tipografía como argumento"
```

---

### Task 10: Pantalla 7 — el decodificador de etiqueta

Es el remate útil del sitio, y va sin JavaScript: la etiqueta de ejemplo es
genérica y compuesta (spec §3).

**Files:**
- Create: `src/components/figuras/Decodificador.astro`
- Create: `src/components/narrativa/07Prueba.astro`
- Create: `tests/decodificador.test.ts`
- Modify: `src/pages/index.astro`

**Interfaces:**
- Consumes: `PANTALLAS`, `Prosa.astro`.
- Produces: `Decodificador.astro`, sin props.

- [ ] **Step 1: Write the failing test**

`tests/decodificador.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { readFileSync } from 'node:fs'
import { join } from 'node:path'
import { leerDist } from './helpers/dist'

const fuente = (rel: string) => readFileSync(join(process.cwd(), rel), 'utf8')

describe('el decodificador (spec §4.1 pantalla 7)', () => {
  it('está en el HTML', () => {
    expect(leerDist('index.html')).toMatch(/<section[^>]*id="prueba"/)
  })

  it('destaca la ausencia de «naturales» como prueba principal', () => {
    const h = leerDist('index.html')
    const i = h.indexOf('id="prueba"')
    const seccion = h.slice(i, i + 6000)
    expect(seccion).toMatch(/naturales/)
    expect(seccion.indexOf('naturales')).toBeLessThan(seccion.indexOf('E-579'))
  })

  it('cita el artículo 12 y que declarar el proceso es voluntario', () => {
    const h = leerDist('index.html')
    expect(h).toMatch(/art[íi]culo 12/i)
    expect(h).toMatch(/voluntario/i)
  })

  it('la etiqueta de ejemplo no nombra ninguna marca', () => {
    const texto = fuente('src/components/figuras/Decodificador.astro')
    for (const marca of [/mercadona/i, /carrefour/i, /hacendado/i, /fragata/i])
      expect(texto, `nombra una marca: ${marca}`).not.toMatch(marca)
    expect(texto).toMatch(/ejemplo/i)
  })

  it('funciona sin JavaScript: el resaltado es CSS', () => {
    const texto = fuente('src/components/figuras/Decodificador.astro')
    expect(texto).not.toMatch(/<script/)
    expect(texto).toMatch(/<mark/)
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npm test`
Expected: FAIL — no existe la section `prueba`.

- [ ] **Step 3: Write minimal implementation**

`src/components/figuras/Decodificador.astro`:
```astro
---
---
<div class="decodificador">
  <p class="aviso mono">Etiqueta de ejemplo. Composición genérica, no corresponde a ninguna marca.</p>

  <div class="envase">
    <span class="cap aparato">1 · La cara del envase</span>
    <p class="denominacion display-min">Aceitunas negras <span class="falta" aria-hidden="true">·</span></p>
    <p class="veredicto aparato">
      Falta la palabra <mark class="clave">naturales</mark>. Con esto ya lo sabes:
      son aceitunas oscurecidas mediante oxidación.
    </p>
    <p class="veredicto aparato bien">
      Si dijera <b>«aceitunas negras naturales»</b>, serían fruto recogido en plena madurez.
    </p>
  </div>

  <div class="envase">
    <span class="cap aparato">2 · La letra pequeña</span>
    <p class="ingredientes aparato">
      INGREDIENTES: aceitunas de mesa negras, agua, sal,
      <mark>gluconato ferroso (E-579)</mark>, ácido láctico, corrector de acidez.
    </p>
    <p class="veredicto aparato">
      El <mark>E-579</mark> (o el E-585, lactato ferroso) solo está autorizado en
      aceitunas ennegrecidas por oxidación. Confirma lo de arriba.
    </p>
  </div>
</div>

<style>
  .decodificador { margin-block: 2.4rem 3rem; display: grid; gap: 1.4rem; }
  .aviso { color: var(--tinta-45); letter-spacing: 0.12em; margin: 0; }

  .envase { border: 1px solid var(--linea); padding: 1.6rem 1.7rem; }
  .cap {
    display: block; font-size: 0.58rem; letter-spacing: 0.24em;
    text-transform: uppercase; color: var(--tinta-45); margin-bottom: 1rem;
  }

  .denominacion {
    font-family: 'Fraunces', Georgia, serif;
    font-variation-settings: 'opsz' 60, 'SOFT' 45, 'WONK' 0;
    font-size: clamp(1.6rem, 4vw, 2.4rem);
    line-height: 1.1; margin: 0 0 1.1rem;
  }
  .falta {
    display: inline-block; width: 5.5ch;
    border-bottom: 2px dashed var(--granate);
    color: transparent;
  }

  .ingredientes { margin: 0 0 1.1rem; color: var(--tinta-70); max-width: 52ch; }

  .veredicto { margin: 0 0 0.6rem; max-width: 54ch; color: var(--tinta-70); }
  .veredicto:last-child { margin-bottom: 0; }
  .veredicto.bien { color: var(--oliva); }

  mark {
    background: none; color: var(--granate); font-weight: 500;
    box-shadow: inset 0 -0.45em 0 color-mix(in srgb, var(--granate) 14%, transparent);
  }
  mark.clave { font-style: italic; }
</style>
```

`src/components/narrativa/07Prueba.astro`:
```astro
---
import { PANTALLAS } from '../../content/narrativa'
import Prosa from '../ui/Prosa.astro'
import Decodificador from '../figuras/Decodificador.astro'
import lata from '../../assets/acuarelas/10-lata-mesa.jpg'
import Acuarela from '../ui/Acuarela.astro'

const p = PANTALLAS.find((x) => x.n === 7)!
---
<section class="pantalla" id={p.id} aria-labelledby={`t-${p.id}`}>
  <span class="mono">07 / La prueba</span>
  <h2 id={`t-${p.id}`} class="display">{p.titulo}</h2>
  <p class="entradilla">{p.entradilla}</p>

  <Decodificador />
  <Acuarela src={lata} alt="Acuarela de una lata de aceitunas abierta sobre una mesa" />
  <Prosa parrafos={p.prosa} />
</section>
```

Añadir el import y `<Prueba />` en `src/pages/index.astro`, tras `<Arbol />`.

- [ ] **Step 4: Run test to verify it passes**

Run: `npm test`
Expected: PASS — 38 tests.

- [ ] **Step 5: Commit**

```bash
git add src/components src/pages/index.astro tests/decodificador.test.ts
git commit -m "feat: decodificador de etiqueta sin javascript"
```

---

### Task 11: Pantalla 8 — el comparador visual

**Files:**
- Create: `src/components/figuras/ComparadorColor.astro`
- Create: `src/components/narrativa/08Reconocerlas.astro`
- Create: `tests/comparador.test.ts`
- Modify: `src/pages/index.astro`

**Interfaces:**
- Consumes: `PANTALLAS`, `Prosa.astro`, las clases `.camino-*` de la Tarea 8.
- Produces: `ComparadorColor.astro`, sin props.

- [ ] **Step 1: Write the failing test**

`tests/comparador.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { readFileSync } from 'node:fs'
import { join } from 'node:path'
import { leerDist } from './helpers/dist'

const fuente = (rel: string) => readFileSync(join(process.cwd(), rel), 'utf8')

describe('el comparador (spec §4.1 pantalla 8, §5.1)', () => {
  it('está en el HTML', () => {
    expect(leerDist('index.html')).toMatch(/<section[^>]*id="reconocerlas"/)
  })

  it('la comparación es una tabla, legible con lector de pantalla', () => {
    const t = fuente('src/components/figuras/ComparadorColor.astro')
    expect(t).toMatch(/<table/)
    expect(t).toMatch(/<th\b[^>]*scope="col"/)
    expect(t).toMatch(/<th\b[^>]*scope="row"/)
  })

  it('las muestras de color son decorativas y llevan texto equivalente', () => {
    const t = fuente('src/components/figuras/ComparadorColor.astro')
    for (const m of t.match(/<span class="muestra[^>]*>/g) ?? [])
      expect(m).toMatch(/aria-hidden="true"/)
  })

  it('el negro plano de la oxidada usa --negro dentro de camino-fabrica', () => {
    const t = fuente('src/components/figuras/ComparadorColor.astro')
    for (const regla of (t.match(/<style>[\s\S]*?<\/style>/)?.[0] ?? '').split('}'))
      if (regla.includes('var(--negro)'))
        expect(regla).toMatch(/camino-fabrica/)
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npm test`
Expected: FAIL — no existe la section `reconocerlas`.

- [ ] **Step 3: Write minimal implementation**

`src/components/figuras/ComparadorColor.astro`:
```astro
---
const FILAS = [
  { rasgo: 'Color', oxidada: 'Negro idéntico en todas', natural: 'Granates, violetas, castaños' },
  { rasgo: 'Piel', oxidada: 'Lisa y tensa', natural: 'A menudo arrugada, tornasolada' },
  { rasgo: 'Textura', oxidada: 'Blanda', natural: 'Firme' },
  { rasgo: 'Sabor', oxidada: 'Dulzona, sin amargor', natural: 'Salada, ácida, amarga' },
  { rasgo: 'Hueso', oxidada: 'Casi siempre deshuesada', natural: 'Habitualmente con hueso' },
]
---
<div class="comparador">
  <table>
    <caption class="mono">Cómo distinguirlas en el plato</caption>
    <thead>
      <tr>
        <td></td>
        <th scope="col" class="camino-fabrica">
          <span class="muestra muestra-oxidada" aria-hidden="true"></span>
          <span class="aparato">Oxidada</span>
        </th>
        <th scope="col" class="camino-arbol">
          <span class="muestra muestra-natural" aria-hidden="true"></span>
          <span class="cuerpo">Negra natural</span>
        </th>
      </tr>
    </thead>
    <tbody>
      {FILAS.map((f) => (
        <tr>
          <th scope="row" class="mono">{f.rasgo}</th>
          <td class="aparato">{f.oxidada}</td>
          <td class="cuerpo">{f.natural}</td>
        </tr>
      ))}
    </tbody>
  </table>
</div>

<style>
  .comparador { margin-block: 2.4rem 3rem; overflow-x: auto; }
  table { border-collapse: collapse; width: 100%; min-width: 34rem; text-align: left; }
  caption { text-align: left; margin-bottom: 1.2rem; }
  th, td { padding: 0.9rem 1rem; border-bottom: 1px solid var(--linea); vertical-align: top; }
  thead th { border-bottom: 1px solid var(--tinta); }
  tbody th { color: var(--oliva); font-weight: 400; white-space: nowrap; }
  td:last-child, thead th:last-child { color: var(--granate); }

  .muestra {
    display: block; width: 2.2rem; height: 2.2rem; border-radius: 50%;
    margin-bottom: 0.6rem;
  }
  .camino-fabrica .muestra-oxidada { background: var(--negro); }
  .muestra-natural {
    background: radial-gradient(circle at 35% 30%, var(--terracota), var(--granate) 70%);
  }
</style>
```

`src/components/narrativa/08Reconocerlas.astro`:
```astro
---
import { PANTALLAS } from '../../content/narrativa'
import Prosa from '../ui/Prosa.astro'
import ComparadorColor from '../figuras/ComparadorColor.astro'

const p = PANTALLAS.find((x) => x.n === 8)!
---
<section class="pantalla" id={p.id} aria-labelledby={`t-${p.id}`}>
  <span class="mono">08 / Reconocerlas</span>
  <h2 id={`t-${p.id}`} class="display">{p.titulo}</h2>
  <p class="entradilla">{p.entradilla}</p>
  <ComparadorColor />
  <Prosa parrafos={p.prosa} />
</section>
```

Añadir el import y `<Reconocerlas />` en `src/pages/index.astro`.

- [ ] **Step 4: Run test to verify it passes**

Run: `npm test`
Expected: PASS — 42 tests.

- [ ] **Step 5: Commit**

```bash
git add src/components src/pages/index.astro tests/comparador.test.ts
git commit -m "feat: comparador visual de aceituna oxidada y natural"
```

---

### Task 12: Pantallas 9 y 10 — qué comprar, honestidad y bibliografía

**Files:**
- Create: `src/components/ui/Bibliografia.astro`
- Create: `src/components/narrativa/09QueComprar.astro`
- Create: `src/components/narrativa/10Honestidad.astro`
- Create: `tests/honestidad.test.ts`
- Modify: `src/pages/index.astro`

**Interfaces:**
- Consumes: `PANTALLAS`, `FUENTES`, `Prosa.astro`, `Acuarela.astro`.
- Produces: `Bibliografia.astro` con props `{ ids: FuenteId[] }`, reutilizado por las páginas de respuesta de la Tarea 14.

- [ ] **Step 1: Write the failing test**

`tests/honestidad.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { leerDist } from './helpers/dist'
import { FUENTES } from '../src/content/fuentes'

const portada = () => leerDist('index.html')

describe('honestidad y bibliografía (spec §4.1 pantalla 10, §10.5)', () => {
  it('ambas pantallas están en el HTML', () => {
    expect(portada()).toMatch(/<section[^>]*id="que-comprar"/)
    expect(portada()).toMatch(/<section[^>]*id="honestidad"/)
  })

  it('cita y enlaza el contraargumento del CSIC', () => {
    const h = portada()
    expect(h).toMatch(/Sánchez Perona/)
    expect(h).toContain(FUENTES['csic-perona'].url)
  })

  it('declara que las ilustraciones son generadas con IA', () => {
    expect(portada()).toMatch(/inteligencia artificial/i)
  })

  it('renderiza todas las fuentes citadas con enlace externo seguro', () => {
    const h = portada()
    for (const id of ['rd-679-2016', 'csic-perona', 'cicytex-acrilamida'] as const)
      expect(h, `falta ${id}`).toContain(FUENTES[id].url)
    for (const a of h.match(/<a\b[^>]*href="https:[^"]*"[^>]*>/g) ?? [])
      expect(a).toMatch(/rel="[^"]*noopener/)
  })

  it('recomienda variedades y no marcas', () => {
    const h = portada()
    expect(h).toMatch(/Empeltre/)
    for (const marca of [/mercadona/i, /carrefour/i, /hacendado/i])
      expect(h, `nombra una marca`).not.toMatch(marca)
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npm test`
Expected: FAIL — no existe la section `que-comprar`.

- [ ] **Step 3: Write minimal implementation**

`src/components/ui/Bibliografia.astro`:
```astro
---
import { FUENTES, type FuenteId } from '../../content/fuentes'

interface Props { ids: FuenteId[] }
const { ids } = Astro.props
---
<ol class="bibliografia">
  {ids.map((id) => {
    const f = FUENTES[id]
    return (
      <li class="aparato">
        <a href={f.url} rel="noopener noreferrer" target="_blank">{f.cita}</a>
        <span class="consultada">Consultada el {f.consultada}</span>
      </li>
    )
  })}
</ol>

<style>
  .bibliografia {
    margin: 2rem 0 0; padding-left: 1.4rem;
    display: grid; gap: 1.1rem; max-width: 68ch;
  }
  .bibliografia li { font-size: 0.8rem; line-height: 1.6; color: var(--tinta-70); }
  .bibliografia a { color: var(--tinta); text-decoration-color: var(--linea); }
  .bibliografia a:hover { color: var(--granate); }
  .consultada { display: block; color: var(--tinta-45); margin-top: 0.2rem; }
</style>
```

`src/components/narrativa/09QueComprar.astro`:
```astro
---
import { PANTALLAS } from '../../content/narrativa'
import Prosa from '../ui/Prosa.astro'
import Acuarela from '../ui/Acuarela.astro'
import manos from '../../assets/acuarelas/06-manos-cesta.jpg'

const p = PANTALLAS.find((x) => x.n === 9)!
const VARIEDADES = [
  { nombre: 'Empeltre', alias: 'negra de Aragón', nota: 'Curada en seco hasta arrugarse. Dulce y mantecosa.' },
  { nombre: 'Kalamata', alias: 'Grecia', nota: 'Morada, firme, punzante. En salmuera o vinagre.' },
  { nombre: 'Cuquillo', alias: 'Andalucía', nota: 'Pequeña, oscura, muy aromática.' },
]
---
<section class="pantalla" id="que-comprar" aria-labelledby="t-que-comprar">
  <span class="mono">09 / Qué comprar</span>
  <h2 id="t-que-comprar" class="display">{p.titulo}</h2>
  <p class="entradilla">{p.entradilla}</p>

  <ul class="variedades">
    {VARIEDADES.map((v) => (
      <li>
        <b class="cuerpo">{v.nombre}</b>
        <span class="alias mono">{v.alias}</span>
        <p class="cuerpo">{v.nota}</p>
      </li>
    ))}
  </ul>

  <Acuarela src={manos} alt="Acuarela de unas manos recogiendo aceitunas en una cesta" />
  <Prosa parrafos={p.prosa} />
</section>

<style>
  .variedades {
    list-style: none; padding: 0; margin: 2.2rem 0 3rem;
    display: grid; grid-template-columns: repeat(auto-fit, minmax(15rem, 1fr)); gap: 1.6rem;
  }
  .variedades li { border-top: 1px solid var(--linea); padding-top: 1rem; }
  .variedades b { display: block; font-size: 1.3rem; color: var(--granate); }
  .alias { display: block; margin: 0.3rem 0 0.7rem; }
  .variedades p { margin: 0; color: var(--tinta-70); font-size: 1rem; }
</style>
```

`src/components/narrativa/10Honestidad.astro`:
```astro
---
import { PANTALLAS } from '../../content/narrativa'
import Prosa from '../ui/Prosa.astro'
import Bibliografia from '../ui/Bibliografia.astro'

const p = PANTALLAS.find((x) => x.n === 10)!
const RESPUESTAS = [
  { href: '/aceitunas-negras-oxidadas/', texto: 'Cómo se fabrica una aceituna negra' },
  { href: '/e-579-gluconato-ferroso/', texto: 'Qué es el E-579 y qué hace ahí' },
  { href: '/aceitunas-negras-naturales/', texto: 'Cuáles son negras de verdad' },
  { href: '/aceitunas-verdes-y-negras/', texto: 'Verdes y negras: la diferencia real' },
]
---
<section class="pantalla honestidad" id={p.id} aria-labelledby={`t-${p.id}`}>
  <span class="mono">10 / Lo que no te estoy diciendo</span>
  <h2 id={`t-${p.id}`} class="display">{p.titulo}</h2>
  <p class="entradilla">{p.entradilla}</p>

  <Prosa parrafos={p.prosa} />

  <h3 class="mono">Fuentes</h3>
  <Bibliografia ids={p.fuentes ?? []} />

  <h3 class="mono">Seguir leyendo</h3>
  <ul class="respuestas">
    {RESPUESTAS.map((r) => (
      <li><a class="cuerpo" href={r.href}>{r.texto}</a></li>
    ))}
  </ul>
</section>

<style>
  .honestidad { border-top: 1px solid var(--linea); }
  .honestidad h3 { margin: 3rem 0 0; }
  .respuestas { list-style: none; padding: 0; margin: 1.4rem 0 0; display: grid; gap: 0.8rem; }
  .respuestas a { color: var(--tinta); text-decoration-color: var(--linea); }
  .respuestas a:hover { color: var(--granate); }
</style>
```

Añadir los imports y `<QueComprar />` y `<Honestidad />` al final de
`src/pages/index.astro`.

- [ ] **Step 4: Run test to verify it passes**

Run: `npm test`
Expected: PASS — 47 tests.

- [ ] **Step 5: Commit**

```bash
git add src/components src/pages/index.astro tests/honestidad.test.ts
git commit -m "feat: pantallas de qué comprar y honestidad con bibliografía renderizada"
```

---

### Task 13: Movimiento ligado al scroll y accesibilidad

**Files:**
- Create: `src/styles/movimiento.css`
- Create: `tests/movimiento.test.ts`
- Create: `playwright.config.ts`
- Create: `tests/e2e/accesibilidad.spec.ts`
- Modify: `src/layouts/Narrativa.astro`
- Modify: `package.json`

**Interfaces:**
- Consumes: las clases `.pantalla` y `.acuarela` de las tareas anteriores.
- Produces: las clases de animación `.aparece` y `.deriva`, aplicables a cualquier elemento.

- [ ] **Step 1: Write the failing test**

`tests/movimiento.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { readFileSync, readdirSync } from 'node:fs'
import { join } from 'node:path'

const ESTILOS = join(process.cwd(), 'src/styles')
const movimiento = () => readFileSync(join(ESTILOS, 'movimiento.css'), 'utf8')
const todoElCss = () =>
  readdirSync(ESTILOS)
    .filter((f) => f.endsWith('.css'))
    .map((f) => ({ f, texto: readFileSync(join(ESTILOS, f), 'utf8') }))

describe('movimiento (spec §6)', () => {
  it('usa animation-timeline y va dentro de un @supports', () => {
    const m = movimiento()
    expect(m).toMatch(/animation-timeline:\s*view\(\)/)
    expect(m).toMatch(/@supports\s*\(animation-timeline:\s*view\(\)\)/)
  })

  it('respeta prefers-reduced-motion', () => {
    expect(movimiento()).toMatch(/@media\s*\(prefers-reduced-motion:\s*reduce\)/)
  })

  it('ninguna animación esconde texto del DOM', () => {
    for (const { f, texto } of todoElCss()) {
      expect(texto, `${f} usa display:none`).not.toMatch(/display:\s*none/)
      expect(texto, `${f} usa content-visibility:hidden`).not.toMatch(
        /content-visibility:\s*hidden/,
      )
      expect(texto, `${f} usa visibility:hidden`).not.toMatch(/visibility:\s*hidden/)
    }
  })

  it('solo anima opacity, transform, translate, scale y color', () => {
    const permitidas = /^(opacity|transform|translate|scale|color)$/
    for (const bloque of movimiento().match(/@keyframes[^{]*\{[\s\S]*?\n\s*\}/g) ?? [])
      for (const prop of bloque.match(/^\s*([a-z-]+):/gm) ?? []) {
        const nombre = prop.trim().replace(':', '')
        expect(nombre, `anima ${nombre}`).toMatch(permitidas)
      }
  })
})
```

`playwright.config.ts`:
```ts
import { defineConfig } from '@playwright/test'

export default defineConfig({
  testDir: './tests/e2e',
  webServer: {
    command: 'npm run preview',
    url: 'http://localhost:4321',
    reuseExistingServer: !process.env.CI,
  },
  use: { baseURL: 'http://localhost:4321' },
})
```

`tests/e2e/accesibilidad.spec.ts`:
```ts
import { test, expect } from '@playwright/test'
import AxeBuilder from '@axe-core/playwright'

const RUTAS = [
  '/',
  '/aceitunas-negras-oxidadas/',
  '/e-579-gluconato-ferroso/',
  '/aceitunas-negras-naturales/',
  '/aceitunas-verdes-y-negras/',
]

for (const ruta of RUTAS) {
  test(`sin infracciones de accesibilidad en ${ruta}`, async ({ page }) => {
    await page.goto(ruta)
    const { violations } = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze()
    expect(violations.map((v) => `${v.id}: ${v.help}`)).toEqual([])
  })
}

test('todo el texto de las diez pantallas está en el DOM al cargar', async ({ page }) => {
  await page.goto('/')
  for (const id of [
    'apertura', 'promesa', 'calendario', 'bifurcacion', 'fabrica',
    'arbol', 'prueba', 'reconocerlas', 'que-comprar', 'honestidad',
  ]) {
    const texto = await page.locator(`#${id}`).innerText()
    expect(texto.trim().length, `${id} vacío`).toBeGreaterThan(20)
  }
})

test('con movimiento reducido no queda ninguna animación activa', async ({ browser }) => {
  const ctx = await browser.newContext({ reducedMotion: 'reduce' })
  const page = await ctx.newPage()
  await page.goto('/')
  const animadas = await page.evaluate(
    () =>
      document.getAnimations().filter((a) => a.playState === 'running').length,
  )
  expect(animadas).toBe(0)
  await ctx.close()
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npx vitest run tests/movimiento.test.ts`
Expected: FAIL — `ENOENT ... src/styles/movimiento.css`.

- [ ] **Step 3: Write minimal implementation**

`src/styles/movimiento.css`:
```css
@layer movimiento {
  /* Estado final por defecto: sin soporte, sin JS y sin animación, todo visible. */
  @keyframes aparece {
    from { opacity: 0; translate: 0 1.5rem; }
    to   { opacity: 1; translate: 0 0; }
  }

  @keyframes deriva {
    from { scale: 1.04; }
    to   { scale: 1; }
  }

  @supports (animation-timeline: view()) {
    @media (prefers-reduced-motion: no-preference) {
      .aparece {
        animation: aparece linear both;
        animation-timeline: view();
        animation-range: entry 10% entry 60%;
      }

      .deriva {
        animation: deriva linear both;
        animation-timeline: view();
        animation-range: entry 0% cover 100%;
      }
    }
  }

  @media (prefers-reduced-motion: reduce) {
    .aparece, .deriva { animation: none; }
  }
}
```

Aplicar las clases. En `src/layouts/Narrativa.astro`, añadir el import
`import '../styles/movimiento.css'` y, al final del `<body>`, dentro de `<main>`,
nada más: las clases se añaden en `src/styles/narrativa.css`, para no repetirlas
en diez componentes. Añadir a `src/styles/narrativa.css`:

```css
@layer narrativa {
  .pantalla > .display,
  .pantalla > .entradilla,
  .pantalla > .prosa { animation-name: aparece; }
}
```

Y en el mismo fichero, para que solo se animen cuando procede, envolver esa regla:

```css
@layer narrativa {
  @supports (animation-timeline: view()) {
    @media (prefers-reduced-motion: no-preference) {
      .pantalla > .display,
      .pantalla > .entradilla,
      .pantalla > .prosa {
        animation: aparece linear both;
        animation-timeline: view();
        animation-range: entry 10% entry 60%;
      }
      .acuarela :global(img) {
        animation: deriva linear both;
        animation-timeline: view();
        animation-range: entry 0% cover 100%;
      }
    }
  }
}
```

Añadir las dependencias y los scripts de Playwright:
```bash
npm i -D @playwright/test @axe-core/playwright
npx playwright install chromium
```
```json
    "test:a11y": "astro build && playwright test",
```

- [ ] **Step 4: Run test to verify it passes**

Run: `npm test && npm run test:a11y`
Expected: `npm test` PASS — 51 tests. `test:a11y` fallará en las cuatro rutas de
respuesta hasta la Tarea 14; las tres pruebas de `/` deben pasar.

- [ ] **Step 5: Commit**

```bash
git add src/styles src/layouts playwright.config.ts tests package.json package-lock.json
git commit -m "feat: movimiento ligado al scroll sin javascript y auditoría de accesibilidad"
```

---

### Task 14: Páginas de respuesta, sitemap y robots

**Files:**
- Create: `src/content.config.ts`
- Create: `src/content/respuestas/aceitunas-negras-oxidadas.md`
- Create: `src/content/respuestas/e-579-gluconato-ferroso.md`
- Create: `src/content/respuestas/aceitunas-negras-naturales.md`
- Create: `src/content/respuestas/aceitunas-verdes-y-negras.md`
- Create: `src/layouts/Respuesta.astro`
- Create: `src/pages/[respuesta].astro`
- Create: `public/robots.txt`
- Create: `tests/respuestas.test.ts`

**Interfaces:**
- Consumes: `Bibliografia.astro` (Tarea 12), los estilos de las Tareas 2, 3 y 5.
- Produces: la colección `respuestas`, con el esquema `{ titulo, descripcion, preguntas: {q, a}[], fuentes: FuenteId[] }`.

- [ ] **Step 1: Write the failing test**

`tests/respuestas.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { leerDist, listarDist } from './helpers/dist'

const RUTAS = [
  'aceitunas-negras-oxidadas',
  'e-579-gluconato-ferroso',
  'aceitunas-negras-naturales',
  'aceitunas-verdes-y-negras',
]

describe('páginas de respuesta (spec §4.2, §9)', () => {
  it('las cuatro se generan', () => {
    const html = listarDist('.html')
    for (const r of RUTAS) expect(html, `falta ${r}`).toContain(`${r}/index.html`)
  })

  it('cada una tiene título y descripción propios y distintos', () => {
    const titulos = RUTAS.map(
      (r) => leerDist(`${r}/index.html`).match(/<title>([^<]*)<\/title>/)![1],
    )
    expect(new Set(titulos).size).toBe(4)
    for (const t of titulos) expect(t.length).toBeGreaterThan(15)
  })

  it('cada una lleva un JSON-LD de tipo FAQPage con preguntas', () => {
    for (const r of RUTAS) {
      const m = leerDist(`${r}/index.html`).match(
        /<script type="application\/ld\+json">([\s\S]*?)<\/script>/,
      )
      const datos = JSON.parse(m![1])
      expect(datos['@type'], r).toBe('FAQPage')
      expect(datos.mainEntity.length, r).toBeGreaterThan(0)
    }
  })

  it('cada una enlaza de vuelta a la narrativa', () => {
    for (const r of RUTAS)
      expect(leerDist(`${r}/index.html`), r).toMatch(/href="\/"/)
  })

  it('no llevan animación: son páginas sobrias', () => {
    for (const r of RUTAS)
      expect(leerDist(`${r}/index.html`), r).not.toMatch(/class="[^"]*aparece/)
  })

  it('el sitemap incluye las cinco páginas', () => {
    const s = leerDist('sitemap-0.xml')
    expect(s).toMatch(/<loc>https:\/\/nuncafuenegra\.com\/<\/loc>/)
    for (const r of RUTAS) expect(s, r).toContain(`/${r}/`)
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npm test`
Expected: FAIL — «falta aceitunas-negras-oxidadas».

- [ ] **Step 3: Write minimal implementation**

`src/content.config.ts`:
```ts
import { defineCollection, z } from 'astro:content'
import { glob } from 'astro/loaders'

export const collections = {
  respuestas: defineCollection({
    loader: glob({ pattern: '**/*.md', base: './src/content/respuestas' }),
    schema: z.object({
      titulo: z.string(),
      descripcion: z.string().min(50).max(160),
      preguntas: z.array(z.object({ q: z.string(), a: z.string() })).min(1),
      fuentes: z.array(z.string()),
    }),
  }),
}
```

`src/layouts/Respuesta.astro`:
```astro
---
import '../styles/tokens.css'
import '../styles/base.css'
import '../styles/tipografia.css'
import '../styles/narrativa.css'
import Bibliografia from '../components/ui/Bibliografia.astro'
import type { FuenteId } from '../content/fuentes'

interface Props {
  titulo: string
  descripcion: string
  canonical: string
  preguntas: { q: string; a: string }[]
  fuentes: FuenteId[]
}
const { titulo, descripcion, canonical, preguntas, fuentes } = Astro.props

const jsonLd = {
  '@context': 'https://schema.org',
  '@type': 'FAQPage',
  inLanguage: 'es',
  mainEntity: preguntas.map((p) => ({
    '@type': 'Question',
    name: p.q,
    acceptedAnswer: { '@type': 'Answer', text: p.a },
  })),
}
---
<html lang="es">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="color-scheme" content="light only" />
    <link rel="preload" href="/fonts/newsreader-latin-ext.woff2" as="font" type="font/woff2" crossorigin />
    <title>{titulo}</title>
    <meta name="description" content={descripcion} />
    <link rel="canonical" href={canonical} />
    <meta property="og:type" content="article" />
    <meta property="og:title" content={titulo} />
    <meta property="og:description" content={descripcion} />
    <meta property="og:image" content={new URL('/og.jpg', Astro.site).href} />
    <meta name="twitter:card" content="summary_large_image" />
    <script type="application/ld+json" set:html={JSON.stringify(jsonLd)} />
  </head>
  <body>
    <main class="respuesta">
      <article>
        <slot />
      </article>

      <h2 class="mono">Preguntas frecuentes</h2>
      <dl class="faq">
        {preguntas.map((p) => (
          <div>
            <dt class="cuerpo">{p.q}</dt>
            <dd class="cuerpo">{p.a}</dd>
          </div>
        ))}
      </dl>

      <h2 class="mono">Fuentes</h2>
      <Bibliografia ids={fuentes} />

      <p class="volver"><a class="cuerpo" href="/">Volver a la historia completa</a></p>
    </main>
  </body>
</html>

<style>
  .respuesta { max-width: 42rem; margin-inline: auto; padding: 4rem 6vw 6rem; }
  .respuesta :global(h1) {
    font-family: 'Fraunces', Georgia, serif;
    font-variation-settings: 'opsz' 90, 'SOFT' 45, 'WONK' 0;
    font-weight: 400; font-size: clamp(2rem, 5.5vw, 3.2rem);
    line-height: 1.05; letter-spacing: -0.02em; margin: 0 0 1.6rem;
  }
  .respuesta :global(h2) { margin-top: 3rem; }
  .respuesta :global(article h2) {
    font-family: 'Newsreader', Georgia, serif; font-size: 1.5rem;
    letter-spacing: 0; text-transform: none; color: var(--granate);
  }
  .respuesta :global(p) { line-height: 1.7; }
  .respuesta :global(a) { color: var(--tinta); text-decoration-color: var(--linea); }
  .respuesta :global(a:hover) { color: var(--granate); }

  .faq { margin: 1.4rem 0 0; }
  .faq dt { color: var(--granate); margin-top: 1.4rem; }
  .faq dd { margin: 0.4rem 0 0; color: var(--tinta-70); }

  .volver { margin-top: 3.5rem; border-top: 1px solid var(--linea); padding-top: 1.5rem; }
</style>
```

`src/pages/[respuesta].astro`:
```astro
---
import { getCollection, render } from 'astro:content'
import Respuesta from '../layouts/Respuesta.astro'
import type { FuenteId } from '../content/fuentes'

export async function getStaticPaths() {
  const entradas = await getCollection('respuestas')
  return entradas.map((entrada) => ({
    params: { respuesta: entrada.id },
    props: { entrada },
  }))
}

const { entrada } = Astro.props
const { Content } = await render(entrada)
const d = entrada.data
---
<Respuesta
  titulo={d.titulo}
  descripcion={d.descripcion}
  canonical={`https://nuncafuenegra.com/${entrada.id}/`}
  preguntas={d.preguntas}
  fuentes={d.fuentes as FuenteId[]}
>
  <Content />
</Respuesta>
```

`public/robots.txt`:
```
User-agent: *
Allow: /

Sitemap: https://nuncafuenegra.com/sitemap-index.xml
```

`src/content/respuestas/aceitunas-negras-oxidadas.md`:
```markdown
---
titulo: 'Aceitunas negras oxidadas: cómo se fabrica una aceituna negra'
descripcion: 'Las aceitunas negras de lata se recogen sin madurar y se ennegrecen por oxidación en medio alcalino. Así es el proceso, paso a paso y sin exageraciones.'
preguntas:
  - q: '¿Las aceitunas negras están teñidas?'
    a: 'No. No se les añade colorante. El negro lo forman los compuestos fenólicos de la propia aceituna al oxidarse en medio alcalino. Lo que sí se añade al final es una sal de hierro que hace que ese negro salga uniforme e intenso.'
  - q: '¿Por qué se recogen verdes?'
    a: 'Porque el color se va a conseguir en la fábrica, no en el árbol. Eso permite adelantar la recogida y obtener un producto de color idéntico en todas las unidades.'
  - q: '¿Es legal?'
    a: 'Sí. Es una categoría reconocida por la norma española de calidad de las aceitunas de mesa, que la llama simplemente «negras».'
fuentes: ['rd-679-2016', 'csic-perona', 'aditivos-1129-2011']
---

# Cómo se fabrica una aceituna negra

Una aceituna negra de lata casi nunca es una aceituna que madurara en el árbol.
Es un fruto recogido antes de madurar al que se le ha provocado el color. La
norma española lo dice sin rodeos: son «las obtenidas de frutos que no estando
totalmente maduros, han sido oscurecidos mediante oxidación».

## El proceso

El fruto se recoge sin madurar y se conserva en salmuera, fermentada o no.
Después recibe tratamientos alcalinos sucesivos, y entre uno y otro se le inyecta
aire. La sosa le quita el amargor y le ablanda la carne. El aire hace que los
compuestos fenólicos de la propia aceituna se oxiden y formen pigmentos oscuros.

## Aquí está el detalle importante

El resultado de ese proceso no es negro: es marrón muy oscuro. Para conseguir el
negro uniforme que se espera de una lata se añade una sal de hierro, gluconato
ferroso (E-579) o lactato ferroso (E-585), que se acopla a esos mismos fenoles.

Los investigadores del CSIC que mejor lo han descrito señalan que ese paso es
opcional y que se adopta por atractivo comercial. Es decir: se añade un aditivo
porque el color honesto no es lo bastante negro para venderse como negro.

Después llegan el deshuesado, el envasado y la esterilización térmica.

## Lo que no es

No es un fraude: la etiqueta cumple la norma. No es un veneno. Y no es lo mismo
que una aceituna negra natural, que es otro producto, con otro sabor y otro
nombre legal.
```

`src/content/respuestas/e-579-gluconato-ferroso.md`:
```markdown
---
titulo: 'E-579 (gluconato ferroso): qué es y por qué está en tus aceitunas'
descripcion: 'El E-579 es gluconato ferroso, una sal de hierro autorizada solo en aceitunas ennegrecidas por oxidación. No es un colorante: fija y uniformiza el negro.'
preguntas:
  - q: '¿Qué es el E-579?'
    a: 'Gluconato ferroso, una sal de hierro. En la Unión Europea está autorizada como estabilizante del color en aceitunas ennegrecidas por oxidación, con un máximo de 150 mg/kg expresado en hierro.'
  - q: '¿El E-579 es un colorante?'
    a: 'No. No aporta color propio. Forma complejos con los compuestos fenólicos de la propia aceituna, y eso convierte el marrón oscuro de la oxidación en un negro uniforme e intenso.'
  - q: '¿Es peligroso?'
    a: 'No hay motivo para pensarlo en las cantidades autorizadas. Una ración normal de aceitunas se queda muy lejos de cualquier umbral de preocupación por hierro. Si algo conviene vigilar en una aceituna es la sal.'
  - q: '¿Y el E-585?'
    a: 'Es lactato ferroso, otra sal de hierro con la misma función. Si ves cualquiera de los dos en la lista de ingredientes, la aceituna es oxidada.'
fuentes: ['aditivos-1129-2011', 'csic-perona', 'rd-679-2016']
---

# Qué es el E-579 y qué hace en una lata de aceitunas

El E-579 es gluconato ferroso: una sal de hierro. Aparece en la lista de
ingredientes de casi cualquier lata de aceitunas negras, y es la razón por la que
esas aceitunas son de un negro tan parejo.

## No colorea. Uniformiza

Es el malentendido más repetido de todo este asunto. El E-579 no es un tinte y no
aporta color propio. Lo que hace es acoplarse a los compuestos fenólicos que la
aceituna ya tiene, y el resultado de esa unión es un negro intenso y homogéneo.

Sin ese paso, la aceituna oxidada saldría marrón muy oscuro, y desigual.

## Por qué se usa

Porque el mercado espera que una aceituna negra sea negra. El paso es opcional, y
se generalizó por atractivo comercial.

## Cómo usarlo para leer una etiqueta

Si ves E-579 o E-585, es oxidada. Pero hay una comprobación mejor y más rápida:
mira la denominación en la cara del envase y busca la palabra «naturales». Si no
está, ya lo sabes sin leer la letra pequeña.
```

`src/content/respuestas/aceitunas-negras-naturales.md`:
```markdown
---
titulo: 'Aceitunas negras naturales: cuáles son de verdad y cómo se llaman'
descripcion: 'Negras naturales es una categoría legal distinta de negras: fruto recogido en plena madurez. Variedades, cómo pedirlas y en qué se nota.'
preguntas:
  - q: '¿Qué significa «aceitunas negras naturales»?'
    a: 'Es una categoría legal propia: aceitunas obtenidas de frutos recogidos en plena madurez o poco antes. La norma española la distingue de las «negras», que son las oscurecidas mediante oxidación.'
  - q: '¿Qué variedades son negras naturales?'
    a: 'La Empeltre o negra de Aragón, curada en seco hasta arrugarse; la kalamata griega; y el cuquillo andaluz, entre otras. Lo importante no es memorizarlas, sino que el envase nombre la variedad.'
  - q: '¿Cómo distingo una negra natural en el plato?'
    a: 'Por el desorden. Dentro del mismo tarro hay granates, violetas y castaños, muchas están arrugadas, la carne es firme y el sabor lleva sal, acidez y un amargor de fondo.'
fuentes: ['rd-679-2016', 'mapa-aceituna', 'csic-perona']
---

# Cuáles son negras de verdad

«Negras naturales» no es una expresión de marketing: es una categoría de la norma
española de calidad. Son las obtenidas de frutos recogidos en plena madurez o
poco antes, y la propia norma describe su color de una forma que ya lo explica
todo: negro rojizo, negro violáceo, violeta, negro verdoso o castaño oscuro.

Cinco maneras de no ser negro.

## Cómo se elaboran

Con espera y poco más. Salmuera, donde fermentan despacio; o sal seca, que las
deshidrata y las arruga; y a veces hierbas, ajo, hinojo, tomillo o cáscara de
naranja. La norma llama «aliñado» a eso, y consiste literalmente en añadir
condimentos o especias.

## Variedades que merece la pena buscar

La **Empeltre**, la negra de Aragón, curada en seco hasta quedar arrugada, dulce y
mantecosa. La **kalamata** griega, morada, firme y punzante. El **cuquillo**
andaluz, pequeño y aromático.

Y una regla que funciona mejor que cualquier lista: fíate de quien te vende la
aceituna por su variedad y no por su color. La norma permite indicar la variedad
de forma voluntaria; que alguien se moleste en ponerlo ya es una señal.

## Un aviso justo

Casi la mitad de la aceituna de mesa española es Hojiblanca, y otra buena parte
Manzanilla. Son variedades excelentes, y muchas acaban oxidadas porque es lo que
se vende. El problema no es la aceituna, ni quien la cultiva.
```

`src/content/respuestas/aceitunas-verdes-y-negras.md`:
```markdown
---
titulo: 'Aceitunas verdes y negras: ¿son la misma aceituna?'
descripcion: 'A veces sí y a veces no. La diferencia real entre verdes y negras no es la variedad, sino el momento de la recogida y lo que pasa después.'
preguntas:
  - q: '¿Las aceitunas verdes y negras son la misma aceituna?'
    a: 'Pueden serlo. Una misma variedad da aceitunas verdes si se recoge antes del envero y negras naturales si se deja madurar. Pero una aceituna negra de lata suele ser fruto recogido verde y oscurecido después por oxidación.'
  - q: '¿Cuál es más sana?'
    a: 'Las dos son alimentos sanos. El factor que más conviene vigilar en una aceituna de mesa es la sal, y eso vale para verdes y negras.'
  - q: '¿Por qué la negra de lata sabe más suave?'
    a: 'Porque el proceso de oxidación deja un producto casi neutro, poco salado y sin amargor. La negra natural, en cambio, es ácida, salada y amarga.'
fuentes: ['rd-679-2016', 'csic-perona']
---

# ¿Son la misma aceituna?

A veces sí. La norma española clasifica las aceitunas por su color, y ese color es
básicamente una fecha: verdes, las recogidas antes del envero; de color
cambiante, las recogidas durante el envero; negras naturales, las recogidas en
plena madurez.

Una misma variedad puede dar las tres cosas. Solo cambia el día de la recogida.

## Y a veces no

Porque hay un cuarto tipo, y es el que llena las latas: «negras», sin más. Son
frutos que no estaban maduros y que se han oscurecido mediante oxidación. No
maduraron: se les provocó el color.

## En qué se nota

La verde es firme y salada. La negra natural es ácida, salada y con un amargor de
fondo. La negra oxidada es blanda, dulzona y casi neutra, y es la más fácil de
comer para casi todo el mundo. Precisamente por eso se vende tanto.

## La comprobación rápida

Mira la denominación del envase y busca la palabra «naturales». Si no aparece, la
aceituna es oxidada, y la norma no obliga a decírtelo de otra manera.
```

- [ ] **Step 4: Run test to verify it passes**

Run: `npm test && npm run test:a11y`
Expected: PASS — 57 tests en Vitest, y las cinco rutas sin infracciones en Playwright.

- [ ] **Step 5: Commit**

```bash
git add src/content.config.ts src/content/respuestas src/layouts/Respuesta.astro src/pages public/robots.txt tests/respuestas.test.ts
git commit -m "feat: cuatro páginas de respuesta con datos estructurados y sitemap"
```

---

### Task 15: Imagen social, presupuesto final y despliegue

**Files:**
- Create: `scripts/og.mjs`
- Create: `public/og.jpg` (generado)
- Create: `tests/presupuesto-final.test.ts`
- Create: `public/_headers`
- Create: `README.md`
- Modify: `package.json`

**Interfaces:**
- Consumes: `src/assets/acuarelas/01-aceituna-negra.jpg`, los helpers de la Tarea 1.
- Produces: `public/og.jpg` a 1200×630, ya referenciada por los layouts de las Tareas 5 y 14.

- [ ] **Step 1: Write the failing test**

`tests/presupuesto-final.test.ts`:
```ts
import { describe, it, expect } from 'vitest'
import { listarDist, pesoDist, leerDist, htmlDeDist } from './helpers/dist'

const kb = (n: number) => Math.round(n / 1024)

describe('presupuesto final (spec §8)', () => {
  it('la portada, con su CSS inline, pesa menos de 100 KB', () => {
    const peso = pesoDist('index.html')
    expect(kb(peso), `index.html pesa ${kb(peso)} KB`).toBeLessThan(100)
  })

  it('el CSS va inline: no hay hojas de estilo externas', () => {
    for (const { ruta, html } of htmlDeDist())
      expect(html, `${ruta} enlaza un CSS externo`).not.toMatch(
        /<link[^>]*rel="stylesheet"/,
      )
  })

  it('ninguna variante de imagen pasa de 180 KB', () => {
    for (const ext of ['.avif', '.webp', '.jpg'])
      for (const f of listarDist(ext))
        expect(kb(pesoDist(f)), `${f} pesa ${kb(pesoDist(f))} KB`).toBeLessThan(180)
  })

  it('la imagen social existe y pesa menos de 200 KB', () => {
    expect(kb(pesoDist('og.jpg'))).toBeLessThan(200)
  })

  it('el primer render pide como mucho seis recursos', () => {
    const html = leerDist('index.html')
    const preloads = (html.match(/rel="preload"/g) ?? []).length
    const eager = (html.match(/loading="eager"/g) ?? []).length
    // 1 documento + fuentes precargadas + imágenes no diferidas
    expect(1 + preloads + eager).toBeLessThanOrEqual(6)
  })

  it('sigue sin haber ni un byte de JavaScript', () => {
    expect(listarDist('.js')).toEqual([])
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npm test`
Expected: FAIL — `ENOENT ... dist/og.jpg`.

- [ ] **Step 3: Write minimal implementation**

`scripts/og.mjs`. Compone la acuarela de apertura sobre el arena de la paleta y
le superpone el titular como SVG con la fuente incrustada en base64, para que
`sharp` la rasterice con la tipografía real:

```js
import sharp from 'sharp'
import { readFileSync, mkdirSync } from 'node:fs'
import { join } from 'node:path'

const RAIZ = process.cwd()
const ANCHO = 1200
const ALTO = 630
const PAPEL = '#EDE4D4'
const TINTA = '#2B2318'

const fuente = readFileSync(
  join(RAIZ, 'public/fonts/fraunces-latin-ext.woff2'),
).toString('base64')

const svg = `
<svg xmlns="http://www.w3.org/2000/svg" width="${ANCHO}" height="${ALTO}">
  <defs>
    <style>
      @font-face {
        font-family: 'Fraunces';
        src: url(data:font/woff2;base64,${fuente}) format('woff2');
      }
      .titular {
        font-family: 'Fraunces', Georgia, serif;
        font-size: 86px;
        fill: ${TINTA};
        font-variation-settings: 'opsz' 144, 'SOFT' 45, 'WONK' 1;
      }
      .pie {
        font-family: Georgia, serif;
        font-size: 26px;
        fill: #5C4F3C;
        letter-spacing: 2px;
      }
    </style>
  </defs>
  <text class="titular" x="72" y="250">Esta aceituna</text>
  <text class="titular" x="72" y="350">nunca estuvo</text>
  <text class="titular" x="72" y="450">negra.</text>
  <text class="pie" x="72" y="558">nuncafuenegra.com</text>
</svg>`

mkdirSync(join(RAIZ, 'public'), { recursive: true })

const acuarela = await sharp(
  join(RAIZ, 'src/assets/acuarelas/01-aceituna-negra.jpg'),
)
  .resize({ width: 520, height: ALTO, fit: 'cover' })
  .toBuffer()

await sharp({
  create: { width: ANCHO, height: ALTO, channels: 3, background: PAPEL },
})
  .composite([
    { input: acuarela, left: ANCHO - 520, top: 0, blend: 'multiply' },
    { input: Buffer.from(svg), top: 0, left: 0 },
  ])
  .jpeg({ quality: 84, mozjpeg: true })
  .toFile(join(RAIZ, 'public/og.jpg'))

console.log('✓ public/og.jpg 1200×630')
```

Actualizar los scripts de `package.json` para que `og` corra antes del build:
```json
    "og": "node scripts/og.mjs",
    "prebuild": "node scripts/og.mjs",
```

`public/_headers` — caché inmutable para lo versionado, en Cloudflare Pages:
```
/_astro/*
  Cache-Control: public, max-age=31536000, immutable

/fonts/*
  Cache-Control: public, max-age=31536000, immutable

/*
  Cache-Control: public, max-age=0, must-revalidate
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
```

`README.md`:
```markdown
# nuncafuenegra.com

Sitio estático en español sobre por qué las «aceitunas negras» de lata son fruto
sin madurar ennegrecido por oxidación, y no «aceitunas negras naturales».

- **Diseño:** `docs/superpowers/specs/2026-09-08-nuncafuenegra-design.md`
- **Plan:** `docs/superpowers/plans/2026-09-08-nuncafuenegra.md`

## Desarrollo

```bash
npm install
npm run acuarelas   # genera sustitutas provisionales si no hay ilustraciones
npm run dev
```

## Comandos

| Comando | Qué hace |
|---|---|
| `npm run dev` | servidor de desarrollo |
| `npm run build` | build estático a `dist/` (genera antes `public/og.jpg`) |
| `npm test` | build + los guardianes del spec en Vitest |
| `npm run test:a11y` | build + auditoría de accesibilidad con Playwright y axe |
| `npm run acuarelas` | genera acuarelas provisionales que aún no existan |

## Las ilustraciones

Las diez acuarelas van en `src/assets/acuarelas/` con los nombres que documenta el
plan. Se generan con IA siguiendo el ancla de estilo del spec §5.4, sobre **papel
blanco**: el sitio las integra con `mix-blend-mode: multiply`. Sustituir un fichero
no requiere ningún cambio de código.

## Reglas que los tests hacen cumplir

- Cero JavaScript enviado al cliente.
- `#000` solo en el camino de la fábrica.
- Prohibidas las palabras «teñidas», «colorante» y «fraude».
- Toda afirmación referencia una fuente de `src/content/fuentes.ts`.
- Ninguna animación esconde texto del DOM.

## Despliegue

Cloudflare Pages. Comando de build `npm run build`, directorio de salida `dist`.
```

- [ ] **Step 4: Run test to verify it passes**

Run: `npm test && npm run test:a11y`
Expected: PASS — 63 tests en Vitest y las cinco rutas limpias en Playwright.

Si el test de «seis recursos» falla, quitar el `preload` de Newsreader: con
Fraunces precargada y el CSS inline, el LCP lo marca el titular.

- [ ] **Step 5: Commit**

```bash
git add scripts public README.md package.json tests/presupuesto-final.test.ts
git commit -m "feat: imagen social generada, cabeceras de caché y presupuesto final verificado"
```

---

### Task 16: Cierre documental

Última tarea: registrar el porqué de las decisiones ya tomadas, poner las
cabeceras en los ficheros que protegen reglas, y añadir a `AGENTS.md` el mapa del
repo, que hasta ahora no existía.

**Files:**
- Create: `docs/DECISIONES.md`
- Modify: `AGENTS.md`
- Modify: `src/content/fuentes.ts`, `src/content/narrativa.ts`, `src/styles/tokens.css`, `src/components/ui/Acuarela.astro` (cabeceras)
- Modify: `tests/documentacion.test.ts`

**Interfaces:**
- Consumes: todo lo construido en las tareas 1 a 15.
- Produces: nada que consuma otra tarea. Es el cierre.

- [ ] **Step 1: Write the failing test**

Añadir a `tests/documentacion.test.ts`:
```ts
describe('cierre documental (spec §13)', () => {
  const CABECERAS: [string, RegExp][] = [
    ['src/content/fuentes.ts', /copy\.test\.ts/],
    ['src/content/narrativa.ts', /copy\.test\.ts/],
    ['src/styles/tokens.css', /paleta\.test\.ts/],
    ['src/components/ui/Acuarela.astro', /multiply/],
  ]

  it('existe el registro de decisiones', () => {
    expect(existsSync(raiz('docs/DECISIONES.md'))).toBe(true)
  })

  it('cada decisión registrada explica su motivo', () => {
    const bloques = leer('docs/DECISIONES.md').split(/^## /m).slice(1)
    expect(bloques.length, 'muy pocas decisiones').toBeGreaterThanOrEqual(6)
    for (const b of bloques)
      expect(b, `sin motivo: ${b.slice(0, 40)}`).toMatch(/\*\*Por qué:\*\*/)
  })

  it('registra el reencuadre editorial, que es el que se deshace solo', () => {
    const d = leer('docs/DECISIONES.md')
    expect(d).toMatch(/CSIC/)
    expect(d).toMatch(/12\.3\.b/)
  })

  it('los ficheros que protegen una regla lo dicen en su cabecera', () => {
    for (const [f, test] of CABECERAS) {
      const cabecera = leer(f).slice(0, 700)
      expect(cabecera, `${f} sin cabecera`).toMatch(/AGENTS\.md|NO-TOCAR/)
      expect(cabecera, `${f} no dice qué test salta`).toMatch(test)
    }
  })

  it('AGENTS.md ya incluye el mapa del repo', () => {
    expect(leer('AGENTS.md')).toMatch(/src\/components\/narrativa\//)
  })
})
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npx vitest run tests/documentacion.test.ts`
Expected: FAIL — «existe el registro de decisiones».

- [ ] **Step 3: Write minimal implementation**

`docs/DECISIONES.md`:
```markdown
# Registro de decisiones

Qué se decidió, y sobre todo por qué. Sin el porqué, la próxima sesión deshace la
decisión de buena fe.

## El sitio no dice que las aceitunas estén teñidas

**Decisión:** el copy afirma que el fruto se recoge sin madurar y se ennegrece por
oxidación, que el pigmento es de la propia aceituna, y que la sal de hierro
uniformiza el color en lugar de aportarlo.

**Por qué:** la versión popular («las tiñen») es falsa. El brief inicial del
proyecto ya decía «oxidadas artificialmente», correctamente; la deriva hacia
«teñidas» se introdujo al redactar el primer borrador del diseño y se corrigió al
verificar las fuentes. Javier Sánchez Perona (CSIC, Instituto de la Grasa) y Marta
Berlanga Del Pozo publicaron «Todas las aceitunas negras de mesa son de verdad»
explicando que no hay colorante, que el negro son pigmentos de tipo melanina
formados a partir de los compuestos fenólicos del propio fruto, y que la sal de
hierro forma complejos con esos fenoles. Publicar la versión pegadiza habría hecho
el sitio desmontable en un tuit.

**Consecuencia:** «teñidas», «colorante» y «fraude» son palabras prohibidas, con
un test que lo comprueba.

## La tesis es legal, no química

**Decisión:** la columna vertebral del sitio es el artículo 12.3.b del Real Decreto
679/2016: declarar el proceso de elaboración es una mención **voluntaria**,
mientras que declarar el color es obligatoria. Y el artículo 4 define «negras»
(oxidadas) y «negras naturales» (recogidas en plena madurez) como categorías
distintas.

**Por qué:** es el único ángulo que resiste al contraargumento del CSIC. Ellos
tienen razón en la química, en la legalidad y en la ausencia de riesgo. Donde
queda un hueco real es en el vocabulario: quien compra «aceitunas negras» cree
comprar fruta madurada en el árbol, y la norma permite no aclarárselo. Eso es
verificable, citable y no lo estaba contando nadie.

**Consecuencia:** la prueba que se le da al lector es la ausencia de la palabra
«naturales» en la denominación. El E-579 pasa a ser confirmación, no prueba.

## La sección 10 admite lo que el sitio no sabe

**Decisión:** la última pantalla cita y enlaza el artículo del CSIC dándole la
razón, aclara que no es fraude ni riesgo sanitario, y dice explícitamente qué dos
cosas no se han podido verificar.

**Por qué:** es lo único que hace creíbles las nueve pantallas anteriores. Un sitio
que solo acusa se lee como panfleto. Y desmontar el bulo que nos beneficiaría es
el activo principal: hay cientos de artículos repitiendo «van teñidas» y ninguno
que lo corrija con el BOE en la mano.

## Cero JavaScript y CSS ligado al scroll

**Decisión:** 0 KB de JS. El movimiento con `animation-timeline: view()`, envuelto
en `@supports`, y sin él se ven los estados finales estáticos.

**Por qué:** el requisito era «que cargue lo más rápido posible». Una narrativa sin
estado no necesita runtime. Y el propietario del proyecto no quiere React ni
Tailwind.

**Consecuencia:** un test falla si aparece un solo `.js` en `dist/`. Solo se
permite `<script type="application/ld+json">`, que no es ejecutable.

## El `#000` solo en el camino de la fábrica

**Decisión:** toda la paleta es cálida y no contiene ni un gris ni un negro puro,
salvo en los elementos del camino industrial.

**Por qué:** el color honesto de una aceituna madura es granate, violeta o castaño
oscuro; nunca `#000`. El negro absoluto y uniforme ES el artificio del que habla
el sitio. Usarlo en cualquier otra parte destruye el argumento visual. Por la
misma razón la fábrica se compone en monoespaciada y el árbol en serif: la
tipografía ejecuta el argumento, no lo acompaña.

## Las acuarelas van en AVIF sobre blanco con `multiply`

**Decisión:** las ilustraciones se generan con IA sobre papel blanco y se integran
con `mix-blend-mode: multiply` sobre el fondo arena.

**Por qué:** un PNG con alpha pesaría cuatro o cinco veces más y los bordes aguados
se recortan mal. Pedirle a la IA el fondo exacto no funciona: nunca da el mismo
hex. El `multiply` hace desaparecer el blanco, mantiene la compresión y, de
propina, tiñe las aguadas con el arena, lo que unifica diez piezas que la IA
entregará desiguales. Es la paleta del sitio imponiéndose a la de la máquina.

**Consecuencia:** hay una línea de crédito de IA en la sección 10. Un sitio que
habla de apariencia artificial y se ilustra con IA sin decirlo es munición para un
lector hostil.

## Una narrativa larga más cuatro páginas de respuesta

**Decisión:** la portada es un scroll narrativo de diez pantallas sin menú de
cabecera. Detrás hay cuatro páginas sobrias que responden una pregunta cada una.

**Por qué:** para «aceitunas negras» a secas no hay nada que rascar: ese resultado
es supermercado y receta, intención de compra. El terreno ganable es el racimo de
preguntas largas, y una sola página solo puede optar a una intención. La narrativa
convierte y se comparte; las páginas de respuesta traen la gente de Google. Y el
menú de cabecera se descartó porque mata la pantalla de apertura.

## Se señala la práctica, nunca las marcas

**Decisión:** se recomiendan variedades y se explica la norma. No se nombra ninguna
marca ni se fotografía ningún envase real. La etiqueta del decodificador es
genérica y compuesta.

**Por qué:** nombrar marcas exige verificar y mantener al día formulaciones que
cambian, envejece pésimamente en un sitio estático sin CMS, y expone a
reclamaciones sin añadir nada al argumento. Además, el lector se queda con una
habilidad en vez de con una lista.
```

Añadir a `AGENTS.md`, antes de la sección «El error que vas a cometer»:
```markdown
## Mapa del repo

| Ruta | Qué hay |
|---|---|
| `src/content/narrativa.ts` | El copy de las diez pantallas. **Todo el texto se edita aquí, no en los componentes.** |
| `src/content/fuentes.ts` | La bibliografía. Toda afirmación referencia un `id` de aquí. |
| `src/content/respuestas/` | Las cuatro páginas de respuesta, en Markdown. |
| `src/components/narrativa/` | Un componente por pantalla, del 01 al 10. |
| `src/components/figuras/` | Los diagramas en SVG y CSS: bifurcación, línea de proceso, decodificador, comparador. |
| `src/components/ui/` | Piezas reutilizables: acuarela, prosa, bibliografía. |
| `src/styles/tokens.css` | La paleta y la escala. Ningún componente declara colores literales. |
| `src/assets/acuarelas/` | Las diez ilustraciones. Se sustituyen sin tocar código. |
| `tests/` | Los guardianes del spec. No los relajes: lee `docs/NO-TOCAR.md`. |
| `scripts/` | Generación de acuarelas provisionales y de la imagen social. |
```

Cabeceras. Al principio de `src/content/fuentes.ts` y `src/content/narrativa.ts`:
```ts
/**
 * Copy y bibliografía de la narrativa. Lee AGENTS.md antes de editar.
 *
 * REGLA: prohibidas las palabras «teñidas», «colorante» y «fraude» (son falsas),
 * y toda mención a pérdida de polifenoles o a tiempos concretos del proceso
 * (no verificados). Toda afirmación de hecho referencia un id de FUENTES.
 * Si lo rompes, falla tests/copy.test.ts.
 */
```

Al principio de `src/styles/tokens.css`:
```css
/**
 * Paleta del sitio. Lee AGENTS.md antes de editar.
 *
 * REGLA: toda cálida, sin un solo gris ni rgba() de negro para texto. El negro
 * puro (--negro) se usa EXCLUSIVAMENTE en el camino de la fábrica: es el
 * artificio del que habla el sitio. Sin modo oscuro, a propósito.
 * Si lo rompes, falla tests/paleta.test.ts.
 */
```

Al principio de `src/components/ui/Acuarela.astro`, dentro del frontmatter:
```astro
---
/**
 * Acuarela. Lee docs/NO-TOCAR.md antes de editar.
 *
 * REGLA: no quites el mix-blend-mode: multiply. Las ilustraciones llegan sobre
 * papel blanco y el multiply lo hace desaparecer sin recurrir a PNG con alpha,
 * que pesaría 4-5x. De propina, tiñe las aguadas con el arena del fondo y
 * unifica diez piezas desiguales. No es un apaño: es la decisión.
 */
```

- [ ] **Step 4: Run test to verify it passes**

Run: `npm test && npm run test:a11y`
Expected: PASS — 68 tests en Vitest y las cinco rutas limpias en Playwright.

- [ ] **Step 5: Commit**

```bash
git add docs/DECISIONES.md AGENTS.md src/content src/styles/tokens.css src/components/ui/Acuarela.astro tests/documentacion.test.ts
git commit -m "docs: registro de decisiones, mapa del repo y cabeceras de reglas"
```

---

## Verificación de cobertura del spec

| Sección del spec | Tareas |
|---|---|
| §1 Tesis, §3 filo editorial | 4 (copy y guardianes de vocabulario), 10, 12 |
| §2 No objetivos | 1 (cero JS), 2 (sin modo oscuro), 10 y 12 (sin marcas) |
| §4.1 Las diez pantallas | 7, 8, 9, 10, 11, 12 |
| §4.2 Páginas de respuesta | 14 |
| §5.1 Tipografía como argumento | 9 (mono en fábrica, serif en árbol) |
| §5.2 Paleta | 2, con guardián del `#000` en 8 y 11 |
| §5.3 Tipografía | 3 |
| §5.4 Ilustración y `multiply` | 6 |
| §6 Movimiento y accesibilidad | 13 |
| §7 Arquitectura técnica | 1, 5, 14 |
| §8 Presupuesto | 1 y 15 |
| §9 SEO | 5, 14, 15 |
| §10 Hechos verificados y bibliografía | 4 (`fuentes.ts`), 12 (renderizado) |
| §10.3 Prohibiciones | 4 (polifenoles), 9 (cifras de tiempos) |
| §10.5 Contraargumento del CSIC | 12 |
| §11 Riesgos | los guardianes de 2, 4, 9, 13 |
| §13 Documentación IA-first | 1.5 (entrada, reglas, glosario), 16 (decisiones, mapa, cabeceras) |

**Pendiente por decisión del usuario:** el requisito de §12 del spec, que aún no
se ha formulado. Cuando aparezca, se añade como tarea nueva antes de la 15.
