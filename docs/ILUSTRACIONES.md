# Las diez acuarelas

Prompts listos para pegar. **El ancla de estilo es idéntica en las diez y no se
toca**: es lo único que hace que las diez piezas parezcan de la misma mano, que es
donde falla el 90 % de los sitios ilustrados con IA.

## Reglas de producción

1. **Fondo blanco, siempre.** El sitio las integra con `mix-blend-mode: multiply`
   sobre el fondo arena `#EDE4D4`. Si generas sobre fondo de color, el `multiply`
   lo oscurece y la pieza se ensucia. Blanco de papel, sin excepción.
2. **Márgenes generosos.** No encuadres apretado: el `multiply` necesita blanco
   alrededor para que la pieza se funda con el papel. Si te llega recortada, pide
   otra.
3. **Sin texto de ningún tipo.** Ni rótulos, ni firmas, ni números. La IA los
   escribe mal siempre y aquí encima serían mentira.
4. **JPEG a la máxima resolución** que te dé la herramienta. El build genera el
   AVIF y el WebP; tú no optimices nada.
5. **Nombre de fichero exacto**, en `src/assets/acuarelas/`. El código no cambia
   al sustituirlas.

## Ancla de estilo (idéntica en las diez)

```
loose modern Mediterranean watercolour, wet-on-wet washes with bleeding edges,
no ink outline, no linework, warm palette of olive green ochre terracotta and
deep garnet, generous white paper margins, natural daylight, painterly and
restrained, editorial illustration, no text, no lettering, no people's faces
```

## Las diez piezas

Cada prompt = **escena + ancla**. La proporción importa: es la del hueco que
ocupa en el diseño.

### 01 · `01-aceituna-negra.jpg` — 4:3

La pieza más importante del sitio, y la única que debe resultar **ligeramente
incómoda**: perfecta de más.

```
A single table olive, absolutely uniform matte-black, unnaturally flawless and
glossy, perfectly ovoid, centred on bare white paper with nothing else around it,
subtle cast shadow, unsettlingly perfect — loose modern Mediterranean watercolour,
wet-on-wet washes with bleeding edges, no ink outline, no linework, warm palette
of olive green ochre terracotta and deep garnet, generous white paper margins,
natural daylight, painterly and restrained, editorial illustration, no text,
no lettering, no people's faces
```

### 02 · `02-rama-verde.jpg` — 3:2

```
An olive branch in early September, hard unripe fruit in bright green and straw
yellow, narrow silver-green leaves, a few olives only, airy and sparse — [ANCLA]
```

### 03 · `03-rama-envero.jpg` — 3:2

El envero: el momento del cambio. Que se vea el desorden.

```
An olive branch mid-envero, fruit caught between colours: some still green, some
pale pink, some wine-rose, some chestnut brown, every olive at a different stage,
deliberately uneven — [ANCLA]
```

### 04 · `04-rama-madura.jpg` — 3:2

```
An olive branch in full ripeness, deep garnet and violet-black fruit, some olives
shrivelled and wrinkled, skin with an iridescent sheen, heavy on the stem,
no two olives the same shade — [ANCLA]
```

### 05 · `05-olivar-diciembre.jpg` — 16:9

```
An old Mediterranean olive grove in December, low raking winter sun, gnarled
trunks and dry ochre soil, long shadows, distant hills washed in pale indigo and
terracotta, calm and empty — [ANCLA]
```

### 06 · `06-manos-cesta.jpg` — 3:2

Sin caras: manos y cesta.

```
Two weathered hands picking ripe olives into a woven wicker basket, seen from
above, only hands and forearms in frame, basket half full of dark uneven fruit,
warm daylight — [ANCLA]
```

### 07 · `07-salmuera-tarro.jpg` — 4:5 (vertical)

```
A clear glass jar of olives in brine on a plain surface, dark uneven fruit
suspended in cloudy liquid, a bay leaf and a strip of orange peel inside, light
passing through the glass — [ANCLA]
```

### 08 · `08-sal-hierbas.jpg` — 3:2

```
A scattered handful of coarse sea salt with sprigs of thyme, wild fennel and a
crushed garlic clove on bare white paper, loose arrangement, nothing else in
frame — [ANCLA]
```

### 09 · `09-nave-industrial.jpg` — 16:9

El único tema frío. Misma técnica, sujeto duro: el contraste lo hace el contenido,
no el estilo.

```
The interior of a food processing plant, rows of tall cylindrical steel tanks and
pipework, a grid of walkways, cold and orderly and empty of people, muted greys
and olive shadows inside the warm palette — [ANCLA]
```

### 10 · `10-lata-mesa.jpg` — 3:2

```
An opened tin of black olives on a kitchen table seen slightly from above, the
lid peeled back, uniform dark olives inside in their liquid, plain unbranded tin
with no label and no text, everyday and unremarkable — [ANCLA]
```

## Qué revisar antes de aceptar una pieza

- ¿Está sobre **papel blanco** de verdad, o hay un tinte crema o gris?
- ¿Tiene **margen** alrededor o está recortada al borde?
- ¿Se ha colado **texto**, una firma o un logotipo?
- ¿Hay **caras**? No debe haberlas.
- Puesta al lado de las otras, ¿parece de **la misma mano**?

Si falla lo último, casi siempre es porque se ha alterado el ancla. Vuelve a
pegarla literal.
