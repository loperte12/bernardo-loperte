# Auditoría UI/UX — EG Route Plan · FASE 1: informe

**Fecha:** 23-sep-2026 · **Alcance:** `D:\egapp` (app Expo/RN + `packages/ui-kit`) ·
**Método:** MCP `ui-ux-suite` + censo propio + guardia del proyecto

**Veredicto: 6,2 / 10 — «Adequate»** (calculado con `uiux_score_overall` sobre 11 dimensiones medidas).

**Resumen en una frase:** el sistema de diseño no hay que reconstruirlo, hay que **terminarlo** — existe,
está razonado y es bueno, pero **la mitad del código lo ignora**, y donde lo ignora no es por descuido:
es porque varias escalas declaradas **no coinciden con la escala que el producto usa de verdad**.

**Nota de revisión (24-sep-2026):** este informe es la medición del 23-sep. Se conserva tal cual, con
tres correcciones marcadas en el sitio: dos datos que resultaron falsos o incompletos (§3.3 y §3.5) y el
estado de `altura`, que ya está adoptada (§6).

---

## 0. Cómo se midió, y la advertencia que hay que leer antes que nada

### 0.1 La suite es para web y esta app es React Native

`uiux_scan_project` miró `D:\egapp` y devolvió `hasDesignTokens: false`, `themeSystem: null`,
`componentLib: null`. **Es falso, y está comprobado que es falso:**
`packages/ui-kit/src/theme/` tiene `colors.ts` (30 tokens semánticos, con acta de decisión del 17/09),
`escalas.ts` (7 escalas) y `movimiento.ts`. Lo que pasa es que **los buscadores solo leen CSS**, y aquí
no hay ni un `.css`: 17 ficheros CSS en el proyecto y **los 17 son de las demos de `.auditoria-servicios`**.
`uiux_extract_colors` leyó 1.189 colores de esas demos: `#f5f5f5`, `#27272a`, `--bm-accent`… el sistema de
diseño de ReactBits, no el de EG Route Plan.

Consecuencia directa y grave: **`uiux_audit_run` dio 5,8/10 y sus tres hallazgos «críticos» no existen.**
Los tres son señales que en React Native no aplican:

| Lo que dice la suite | Por qué no aplica |
|---|---|
| «No responsive breakpoints» | RN no usa breakpoints: usa flex + dp + `useWindowDimensions` |
| «No `focus-visible:` — el teclado no se ve» | App táctil. No hay teclado que navegue |
| «Solo 0 variantes `hover:`» | No hay ratón |
| «Falta `<html lang>` / no hay `<h1>`» | No hay HTML. Es una vista nativa |
| «No `prefers-reduced-motion`» | RN usa `AccessibilityInfo.isReduceMotionEnabled` |
| «No hay primitivas en `components/ui/`» | Hay `packages/ui-kit/src/{primitives,micro,feedback,a11y}` |

Y el daño peor es el del **color y la tipografía: les dio 8/10 por no encontrar nada.** Ausencia no es
calidad. Un 8 que sale de un cero.

### 0.2 Lo que sí es fiable de la suite (y por eso se usó)

- **`uiux_check_contrast`** — el mejor de todos. Contraste puro y cálculo APCA; no depende del
  framework. **Es la herramienta que encontró el hallazgo crítico §3.1.**
- **`uiux_generate_*`** — genera, y generó. Útil, pero **hay que verificar lo que genera** (§4.3).
- **`uiux_score_overall`** — calcula de verdad una media ponderada de lo que le des.
- **`uiux_audit_log` + `uiux_audit_report`** — el registro funciona y el informe recoge lo que se registra.

### 0.3 Lo que NO es fiable

- **`uiux_score_dimension`: devuelve una plantilla, ignora los datos.** Se le pasaron los datos reales de
  color (26 parejas medidas, 5 fallos de AA en oscuro) y devolvió `score: 8` con `confidence: insufficient`
  y la queja genérica «Missing semantic colors: primary, error, success, warning» — que es **falsa**:
  `colors.ts` tiene 30 tokens semánticos, `primary` incluido. Se le pasó después el censo de componentes
  y devolvió `score: 2` con las mismas seis frases sobre `cn()`/`clsx()`, `cva` y `lucide-react`.
- **`uiux_audit_run`** en un proyecto sin CSS: sus puntajes no se usaron. Sí se conservó como evidencia
  de la limitación.

**Regla que sale de aquí:** en un proyecto React Native, de la suite se usa el **contraste** y los
**generadores**, y el diagnóstico se construye **midiendo el código**. El resto es ruido con formato.

---

## 1. Perfil real del proyecto (medido por nosotros)

| Dato | Valor |
|---|---|
| Ficheros fuente en zonas de app (`app`, `components`, `core`, `api`, `state`, `utils`, `constants`) | **310** |
| Ficheros que usan al menos un token del kit | **208 (67 %)** |
| Usos de token | **4.009** |
| Literales de escala escritos a mano | **3.913** |
| **Proporción token / literal** | **1,02** |
| Ficheros con alguna deuda de diseño | **192 (62 %)** |
| Pantallas en el mapa de rutas | 131 (0 enlaces rotos, 0 huérfanas) |
| Sistema de diseño | `packages/ui-kit`: `colors.ts` · `escalas.ts` · `movimiento.ts` |
| Guardia de diseño | `pruebas/verifica-diseno.cjs` (trinquete: la deuda solo puede bajar) |

**El número que resume la auditoría es 1,02.** Por cada token que se usa, hay un valor escrito a mano.
No es un proyecto sin sistema de diseño: es un proyecto **con un sistema de diseño a medio adoptar**.

---

## 2. Tarjeta de puntuación

Calculada con `uiux_score_overall` sobre las dimensiones que tenemos medidas. **`performance` queda fuera
a propósito: no se ha medido nada de rendimiento y no se inventa.**

| Dimensión | Nota | Peso | Base de la nota |
|---|---|---|---|
| Color System | **8,0** | 10 % | 16/16 parejas de claro en AA o mejor; 30 tokens semánticos con acta; falta el modo oscuro (§3.1) |
| Information Architecture | **8,0** | 5 % | 131 rutas, 0 enlaces rotos, 0 huérfanas, mapa generado y auditado |
| Typography System | **7,5** | 10 % | 2.462 usos de `tipografia` (79 % de adopción); `peso` al 8 % (§3.2) |
| Accessibility | **6,5** | 12 % | `a11y/` real, contraste documentado; alturas táctiles sin adoptar (§3.5) |
| Component Quality | **6,0** | 10 % | Kit con 4 módulos y 4.009 usos; acciones de tarjeta duplicadas en 3 sitios (§3.6) |
| Visual Hierarchy | **6,0** | 10 % | Escala tipográfica real; con `peso` al 8 % la jerarquía la decide cada pantalla |
| Interaction Quality | **6,0** | 8 % | `movimiento.ts` con tokens; `trazo` al 18 % (§3.4) |
| Visual Polish | **6,0** | 7 % | 3 niveles de elevación (antes 12 valores en 16 ficheros); radios al 49 % |
| Platform Appropriateness | **6,0** | 5 % | Mínimos de 44 dp documentados pero no aplicados (§3.5) |
| Responsiveness | **5,0** | 8 % | Rejilla fija a 4 columnas; `font_scale` resuelto con `minHeight` en 1 sitio |
| **Layout & Spacing** | **4,0** | 10 % | **La peor**: 5.446 literales, 54 % fuera de la escala, y la guardia no la mira (§3.3) |
| Performance UX | — | 5 % | **No medido** |
| **TOTAL** | **6,2** | | **«Adequate»** |

---

## 3. Los siete hallazgos

### 3.1 · CRÍTICO — Modo oscuro: los 5 colores de marca suspenden AA al usarse como texto

**La evidencia.** Los colores de marca se diseñaron para **llevar texto blanco encima**, y ahí cumplen
(blanco sobre `#0066CC` = 5,57). Pero usados **como texto o icono** sobre el fondo oscuro `#17171A`:

| Token | Valor | Sobre `#17171A` | APCA |
|---|---|---|---|
| `primary` | `#0066CC` | **3,21 — falla** | −22,8 · «decorativo» |
| `success` | `#1E7A45` | **3,34 — falla** | −24,0 · «decorativo» |
| `danger` | `#C62828` | **3,18 — falla** | −22,6 · «decorativo» |
| `secondary` | `#C2410C` | **3,45 — falla** | −24,9 · «decorativo» |
| `neutral` | `#64748B` | **3,76 — falla** | −27,4 · «decorativo» |

**El por qué, y es un hueco de diseño, no un descuido.** El kit **sí** tiene la variante para el caso
contrario: `dangerText #991B1B` y `warningText #78350F` existen «porque danger (3,4:1) y warning no llegan
a AA como texto» sobre fondo claro. **No existe el equivalente para fondo oscuro.** O sea: el kit pensó
«color de marca → texto sobre blanco» y no pensó «color de marca → texto sobre negro». Y `darkColors`
hace `...brand`: hereda los cinco valores saturados tal cual.

**Impacto.** En modo oscuro —el de la noche, y el que la gente usa en la cama— las etiquetas de estado,
los precios en verde y los avisos en ámbar/rojo son ilegibles para cualquiera, no solo para quien tiene
baja visión.

**El arreglo (pequeño, 6 valores, ya verificados):**

```ts
/** SOLO para usar como TEXTO o ICONO sobre fondo oscuro. Los base se quedan como están: ahí llevan blanco encima. */
primaryDark:   '#4D9AEB',  // 6,08 sobre #17171A · 5,31 sobre tarjeta
successDark:   '#45B87A',  // 7,15 · 6,24
dangerDark:    '#F26D6D',  // 6,12 · 5,35
secondaryDark: '#F08A4B',  // 7,19 · 6,28
neutralDark:   '#B0B8C4',  // 8,94 · 8,30 · 7,81
warningDark:   '#F5B942',  // 10,14 · 8,86
```

**Esfuerzo:** `small`. Seis líneas en `colors.ts` y sustituir los usos como texto en modo oscuro.
**Estado:** **HECHO** (24/09/2026). Los seis están en `colors.ts` y en el `ThemeColors`.
**Ley citada:** Jakob's Law (el usuario espera que el modo oscuro sea el mismo producto, legible) ·
Postel's Law.

---

### 3.2 · CRÍTICO — La escala `peso` existe pero no se usa: 8 % de adopción

**La evidencia.** `peso` tiene 4 peldaños y **175 usos** en 31 ficheros. Hay **1.975 literales** de
`fontWeight`. El reparto real, medido hoy:

| Valor escrito a mano | Veces | ¿Está en la escala? |
|---|---|---|
| **800** | 727 | **No** |
| 900 | 532 | Sí (`titulo`) |
| 700 | 500 | Sí (`fuerte`) |
| **600** | 214 | **No** |
| 500 | 2 | Sí (`medio`) |
| 400 | **0** | Sí (`normal`) |

**El por qué.** **1.259 de 1.975 literales (64 %) usan valores que la escala no contiene.** No se puede
migrar a una escala donde el valor más escrito no existe. Y el peldaño que el kit añadió justificándose
—«antes no existía el 400, así que todo estaba en semibold o más y nada destacaba»— **tiene cero usos**:
el diagnóstico era correcto y la solución no se adoptó.

**El arreglo, y es más barato de lo que parece:**

```ts
export const peso = {
  normal: '400',
  medio:  '500',
  fuerte: '700',
  maximo: '800',   // NUEVO — se queda con los 727 usos más frecuentes del proyecto
  titulo: '900',
} as const;
```

**Con un solo peldaño nuevo (`maximo: 800`) se conservan 1.759 de los 1.975 literales sin mover un píxel.**
Solo queda decidir el 600 (214 usos): a `medio` (500) o a `fuerte` (700). **Es una decisión de diseño, no
de código**, y hay que tomarla mirando dos pantallas, no razonando.

**Esfuerzo:** `large` (es migración por tandas, con la guardia como trinquete).
**Estado:** `maximo` añadido y el `600` **decidido a `medio`** (24/09/2026). La migración de los literales
es Fase 3. Al aplicarlo apareció un dato que refuerza la decisión: **la app no declara ninguna
`fontFamily` ni carga fuentes**, así que `600` es el único peso cuyo dibujo decide el aparato.
**Ley citada:** Law of Similarity · Law of Prägnanz.

---

### 3.3 · CRÍTICO — Espaciado: 5.446 literales, 54 % fuera de la escala — y la guardia no lo mira

**La evidencia.** La familia peor adoptada y la única **ciega para la guardia** (`verifica-diseno.cjs`
cuenta `hex`, `fontSize`, `borderRadius`, `fontWeight`, `borderWidth` y `precioFigura`; **espaciado no**).
Medido a mano, en 20 valores distintos:

| px | 8 | **10** | 12 | 6 | **14** | 16 | 4 | **2** | **3** | **7** | 18 | 5 | 20 | 24 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| veces | 834 | 791 | 754 | 563 | 400 | 352 | 337 | 273 | 142 | 136 | 117 | 91 | 81 | 78 |

**2.489 son múltiplos de 4; los 2.957 restantes (54 %) no.** La escala declara `4/8/12/16/24/32`.
El segundo valor más usado después del 8 es **10**, que no está en la escala.

**El por qué, y no es dejadez.** En una rejilla densa de marketplace a 360 dp, con celdas de 80 dp, la
granularidad de 4 px es **demasiado gruesa**: entre 8 y 12 no hay nada, y hay sitios donde 8 aprieta y 12
sobra. Cada pantalla se ha buscado su valor. **La escala declara una base que el producto no usa.**

**El arreglo.** No obligar al 4. La evidencia dice que **la base real del producto es 2 px**:

```ts
/* Antes: nombres de intención sobre base 4 (4/8/12/16/24/32) → 432 usos frente a 5.446 literales.
   Ahora: el espaciado se nombra por su valor, porque en espaciado el número ES la intención.
   Esto convierte los «excepciones» en usos legítimos SIN TOCAR UN PÍXEL. Solo después se migra. */
export const espaciado = {
  e2: 2, e3: 3, e4: 4, e5: 5, e6: 6, e7: 7, e8: 8, e9: 9, e10: 10,
  e11: 11, e12: 12, e13: 13, e14: 14, e16: 16, e18: 18, e20: 20, e22: 22, e24: 24, e32: 32,
} as const;
```

> **CORRECCIÓN (24-sep-2026), y va contra este mismo informe.** La primera versión declaró **base 2 pura**
> (11 peldaños) y se aplicó. Al medir **qué quedaba fuera** apareció el fallo: **una base 2 no puede
> expresar un número impar**, y los impares del proyecto no son ruido — **3 (181 literales), 7 (168),
> 5 (101) = 450**. Declarar base 2 y dejar 450 usos fuera era repetir el error original con otro número.
> La escala aplicada es la de arriba, 20 peldaños: cubre **95,7 %**, frente al 43,9 % de la base 4 y el
> 82,3 % de la base 2. El detalle está en `AUDITORIA-UIUX-TOKENS.md` §4.1.

Renombrar aquí es lo **más barato de las cinco familias precisamente porque casi nadie la usa**
(432 usos). Es un codemod mecánico — y se hizo: **450 usos en 34 ficheros, 0 claves viejas.**

**La alternativa, si se insiste en el 4:** mover los 791 usos de 10 a 8 o a 12. **Eso sí cambia el
aspecto de la app entera** y es una decisión de diseño: 791 sitios son demasiados para decidirlos sin
mirar.

**Y hay que arreglar la guardia:** un trinquete que no ve el espaciado no aprieta el espaciado. Añadir la
regla de `padding|margin|gap` a `verifica-diseno.cjs` y crear su base. **HECHO**, con la base regenerada
en el mismo paso.

**Esfuerzo:** `large` (declarar es `trivial`; migrar es `large`).
**Ley citada:** Law of Proximity · Law of Prägnanz.

---

### 3.4 · IMPORTANTE — `trazo`: 122 usos frente a 550 literales, y el kit declaró media escala

**La evidencia.**

| Valor | 1 | 1,5 | 2 | 3 | 0 | 1,2 | 0,6 |
|---|---|---|---|---|---|---|---|
| veces | **450** | 66 | 19 | 6 | 5 | 3 | 1 |

**Aquí la escala acertó:** `fino 1` / `base 1,5` / `fuerte 2` / `marcado 2,5` cubre el 97 % de los
literales. **450 de 550 son exactamente `fino`.** Es la familia donde **no hay nada que rediseñar, solo
que migrar** — y con 122 usos frente a 550, no se ha migrado.

**El por qué del resto.** El comentario del kit ya midió el origen: nueve grosores entre bordes e iconos.
Y ahí está la parte que falta: el kit declaró la escala del **borde** (1/1,5/2/2,5) pero **el producto
tiene además una escala de trazo de icono** (`1.8`, `2`, `2.2`, `3` como `strokeWidth`), que no está
declarada en ningún sitio.

**El arreglo.** (1) Migrar `borderWidth` → `trazo.*` (mecánico, 450 de 550 son un solo valor).
(2) Declarar la escala de trazo de icono o documentar que comparte `trazo`.
**Estado:** `trazoIcono` declarado y exportado (24/09/2026).

**Esfuerzo:** `medium`. **Ley citada:** Law of Similarity.

---

### 3.5 · IMPORTANTE — Alturas táctiles: la escala con menos adopción del kit

**La evidencia.** `altura` tiene **12 usos en 3 ficheros de 310**. Y **dos de sus cuatro peldaños
—`campo` (50) y `boton` (52)— no tienen un solo uso.** El mínimo de plataforma es 44 dp (Apple HIG) y
48 el que recomienda Android. La auditoría del kit ya encontró dos sitios por debajo: **la casilla del
carrito mide 21 dp y el micrófono del buscador 28 dp**, y los dos se «salvaron» con `hitSlop`.

> **CORRECCIÓN (24-sep-2026): el «micrófono del buscador» NO EXISTE.** Era una nota sin verificar. El
> censo real, medido sobre los `StyleSheet`: **152 objetos de estilo** por debajo de 44 dp, de los que
> **39 llevan nombre de control**. La casilla del carrito era correcta (`21×21` en `lifebook-carrito.tsx`)
> y ya está arreglada a **44 dp de dedo con 21 de ojo**, sin cambiar el diseño. El resto del censo está en
> `AUDITORIA-UIUX-TOKENS.md` §7.1, y su conclusión es aritmética: **en una fila con `gap`, un `hitSlop` solo
> alcanza `tamaño + gap`**, así que la mayoría de esos 39 no llegan a 44 sin cambiar el diseño.

**El por qué importa más aquí que en ningún otro sitio.** `hitSlop` agranda el **área táctil** pero **no
el elemento**: el usuario sigue viendo un cuadrado de 21 dp y apuntando a él. En un mercado al sol, con el
móvil en una mano y el dedo gordo, eso no es un detalle estético: es un toque que se falla.

**El arreglo.** (1) Auditar los `Pressable` por debajo de 44 dp y **subir el elemento**, no el hitSlop.
(2) Migrar las alturas de campo y botón a tokens, que ya están decididos y sin usar. **Las dos cosas
hechas el 24/09/2026:** `altura.campo` y `altura.boton` ya no están muertos y la casilla está a 44.

**Esfuerzo:** `medium`. **Ley citada:** Fitts's Law.

---

### 3.6 · IMPORTANTE — Acciones duplicadas en las tarjetas: 3 copias del favorito

**La evidencia.** El kit se usa en 208 de 310 ficheros: **no es cierto que falten primitivas**. Pero
cuando la tarjeta es la misma en tres pantallas, la lógica se copia. Al pintar la pantalla de
subcategoría hubo que extraer `useAccionesProducto.ts` **porque copiar el corazón habría roto la regla
escrita en la home** («en un solo sitio»). Quedan dos copias conocidas (`ecomerse-detail.tsx`,
`ecomerse-tienda.tsx`) que conservan estado propio del producto.

**El por qué del riesgo.** El favorito optimista con vuelta atrás existe hoy **en 3 versiones**. Cada
copia es un sitio donde el comportamiento se desvía sin que nadie lo vea.

**El arreglo.** Regla corta: **si una acción se repite en 3 pantallas, sale a un hook del módulo.**
Aplicarla a `ecomerse-detail`/`ecomerse-tienda` y revisar `lifebook` y `food` con el mismo criterio.
**No** crear «un componente universal de tarjeta»: las tres tarjetas tienen datos distintos y unirlas
costaría más de lo que ahorra. **HECHO el 24/09/2026:** las tres versiones son una.

**Esfuerzo:** `medium`. **Ley citada:** Tesler's Law · Law of Similarity.

---

### 3.7 · CRÍTICO (sobre la propia propuesta) — La paleta generada es MENOS accesible que la actual

Esto no es un defecto de la app: es un aviso sobre el plan de reconstrucción. Detalle en §4.3.

---

## 4. El diagnóstico que reordena todo el plan

### 4.1 La escala declarada y la escala real no son la misma

Al medir la distribución de literales sale un patrón que se repite en tres familias y explica por qué la
adopción está al 50 %:

| Familia | Lo que el código escribe de verdad | Lo que la escala contiene | ¿Coinciden? |
|---|---|---|---|
| `fontSize` | **15** (151) · **10,5** (110) · **17** (83) · 14,5 (56) · 10 (44) | 9 / 11 / 12 / 14 / 16 / 20 / 28 | **Ninguno de los 4 primeros** |
| `fontWeight` | **800** (727) · 900 (532) · 700 (500) · 600 (214) | 400 / 500 / 700 / 900 | 800 y 600 **no existen** |
| `borderRadius` | **14** (168) · **10** (142) · 22 (67) · 18 (52) | 8 / 12 / 16 / 999 | 14 y 10 **no existen** |
| `borderWidth` | 1 (450) · 1,5 (66) · 2 (19) | 1 / 1,5 / 2 / 2,5 | **Sí** → solo migrar |

**Donde la escala acertó (`borderWidth`), el trabajo es migrar. Donde la escala se inventó el valor
(`peso`, espaciado, radios), el código la ignora — y hace bien en ignorarla, porque el valor no le sirve.**

Esto cambia la naturaleza del plan: **no es «sustituir el sistema de diseño», es «reconciliar las escalas
declaradas con las que el producto usa, y después migrar».** Es más barato, más seguro y no toca la
identidad visual que el producto ya tiene.

### 4.2 Lo que la suite llamó «crítico» y no lo es

«No consistent type scale detected — sizes appear random» (typography) y «Inconsistent spacing values»
(layout) acertaron **en el diagnóstico y fallaron en la causa**: no es aleatoriedad, es una escala
declarada que no encaja. Y en color/tipografía dio 8/10 porque no encontró nada que medir.

### 4.3 Y el aviso más importante: no adoptar la paleta generada

`uiux_generate_palette` propuso, entre otros, `success #22c365`, `warning #ec9c13`, `info #2d7fd2` y un
neutro 600 `#898b8f`. Comprobados con **el comprobador de contraste de la propia suite**:

| Propuesta | Medición | ¿Se adopta? |
|---|---|---|
| blanco sobre `success #22c365` | **2,32 — falla** (y falla en texto grande) | **No** |
| blanco sobre `warning #ec9c13` | **2,25 — falla** | **No** |
| blanco sobre `info #2d7fd2` | **4,14 — falla** en texto normal | **No** |
| `#898b8f` como texto secundario sobre `#f9fafb` | **3,27 — falla** | **No** |
| `error #d61f1f` | 5,15 AA | Indiferente: no mejora `#C62828` (5,62) |

**Son exactamente los cuatro defectos que ya se arreglaron el 17/09.** El acta lo dice con estas
palabras: *«Antes el naranja daba 2,57 y el azul 3,66: las etiquetas de los botones no se leían al sol.»*
Adoptar la paleta generada sería **reintroducir ese problema**, y además en un producto que se usa a
pleno sol.

**Lo que sí se adopta de la propuesta:** la **rampa de 11 neutros** (el kit no tiene ninguna, y resuelve
`border` y los estados apagados) y el **4.º nivel de superficie en oscuro** (hoy hay 2, y una hoja sobre
una tarjeta necesita distinguirse).

---

## 5. Contraste — la tabla completa (26 parejas medidas)

### Modo claro: 16 de 16 en AA o mejor

| Pareja | Ratio | Nivel | Nota |
|---|---|---|---|
| texto principal / fondo | 16,13 | AAA | |
| texto principal / surface | 15,03 | AAA | |
| **texto secundario / fondo** | **4,83** | AA | APCA pide 24 px+: es el más justo del modo claro |
| **texto secundario / surface** | **4,50** | AA | **clavado en el límite** |
| blanco / `primary` | 5,57 | AA | |
| blanco / `primaryPressed` | 7,78 | AAA | |
| blanco / `secondary` | 5,18 | AA | |
| blanco / `success` | 5,35 | AA | |
| blanco / `danger` | 5,62 | AA | |
| `onWarning` / `warning` | 6,97 | AA | el truco del texto oscuro sobre ámbar |
| `onInfo` / `info` | 5,01 | AA | |
| `dangerText` / fondo | 8,31 | AAA | |
| `warningText` / fondo | 9,07 | AAA | |
| `neutral` / fondo | 4,76 | AA | |
| `like` `#FF2442` / fondo | 3,76 | **falla texto** | correcto: es **uso gráfico** (el kit lo dice) |

### Modo oscuro: 5 de 5 en AAA para texto, **y el agujero de los colores de marca**

| Pareja | Ratio | Nivel |
|---|---|---|
| texto principal / fondo | 16,11 | AAA |
| texto principal / surface | 14,95 | AAA |
| texto principal / tarjeta | 14,07 | AAA |
| texto secundario / fondo | 8,04 | AAA |
| texto secundario / tarjeta | 7,02 | AAA |
| **`primary` como texto** / fondo | **3,21** | **FALLA** |
| **`success` como texto** / fondo | **3,34** | **FALLA** |
| **`danger` como texto** / fondo | **3,18** | **FALLA** |
| **`secondary` como texto** / fondo | **3,45** | **FALLA** |
| **`neutral` como texto** / fondo | **3,76** | **FALLA** |
| `warning` como texto / fondo | 8,33 | AAA (la excepción: el ámbar claro funciona en oscuro) |

**Y los 6 tokens nuevos, ya medidos sobre los tres fondos oscuros:** `primaryDark` 6,08 · `successDark`
7,15 · `dangerDark` 6,12 · `secondaryDark` 7,19 · `neutralDark` 8,94 · `warningDark` 10,14. **Ningún
fallo.**

---

## 6. Adopción por familia de token (los 4.009 usos)

| Familia | Usos | Ficheros | Literales a mano | Adopción |
|---|---|---|---|---|
| `tipografia` | 2.462 | 203 | 664 | **79 %** |
| `radios` | 702 | 168 | 724 | 49 % |
| `espaciado` | 432 | **27** | **5.446** | **7 %** |
| `peso` | 175 | 31 | 1.975 | **8 %** |
| `trazo` | 122 | 30 | 550 | 18 % |
| `icono` | 74 | 20 | — | — |
| `elevation` | 17 | 11 | — | — |
| `ilustracion` | 13 | 9 | — | — |
| `altura` | **12** | **3** | — | **~0 %** |

**Peldaños declarados que no se usaban ni una vez:** `altura.campo`, `altura.boton`, `icono.hero`.
**`altura` ya está adoptada** (24/09/2026): **37 usos en 13 ficheros**, con `campo` y `boton` en uso.

**Los 15 ficheros con más deuda** (peso · letra · radio · trazo · hex):

```
74p 37l 34r 26t 16h  app/conductor.tsx
71p 30l 20r 18t 12h  app/taxi.tsx
46p 13l 24r  4t  1h  app/lifebook-chat/[id].tsx
43p 13l 17r  9t  1h  app/profile.tsx
51p  9l 13r 12t  6h  app/edit-profile.tsx
39p 14l  8r  4t  1h  components/lifebook/GroupManageSheet.tsx
34p 12l  9r  3t 15h  app/lifebook-post/[id].tsx
34p  9l 13r 11t  1h  app/food-rider.tsx
28p  9l 14r 10t  2h  app/food-orders.tsx
28p 13l  6r  6t  1h  app/lifebook-carrito.tsx
28p 11l  9r  7t  2h  app/lifebook-user.tsx
27p 11l  8r 10t  1h  app/food-owner.tsx
22p 10l 14r  5t  0h  app/work-detail.tsx
24p 11l  7r 14t  0h  app/lifebook-hotel.tsx
29p  6l 11r  6t  1h  components/DriverHomeSheet.tsx
```

**El patrón:** la deuda se concentra en **los módulos antiguos** (`conductor`, `taxi`, `lifebook-*`,
`food-*`, `profile`) y **no** en el Mercado, que es donde se ha trabajado el diseño. El Mercado ya está
migrado; lo que queda es el resto del producto.

---

## 7. Lo que NO está medido (y no se inventa)

1. **Rendimiento.** La suite dio 8,5/10 sin medir nada. No hay número real: no se puntúa.
2. **`font_scale` distinto de 1,15.** Toda la geometría está calibrada con el 1,15 del móvil de
   referencia. A 1,3 la rejilla pide 102 dp de rótulo y crece (`minHeight`), pero **no se ha mirado**.
3. **`movimiento.ts`.** 150 líneas de tokens de animación; **no se ha contado su adopción**. Se intuye
   baja por el contexto, pero no está medido, así que no se afirma.
4. **Las 110 familias de la rejilla, una a una.** Se vieron 35 rótulos en el móvil, los 5 más difíciles
   incluidos. El «0 de 110» es del medidor, validado en esos 35.
5. **`useWindowDimensions`.** No se ha comprobado si alguna pantalla reacciona al ancho. El 5/10 de
   Responsiveness es por la rejilla fija, no por un censo de pantallas adaptables.
6. **Los 39 controles del censo táctil, uno a uno.** El censo es automático y **no distingue el `Pressable`
   del icono que lleva dentro**: son candidatos, no 39 fallos confirmados. Y falta medir el `gap` de cada
   fila, que es lo que fija el alcance máximo de `hitSlop` (`AUDITORIA-UIUX-TOKENS.md` §7.3).

---

## 8. Fuentes de cada número

| Fuente | Qué dio |
|---|---|
| `uiux_check_contrast` | Las 26 + 14 parejas, WCAG y APCA |
| `uiux_generate_palette` / `_type_scale` / `_tokens` / `_spacing_scale` | La propuesta, y la prueba de que no se adopta |
| `uiux_score_overall` | 6,2 / «Adequate» |
| `uiux_audit_log` + `uiux_audit_report` | El registro de los 7 hallazgos (ids `typography-0`, `layout-1`, `color-2`, `interaction-3`, `accessibility-4`, `color-5`, `components-6`) |
| `uiux_scan_project` / `uiux_extract_*` / `uiux_audit_run` / `uiux_score_dimension` | **Evidencia de la limitación**: solo leen CSS |
| Censo propio `_a1-censo-diseno.py` | 310 ficheros, 4.009 usos, 208 con token |
| `pruebas/verifica-diseno.cjs` | La deuda de partida: `hex 189 · fontSize 664 · borderRadius 724 · fontWeight 1975 · borderWidth 550 · espaciado 5446` |
| `grep` sobre 7 zonas | Las distribuciones de §3.2, §3.3, §3.4 |
| Teléfono real (PKK110, 1080×2374 @ 480 dpi) | 3 px/dp exactos, `font_scale` 1,15 |
