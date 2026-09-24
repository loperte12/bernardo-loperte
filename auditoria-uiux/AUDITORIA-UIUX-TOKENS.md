# Auditoría UI/UX — EG Route Plan · FASE 3a: los tokens nuevos

**Principio de este documento:** los tokens no se inventan, **se reconcilian**. Donde la escala declarada
coincide con lo que el producto usa, se migra sin tocar el valor. Donde no coincide, **manda el producto**
y se declara el valor que ya está en el código, para no cambiar un píxel.

Cada peldaño de aquí abajo lleva al lado **cuántos literales lo piden**. Un peldaño sin número no entra.

---

## 0. DECISIONES TOMADAS Y APLICADAS — 24/09/2026

Este bloque se añade **después**: el documento de arriba es la medición, y aquí queda lo que se decidió
sobre ella. Todo lo de esta sección **ya está en el código** y verificado.

| Decisión | Se eligió | Coste real, medido |
|---|---|---|
| Tipografía | **Propuesta A** (`micro 10,5` · `display 26`) | 359 usos de token mueven −0,5 px · las cifras hero −2 px · **261 literales dejan de ser excepción** |
| Los 214 literales de `600` | **`medio` (500)** | baja un paso · ver §3.1, donde hay un hallazgo nuevo que lo refuerza |
| Radios nuevos | **`marca` · `chip` · `campo` · `tarjeta`** | los 4 existentes (`sm/md/lg/full`) **no se tocan** |
| Espaciado | base 2 **+ corrección**: ver §4.1 | la primera versión cubría el 82,3 %; la corregida, el **95,7 %** |

**Y una corrección a este mismo documento**, porque la medición se hizo en dos pasadas y la primera fue
incompleta: en §4 se afirmaba que declarar la base 2 convertía «2.957 excepciones en usos legítimos».
Cierto para los múltiplos de 4, **pero la base 2 deja fuera los impares** — y los impares del proyecto no
son ruido: **3 (181 literales), 7 (168) y 5 (101)**. La corrección está en §4.1.

### 0.1 Lo aplicado, fichero por fichero

| Fichero | Qué se hizo | Puerta |
|---|---|---|
| `packages/ui-kit/src/theme/colors.ts` | +6 `*Dark` · +`sheet` · +rampa `neutro` | `tsc` 0 |
| `packages/ui-kit/src/theme/escalas.ts` | `micro 10,5` · `cuerpo 15` · `subCabecera 17` · `display 26` · `peso.maximo` · `espaciado` (20 peldaños) · `radios` +4 · `trazoIcono` | `tsc` 0 |
| `packages/ui-kit/src/index.ts` | exporta `neutro` · `trazoIcono` · `type TrazoIcono` · `ScreenHeader` | `tsc` 0 |
| `pruebas/verifica-diseno.cjs` | + regla `ESPACIADO` (era el mayor punto ciego) | — |
| `.diseno-baseline.json` | regenerado **en el mismo paso** que la regla nueva, y bajado dos veces más (§9.1 y §9.2) | `diseno` OK |
| **450 usos** de `espaciado` en **34 ficheros** | renombrados por codemod (`xs→e4` … `xxl→e32`), **0 claves viejas** | `tsc` 0 |
| `app/lifebook-carrito.tsx` | la casilla: Pressable a `altura.punto` (44) con el círculo de 21 dentro | `tsc` 0 |
| `components/lifebook/ui/Sheet.tsx` + 19 sitios | la geometría de la lámina inferior, exportada y extendida (§9.1) | `tsc` 0 |
| `packages/ui-kit/src/primitives/ScreenHeader.tsx` | **la cabecera de pantalla unificada (§9.2)**, con el piloto en `app/food-orders.tsx` | `tsc` 0 |

**Las tres puertas del cierre de la Fase 1, medidas entonces** (la base de hoy, más baja, está en §9.2):

```
npx tsc --noEmit   → 0 errores
npm run diseno     → OK: ninguna zona ha empeorado
                     hex 189 · fontSize 664 · borderRadius 724 · fontWeight 1975 ·
                     borderWidth 550 · espaciado 5446 · precioFigura 12
npm run rutas      → navegación en orden: sin enlaces rotos, mapa al día, 139/542 por el ayudante
```

---

## 1. Color · lo único que se AÑADE (nada se cambia)

### 1.1 Los 6 tokens de texto sobre fondo oscuro — el hallazgo crítico §3.1

```ts
/* ---------------------------------------------------------------------------------------------
   TEXTO DE MARCA SOBRE FONDO OSCURO.
   POR QUÉ EXISTEN: los colores de marca se diseñaron para LLEVAR texto blanco encima (y ahí cumplen).
   Usados como TEXTO o ICONO sobre el fondo oscuro #17171A suspenden el mínimo AA de 4,5:1:
   primary 3,21 · success 3,34 · danger 3,18 · secondary 3,45 · neutral 3,76.
   El kit ya tenía la variante del caso contrario (dangerText #991B1B, warningText #78350F, «para fondos
   claros»); lo que faltaba era la de fondos oscuros. Estos seis la cierran.
   REGLA: los tokens base NO se tocan — se usan como RELLENO o con texto blanco encima. Estos, solo
   como texto o icono sobre oscuro. Medidos con el comprobador de contraste, sobre #17171A / #1E1E23 /
   #232329.
   --------------------------------------------------------------------------------------------- */
primaryDark:   '#4D9AEB',  // 6,08 · 5,31 · ~5,0
successDark:   '#45B87A',  // 7,15 · 6,24 · ~5,9
dangerDark:    '#F26D6D',  // 6,12 · 5,35 · ~5,0
secondaryDark: '#F08A4B',  // 7,19 · 6,28 · ~5,9
neutralDark:   '#B0B8C4',  // 8,94 · 8,30 · 7,81
warningDark:   '#F5B942',  // 10,14 · 8,86 · ~8,4
```

**Esfuerzo `small`** (6 líneas + sustituir los usos como texto en oscuro). **Riesgo:** bajo — es aditivo,
nada existente cambia de valor.

### 1.2 La rampa de neutros — lo único que se adopta de la propuesta generada

El kit **no tiene rampa de neutros**: tiene `textPrimary`, `textSecondary`, `border` y `surface` sueltos.
La propuesta generada trae 11 pasos que sí resuelven tres cosas que hoy no tienen color: `border` en
oscuro, los estados deshabilitados y los separadores.

```ts
/** Rampa de neutros, adoptada de la propuesta generada. El kit no tenía ninguna. */
export const neutro = {
  n100: '#FAFAFA', n200: '#F2F2F2', n300: '#E5E5E6', n400: '#D0D1D2',
  n500: '#B1B2B4', n600: '#898B8F', n700: '#686A6F', n800: '#4F5155',
  n900: '#3B3C40', n1000: '#2C2D30', n1100: '#18191B',
} as const;
```

**Ojo, con dos avisos que la propia medición impone:**
- `n600 #898B8F` **no vale como texto secundario** sobre fondo claro: da 3,27 y falla. Para eso se queda
  `textSecondary #6B7280` (4,83). La rampa es para **bordes, separadores y rellenos apagados**.
- `n1100 #18191B` **no es el fondo de la app**: el fondo oscuro es `#17171A` por decisión del dueño
  («modo oscuro sin negro puro»). La rampa no lo sustituye.

### 1.3 El cuarto nivel de superficie en oscuro

Hoy `darkColors` tiene `background #17171A`, `surface #1E1E23`, `card #232329`. Falta el nivel de la
**hoja** (un `Sheet` sobre una tarjeta): sin un cuarto valor, la hoja se ve igual que lo que tapa.

```ts
// en darkColors
sheet: '#2E3338',   // 4.º nivel: la hoja sobre la tarjeta (#232329). Antes no existía y se usaba card.
```

### 1.4 Lo que NO se adopta, y por qué (tabla de decisión)

| Elemento generado | Valor | Medición | Decisión |
|---|---|---|---|
| `success` | `#22c365` | blanco encima = **2,32 falla** | **Rechazado.** Se queda `#1E7A45` (5,35) |
| `warning` | `#ec9c13` | blanco encima = **2,25 falla** | **Rechazado.** Se queda `#F59E0B` + `onWarning` (6,97) |
| `info` | `#2d7fd2` | blanco encima = **4,14 falla** | **Rechazado.** Se queda `#0EA5E9` + `onInfo` (5,01) |
| neutro 600 como texto 2.º | `#898b8f` | 3,27 en claro = **falla** | **Rechazado** para texto; **aceptado** para borde/relleno |
| `error` | `#d61f1f` | 5,15 AA | **Indiferente** (no mejora `#C62828`, 5,62) → se mantiene el actual |
| rampa de 11 neutros | 100…1100 | el kit no tenía | **Adoptada** (§1.2) |
| superficies en oscuro | 3 + overlay | el kit tiene 2 | **Adoptada** (§1.3) |

---

## 2. Tipografía · reconciliar, y con los números delante

### 2.1 Lo que el código escribe de verdad (medido, 664 literales)

| 15 | 10,5 | 17 | 14,5 | 10 | 18 | 19 | 26 | 15,5 | 22 | 9,5 | 9 | 40 | 30 | 24 | 16,5 | 34 | 38 | 21 | 8,5 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **151** | **110** | **83** | **56** | 44 | 41 | 21 | 18 | 18 | 16 | 14 | 13 | 13 | 12 | 12 | 11 | 9 | 6 | 4 | 3 |

La escala actual es `9 / 11 / 12 / 14 / 16 / 20 / 28`. **Ninguno de los cuatro valores más escritos a mano
está en ella.** Y hay seis valores fraccionarios (10,5 · 14,5 · 15,5 · 16,5 · 9,5 · 8,5) — exactamente lo
que el propio kit describe como «señal de ajuste a ojo pantalla por pantalla».

### 2.2 Propuesta A — **ELEGIDA**: 9 peldaños, 0 píxeles movidos en 234 literales

```ts
export const tipografia = {
  rotulo:      9,    // 27 literales (9 + 9,5) + la rejilla de familias. Ya existe.
  micro:      10.5,  // 154 literales (10,5 + 10). HOY 11 → se alinea con el código.
  caption:    12,    // token, 1.096 usos. No se toca.
  body:       14,    // token, 833 usos + 56 literales de 14,5.
  cuerpo:     15,    // NUEVO — 151 literales, el valor MÁS escrito del proyecto.
  subtitle:   16,    // token, 125 usos. No se toca.
  subCabecera:17,    // NUEVO — 83 literales.
  title:      20,    // token. No se toca.
  display:    26,    // 18 literales de 26 + 16 de 22. HOY 28.
} as const;
```

**Qué mueve esta propuesta, en píxeles:**

| Cambio | Sitios afectados | Movimiento |
|---|---|---|
| `micro` 11 → 10,5 | 359 usos de token + 44 literales | −0,5 px (**imperceptible**) |
| `display` 28 → 26 | usos de token de `display` (cifras grandes) | −2 px (visible, pero son cifras hero) |
| `subtitle` 16 → 17 | 125 usos de token | +1 px |

**Los 234 literales más frecuentes —151 de `15` y 83 de `17`— no se mueven ni un píxel**: se les da casa.
Y con los **27 de `rotulo 9`**, que ya la tenían, hay **261 literales que dejan de ser una excepción
declarada** y pasan a ser uso legítimo de la escala. Eso es lo que compra la propuesta A.

*Nota de aritmética, porque el documento se corrige a sí mismo:* el borrador anterior decía «8 peldaños»
en A y «7» en B. **Las dos tienen 9** —`rotulo · micro · caption · body · cuerpo · subtitle · subCabecera
· title · display`—, y el «226» del borrador no cuadraba con sus propios 151 + 83 = 234. Los peldaños y
los literales que se citan aquí son los que están en el código aplicado.

### 2.3 Propuesta B (no elegida): 9 peldaños, 0 píxeles movidos en ningún token

```ts
export const tipografia = {
  rotulo: 9, micro: 11, caption: 12, body: 14,            // los cuatro de hoy, sin tocar
  cuerpo: 15, subCabecera: 17,                            // los dos NUEVOS que el código ya usa
  subtitle: 16, title: 20, display: 28,                    // los de hoy, sin tocar
} as const;
```

Se acepta que `micro` no cubre los 154 sitios de 10,5 y 10 —viven como excepción declarada para
siempre— y a cambio **ningún token existente cambia de valor**. Tiene 9 peldaños igual que A: la
diferencia no es el número de peldaños, son **dos valores**.

**DECIDIDA: A** (24/09/2026). B queda documentada a propósito, porque el argumento que la sostenía
—«ningún token existente cambia de valor»— es legítimo y conviene tenerlo a mano si al verlo en el móvil
el `display` a 26 se quedara corto para las cifras de precio. Lo que estaba medido, y es lo que faltaba,
es el coste de cada una.

### 2.4 La escala no cubre las cifras hero, y no debe

`40` (13), `38` (6), `34` (9) = 28 literales muy grandes. Casi con seguridad son **cifras de efecto**
(estado vacío, doble toque del corazón) — el mismo caso que ya obligó a crear `ilustracion` («un emoji de
56 no es letra grande»). **No entran en `tipografia`.** Si se confirma qué son, van a su propia escala
con nombre propio (p. ej. `cifraHero`), igual que se hizo con `ilustracion`.

---

## 3. Peso · un solo peldaño nuevo conserva 1.759 de 1.975 literales

| Valor a mano | 800 | 900 | 700 | 600 | 500 | 400 |
|---|---|---|---|---|---|---|
| veces | **727** | 532 | 500 | 214 | 2 | **0** |

```ts
export const peso = {
  normal: '400',   // sin usos hoy, y el kit lo añadió justificándose. Se queda: es el texto base.
  medio:  '500',
  fuerte: '700',
  maximo: '800',   // NUEVO — el valor MÁS escrito del proyecto (727). Sin él no se puede migrar.
  titulo: '900',
} as const;
```

**El mapeo y su coste:**

| Literal | → Token | Sitios que se mueven |
|---|---|---|
| 800 (727) | `maximo` | **0** |
| 900 (532) | `titulo` | **0** |
| 700 (500) | `fuerte` | **0** |
| 500 (2) | `medio` | **0** |
| **600 (214)** | `medio` o `fuerte` | **214 → decisión de diseño** |

**1.759 de 1.975 literales pasan a token sin mover un píxel.** Los 214 de `600` son la única decisión.

**DECIDIDA: `medio` (500)** (24/09/2026). Y al ir a aplicarla apareció un dato que **refuerza la decisión
por una razón técnica, no estética** (§3.1). Queda un aviso honesto: **estos 214 son los únicos literales
de toda la migración cuyo aspecto cambia**, así que son los únicos que hay que mirar en el móvil.

### 3.1 `600` es el único peso cuyo aspecto decide el aparato

Medido en el proyecto, y no era parte del encargo original:

- **`fontFamily`: cero declaraciones en todo el código.** No hay ni una.
- **`expo-font` está instalado (`package.json`, `~/.3`)** pero **nunca se llama**: `useFonts` y
  `Font.loadAsync` tienen **0 apariciones**.

Es decir: **la aplicación no elige su tipografía.** Usa la del sistema operativo y deja el dibujo de las
letras en manos del aparato. Consecuencia directa sobre la escala:

| Peso | ¿Roboto lo tiene? | Cómo se dibuja |
|---|---|---|
| `400` `500` `700` `900` | **sí** (Regular, Medium, Bold, Black) | idéntico en todas partes |
| **`600`** | **depende**: existe en Roboto variable (Android 12+) y en SF Pro; **no** en Roboto clásico | **donde no existe, el sistema lo ajusta al más cercano — 500 o 700** |

O sea: **`600` es el único valor del peso que no tiene una forma garantizada.** Los otros cuatro son
iguales en cualquier Android y en cualquier iPhone. Retirarlo no es solo «abrir la franja ligera que el
kit echaba de menos»: es **quitar de la escala el único peldaño que no se puede prometer**.

**Y dice dónde hay que ir a mirarlo, que es el otro dato nuevo:**

| Módulo | Literales de `600` |
|---|---|
| **Mercado** (`app/ecomerse*`, `components/ecomerse/*`) | **0** |
| Transporte y perfil (`conductor` 16 · `edit-profile` 15 · `intercity` 11 · `profile` 10 · `DriverHomeSheet` 9 · `trips-history` 8 · `taxi` 8 · `work-detail` 6 · `intercity-publish` 5 · `driver-profile` 5…) | **106 de 173 en `app/`** |

Cero en el Mercado. La decisión del peso **no hay que verificarla en el Mercado** —allí no hay ni un
`600`— sino en las pantallas de **Transporte y perfil**, que son un puñado de ficheros y se recorren en
una tanda. Eso convierte la verificación de «media app» en «un módulo».

---

## 4. Espaciado · la familia que exige cambiar la base

| px | 8 | **10** | 12 | 6 | **14** | 16 | 4 | **2** | **3** | **7** | 18 | 5 | 20 | 24 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| veces | 834 | 791 | 754 | 563 | 400 | 352 | 337 | 273 | 142 | 136 | 117 | 91 | 81 | 78 |

**2.489 múltiplos de 4 · 2.957 (54 %) no.** La escala declara `4/8/12/16/24/32`.

```ts
/* ---------------------------------------------------------------------------------------------
   ESPACIADO — BASE 2, no 4.
   POR QUÉ CAMBIA LA BASE: la escala declaraba múltiplos de 4 (4/8/12/16/24/32) y tiene 432 usos frente
   a 5.446 literales escritos a mano, de los que 2.957 (54 %) NO son múltiplos de 4. El segundo valor
   más usado después del 8 es 10, que la escala no contiene. En una rejilla densa de marketplace a
   360 dp, con celdas de 80 dp, la granularidad de 4 px es demasiado gruesa: entre 8 y 12 no hay nada y
   hay sitios donde 8 aprieta y 12 sobra. La escala declaraba una base que el producto no usa.
   Declarar la base real convierte 2.957 «excepciones» en usos legítimos SIN TOCAR UN PÍXEL.
   Se nombra por el valor a propósito: en espaciado el número ES la intención (nadie dice «un hueco
   semántico de 10»). Renombrar aquí es lo más barato de las cinco familias porque casi nadie la usa.
   --------------------------------------------------------------------------------------------- */
export const espaciado = {
  e2: 2, e4: 4, e6: 6, e8: 8, e10: 10, e12: 12, e14: 14, e16: 16,
  e20: 20, e24: 24, e32: 32,
} as const;
```

**La alternativa si se insiste en la base 4:** mover los **791 usos de 10** a 8 o a 12. Eso sí cambia el
aspecto de la app entera: 791 sitios son demasiados para decidirlos sin mirar. **No la recomiendo.**

### 4.1 CORRECCIÓN — la base 2 deja fuera los impares, y eran 450 literales

La primera versión de esta sección proponía una base 2 con 11 peldaños y se aplicó así. Al medir **qué
quedaba fuera** apareció el fallo: **una base 2 no puede expresar un número impar**, y los impares del
proyecto no son ruido de nadie.

Medición sobre **6.566 literales** de `padding`/`margin`/`gap` en todo el proyecto menos `node_modules`
(alcance algo mayor que el de la guardia, que ve 5.446; los porcentajes sí son comparables):

| Escala declarada | Cubre | Queda fuera |
|---|---|---|
| **base 4** (la original) | 2.880 = **43,9 %** | 3.686 |
| **base 2** (1.ª versión) | 5.406 = **82,3 %** | 1.160 |
| **base 2 + impares + 18/22** (aplicada) | 6.284 = **95,7 %** | **282** |

Los valores que la base 2 dejaba fuera por ser impares, con su uso real medido:

| valor | literales | valor | literales |
|---|---|---|---|
| **3** | **181** | 9 | 79 |
| **7** | **168** | 13 | 57 |
| **5** | **101** | 11 | 90 |

**450 literales** con uso alto. Y no eran capricho: en una rejilla densa de marketplace —celdas de 80 dp
a 360 dp— el hueco entre el icono y su rótulo, o el aire dentro de un chip, se resuelve en **3, 5 o 7**,
no en 4 ni en 8. **Declarar base 2 y dejar 450 usos fuera era repetir el error original con otro
número: declarar una base que el producto no usa.**

Lo aplicado, entonces:

```ts
export const espaciado = {
  e2: 2, e3: 3, e4: 4, e5: 5, e6: 6, e7: 7, e8: 8, e9: 9, e10: 10,
  e11: 11, e12: 12, e13: 13, e14: 14, e16: 16, e18: 18, e20: 20, e22: 22, e24: 24, e32: 32,
} as const;
```

**Y hay que decir en voz alta lo que esto es:** ya **no es una progresión, es el conjunto de los valores
en uso, con nombre y contados uno a uno.** Es deliberado. Lo que aporta nombrarlos no es restringir
—no restringe— sino que **se puedan contar y cambiar en un sitio**, que es justo lo que la guardia no
podía hacer mientras eran literales sueltos.

**Los 282 que siguen fuera, y qué son:** **77 son `1`** —un nudo de alineación de un píxel, no un hueco— y
el resto son anchos de contenedor grandes (26, 28, 30, 40, 48, 60, hasta 130) que **no son «espaciado»
sino tamaño de caja**, y deben ir a su propia familia el día que se cree. No se inventan peldaños para
ellos: **un peldaño sin literales que lo pidan no entra.** (El recuento definitivo, con el patrón exacto
de la guardia, es el del plan de la Fase 3: **245 de 5.459**.)

**Y la guardia quedó arreglada en el mismo paso:** `verifica-diseno.cjs` contaba cinco familias y **no
contaba el espaciado** — un trinquete que no ve el espaciado no aprieta el espaciado. Se añadió la regla
`(padding|margin|gap)[A-Za-z]*:\s*\d+`, se marcó `espaciado` como familia auditada y **se regeneró la
base en el mismo commit**, para que los 5.446 literales nazcan *registrados* y no como 5.446 fallos.

---

## 5. Trazo · la única familia donde la escala acertó

| Valor | 1 | 1,5 | 2 | 3 | 0 | 1,2 | 0,6 |
|---|---|---|---|---|---|---|---|
| veces | **450** | 66 | 19 | 6 | 5 | 3 | 1 |

`fino 1` / `base 1,5` / `fuerte 2` / `marcado 2,5` **ya cubre el 97 %** de los literales: 450 son
exactamente `fino`. **No se cambia ni un valor.** Falta una cosa:

```ts
/** Escala de trazo de ICONO (strokeWidth), que el kit nunca declaró y el producto sí usa:
    hay 1.8, 2, 2.2 y 3 escritos a mano. Sin esto, la escala de trazo está a medias. */
export const trazoIcono = {
  fino: 1.8,     // icono pequeño y denso
  base: 2,       // icono de fila o botón (el más común)
  fuerte: 2.2,   // icono de acción principal
  marcado: 3,    // icono de estado o de acento
} as const;
```

---

## 6. Radios · conservar los 4 y sumar los 2 que el código pide

| Valor | 14 | 10 | 22 | 18 | 20 | 9 | 6 | 4 | 24 | 26 |
|---|---|---|---|---|---|---|---|---|---|---|
| veces | **168** | **142** | 67 | 52 | 47 | 32 | 32 | 28 | 25 | 14 |

La escala tiene `sm 8 / md 12 / lg 16 / full 999` — y **está bien adoptada** (702 usos, 168 ficheros, una
de las mejores). El problema es que le faltan los dos valores más usados a mano: **14 (168)** y **10
(142)**, 310 literales entre los dos. **Los cuatro existentes no se tocan** (tocarlos movería 700 usos).

```ts
export const radios = {
  marca: 6,      // NUEVO — 60 literales (4 + 6). Sellos y micro-etiquetas.
  sm: 8,         // existente, 97 usos
  chip: 10,      // NUEVO — 142 literales
  md: 12,        // existente, 299 usos
  campo: 14,     // NUEVO — 168 literales, el valor MÁS escrito
  lg: 16,        // existente, 107 usos
  tarjeta: 20,   // NUEVO — 99 literales (18 + 20 + 22)
  full: 999,     // existente, 199 usos
} as const;
```

**Nombres CONFIRMADOS** (24/09/2026): `marca`, `chip`, `campo`, `tarjeta`. Siguen la regla del kit
—nombre de intención, no de tamaño— y ya están en el código.

---

## 7. Altura · no hay nada que añadir, hay que USARLA

`altura` tiene `punto 44 / control 46 / campo 50 / boton 52`. **Está bien diseñada.** Cuando se midió
(23/09/2026) tenía **12 usos en 3 ficheros** y **`campo` y `boton` no se usaban ni una vez**. No se toca:
se adopta.

**ADOPTADA el 24/09/2026 (Fase 2).** Ahora tiene **37 usos en 13 ficheros** y los cuatro peldaños están en
uso. Cómo se hizo, porque el detalle importa:

- `altura.boton` (52) → `PrimaryButton.tsx` y la tecla grande de `app/conductor.tsx` (76×52).
- `altura.campo` (50) → `FormField.tsx` (la caja de una línea), los tres CTA del flujo de hotel
  (`lifebook-hotel-detalle`, `-fechas`, `-reservar`) y la caja de ciudad de `food-owner.tsx`.
- `altura.control` (46) y `altura.punto` (44) → los formularios de las cinco pantallas de reserva de hotel,
  que ya escribían esos valores a mano.

**Cero píxeles movidos:** cada literal se sustituyó por el peldaño que vale exactamente lo mismo.
Medidor: `_a3-codemod-altura.py` (15 sitios, simulación antes de escribir).

**Y aparece una inconsistencia que la escala NO resuelve, y hay que decirla:** el flujo de hotel tiene
**tres alturas de campo distintas para la misma pieza** — 44 en `lifebook-hotel.tsx` y `-reservas.tsx`,
46 en `-reserva.tsx` y `-reservar.tsx`, 50 en los CTA de `-detalle.tsx` y `-fechas.tsx`. Migrarlas a
`punto`/`control`/`campo` **conserva el valor y por tanto conserva la inconsistencia**: la escala queda
adoptada, no unificada. Unificar las tres es un cambio visual de 2 a 6 px sobre formularios de pago y
**es decisión de Bernardo**, no una consecuencia automática de la escala.

### 7.1 Censo de blancos táctiles — la primera cifra era una nota, no una medición

**Corrección.** El borrador de este documento decía «los dos sitios por debajo del mínimo: la casilla del
carrito (21 dp) y el micrófono del buscador (28 dp)». La casilla era correcta, en
`app/lifebook-carrito.tsx` (`casilla: 21×21`), pero **el «micrófono del buscador» no existe**: no hay
ningún micrófono en el código. Era una nota sin verificar y se retira.

El censo real —objetos de estilo **con `width` Y `height` explícitos menores de 44 dp**, en todos los
`.ts/.tsx` del proyecto menos `node_modules` y las copias de `.auditoria-servicios/`:

| | |
|---|---|
| Objetos de estilo por debajo de 44 dp | **152** |
| De ellos, **nombres de control** (btn, casilla, step, qty…) y no decoración (avatar, logo, punto…) | **39** |

**Los 39, por módulo** — el orden dice dónde está el trabajo:

| Módulo | Controles < 44 dp | Ejemplos |
|---|---|---|
| **lifebook** | **15** | `casilla` **21×21** · `check` 20 y 24 · `closeBtn` 22 · `moreBtn` 24 · `iconBtn` 32–38 |
| food | 3 | `iconBtn` **28** · `stepBtn` **28** · `stepBtn` 30 |
| conductor / taxi / work | 7 | `swapBtn` **24** · `mapPickBtn` **28** · `contactBtn` 36 · `roundBtn` 36 |
| ecomerse | 4 | `qtyBtn` 34 · `enviar` 40 · `iconButton` 36 · `menuGhostBtn` 30 |
| ui-kit y otros | 10 | `DocumentChoiceTree:check` 24 · `RoundBtn` 36 · `tile` 38 |

**Dos avisos sobre esta tabla, porque un censo también tiene límites:**

1. **No distingue el Pressable del icono de dentro.** Un `closeBtn: { width: 22, height: 22 }` puede ser
   el botón o el icono que va dentro de un botón mayor. **Los 39 son candidatos, no 39 fallos
   confirmados**; hay que abrir cada uno.
2. **No mide lo que no está declarado.** Un `Pressable` sin `width`/`height` que solo envuelve un icono
   mide lo que mida el icono, y aquí no se ve.

### 7.2 El arreglo, y por qué `hitSlop` no vale

`hitSlop` agranda el área que **responde**, pero **no lo que el usuario ve ni a lo que apunta**. Un
círculo de 21 dp con `hitSlop={8}` responde en 37 dp mientras el dedo sigue yendo a un blanco de 21. La
regla queda escrita así:

```ts
/** REGLA DE ALTURA TÁCTIL.
 *  Ningún elemento tocable mide menos de `altura.punto` (44 dp). `hitSlop` NO cuenta: agranda el área
 *  táctil pero el usuario sigue VIENDO y apuntando a un cuadrado de 21 dp. En un mercado al sol, con
 *  el móvil en una mano y el dedo gordo, eso es un toque que se falla.
 *  Excepción legítima: iconos decorativos no tocables, y el corazón de doble toque (efecto, no control). */
```

**Y el patrón con el que se arregla, que es el que se aplicó a la casilla del carrito
(`app/lifebook-carrito.tsx`, 24/09/2026):** separar las dos cosas.

- el **`Pressable` mide 44 dp** (`altura.punto`) → es lo que el dedo encuentra;
- el **círculo sigue midiendo 21** y va centrado dentro mediante un `View` → **el diseño no cambia**;
- se **retira el `hitSlop`**, que era el parche que ocultaba el problema.

Funciona cuando el control va **suelto**, como la casilla. **No funciona cuando el control va en una fila
con `gap`**, como el stepper de `food-checkout`: al envolver en 44 dp, la separación *visible* entre las
cajas de color pasa de 8 a 24 dp porque el `gap` se mide entre las cajas de 44, no entre las de 28. Ese
caso **no se cambia a ciegas**: subir la caja visible a 44 sí altera el diseño, y eso es una decisión de
Bernardo mirando la pantalla, no una consecuencia automática de la escala.

### 7.3 La decisión de Bernardo (24/09/2026), y la regla aritmética que la generaliza

**Decisión tomada: el stepper de `food-checkout` se queda como está.** Prefiere mantener el diseño visual
actual antes que ganar 16 dp de dedo. Cierra ese caso y, sin querer, fija el criterio del resto.

Y al comprobarlo apareció el motivo por el que ese caso **no es aislado: es aritmético.** En una fila con
`gap`, un `hitSlop` **no puede llegar a 44 dp sin invadir al vecino**. El área táctil crece `2·s`, la
separación entre dos controles es `gap`, y sin solaparse el máximo es `s = gap/2`. Por tanto:

> **alcance máximo sin invadir = tamaño del control + gap de la fila**

**Caso medido, `app/ecomerse-detail.tsx`:** la fila `acciones` tiene `gap: 10` y el corazón vive en un
`iconCol` de **26×22 dp** (icono de 22 + `paddingHorizontal: 2`). Con `hitSlop={8}` responde en **42×38**
—ninguno de los dos llega a 44—, y esos 8 dp **ya invaden 3 dp** del botón de WhatsApp, porque el `gap`
solo admite 5 por lado. Para llegar a 44 harían falta 9 dp por lado (ancho) y 11 (alto), cerca del doble
de lo que la fila admite.

| Tamaño en la fila | `gap` | Alcance máximo sin invadir | ¿Llega a 44? |
|---|---|---|---|
| 38 dp | 8 | **46** | **sí** |
| 34 dp | 8 | 42 | no |
| 32 dp | 8 | 40 | no |
| 26 dp | 10 | 36 | no |
| 22 dp | 10 | 32 | no |

**Lo medido y lo inferido, separados:** la tabla es aritmética sobre valores leídos del código. Que el
solape haga «ganar» el toque al hermano siguiente depende del orden de dibujo de `Pressable` y **no se ha
medido en el aparato**; se enuncia como riesgo, no como hecho.

**Conclusión, y es la que ordena lo que queda de la fase:** del censo de 39 controles, **solo la clase de
38 dp en filas con `gap` 8 alcanza 44 dp sin tocar el diseño** — y para eso hay que poner `hitSlop` **por
lado** (`{top:4,bottom:4,left:4,right:4}`), no un número suelto, que es lo que haría invadir. Todo lo demás
exige **o** separar más las filas **o** agrandar la caja visible: las dos son cambios de diseño, y con la
decisión de arriba ya tomada, **se quedan como están y se documentan.**

Por eso la Fase 2 cierra este punto con **censo + regla + una decisión**, no con 38 parches. Un `hitSlop`
que invade al vecino no es una mejora: es un toque que acierta el control de al lado.

---

## 8. Resumen de esfuerzo por token

| Familia | Tokens nuevos | Valores existentes que cambian | Literales que pasan a token sin moverse | Esfuerzo |
|---|---|---|---|---|
| Color | 6 (+ rampa de 11 + 1 superficie) | **0** | — | `small` |
| Peso | **1** (`maximo 800`) | 0 | **1.759 de 1.975** | `large` (migrar) |
| Tipografía | 2 (`cuerpo 15`, `subCabecera 17`) | 2–3 según A o B | 234 | `large` (migrar) |
| Espaciado | **20 peldaños** (base 2 → corregido en §4.1: base 2 no puede expresar impares, y eran 450 literales) | **cambia la base** | 5.446 → cobertura 95,7 % de los impares incluidos | `large` (migrar) |
| Trazo | 4 de icono | 0 | 535 de 550 | `medium` (migrar) |
| Radios | 4 | 0 | 310 | `medium` (migrar) |
| Altura | 0 | 0 | — | `medium` (auditar los < 44 dp) |

---

## 9. Formas repetidas · la geometría que estaba copiada

**Esta sección no existía, y era un hueco del método.** Las siete anteriores son familias de *valores*
(color, tamaño, peso, espaciado, trazo, radio, altura). Pero el trinquete cuenta literales y el censo
`_a1` cuenta adopción de tokens: **ninguno de los dos ve una forma repetida**, porque una forma repetida
no gasta ningún literal — gasta *sitios*. Diecisiete ficheros escribiendo siete valores idénticos suman
cero deuda a mano y, sin embargo, son diecisiete sitios que hay que editar a la vez.

Medidor nuevo: **`_a5-censo-formas-repetidas.py`**. Extrae cada `StyleSheet.create` del proyecto, parte
cada entrada `nombre: { … }` y compara **solo las declaraciones** —no el nombre de la clave—, con
espacios y orden normalizados. Así `handle` y `barra` con el mismo cuerpo cuentan como la misma forma.

**Resultado global, medido el 24/09/2026:** 310 ficheros · 2.831 entradas de estilo ·
**110 formas que aparecen en 3 o más ficheros**. Las cuatro mayores:

| Forma | Ficheros | Qué es |
|---|---|---|
| `alignItems:'center' flex:1 justifyContent:'center'` | 28 | el centrado de un estado vacío o de carga |
| **`header`** — `flexDirection:'row' justifyContent:'space-between' borderBottomWidth:1 paddingHorizontal:16 paddingVertical:12` | **21** | **el encabezado de pantalla** |
| **`sheet`** — absoluta pegada abajo, dos radios de 22, relleno de 18 | **17** | **la lámina inferior** |
| `topBar` — igual que `header` con `gap:10` y `paddingHorizontal:12` | 10 | la barra superior de las pantallas de lifebook |

### 9.1 La lámina inferior — CERRADA el 24/09/2026 (Fase 2)

**Hecho, y es la primera vez que se cierra una forma repetida.** Los 17 sitios tenían la misma firma
byte a byte, y **uno de los 17 era su propio dueño**: `components/lifebook/ui/Sheet.tsx` ya declaraba ese
cuerpo en `sheetStyles.sheet`, **exportado y sin que lo importara nadie**. El tirador (`width 40 / height
4 / radio 2`) estaba en 3 sitios.

Lo que se hizo no fue crear un componente —eso obligaría a tocar el marcado de 17 pantallas— sino
**exportar la geometría y extenderla**:

```ts
export const formaHoja = { position:'absolute', left:0, right:0, bottom:0,
                           borderTopLeftRadius:22, borderTopRightRadius:22, padding:18 } as const;
export const formaTirador = { width:40, height:4, borderRadius:2, alignSelf:'center', marginBottom:14 } as const;
// y cada sitio:  sheet: { ...formaHoja }   ·   handle: { ...formaTirador }
```

| | |
|---|---|
| Sitios unificados | **19** (17 hojas + 2 tiradores), en 16 ficheros + el dueño |
| De ellos, en `lifebook` | **16 de 17**; el restante es `app/settings.tsx` (`sheetCard`) |
| Trinquete antes → después | `borderRadius 724 → 688` · `espaciado 5423 → 5404` |
| Píxeles movidos | **0, y no por razonamiento: medido** |
| Medidores | `_a6-codemod-forma-hoja.py` (simulación antes de escribir) · `_a7-verifica-forma-hoja.py` |

**Cómo se demuestra que no se movió un píxel.** No basta con decir «es el mismo valor». `_a7` recupera el
cuerpo ORIGINAL de cada sitio desde el respaldo que el codemod escribe antes de tocar nada, lo compara
declaración a declaración con el que deja el código, y **resuelve `...formaHoja` leyendo la constante real
del disco**, no lo que el script crea recordar. Resultado: **19 de 19 idénticos, 0 distintos.**

**Dos trampas que aparecieron y que quedan escritas, porque las dos costaron un intento fallido:**

1. **El trinquete cuenta los literales que están DENTRO de los comentarios, y no distingue el que explica
   del que escribe.** El primer intento falló porque el bloque de comentario que explicaba el cambio citaba
   una declaración de relleno con su cifra; el segundo, porque el aviso que advertía de eso **la volvía a
   citar**. Lección para el que venga: en un fichero vigilado, los valores se explican **con palabras**.
   (El `_a3` ya lo sabía: un comentario que cita `height: 50` habla del valor, no lo escribe.)
2. **Un codemod que lee con traducción de saltos y escribe con `newline=''` convierte CRLF en LF sin
   avisar.** 11 de las 310 fuentes del proyecto están en CRLF —una de ellas, `messaging-sheets.tsx`, es de
   esta tanda— y el cambio saldría como un diff de 220 líneas donde se tocó una. Ahora el codemod detecta
   el salto dominante del fichero y escribe el mismo.

### 9.2 La cabecera de pantalla — CERRADA el 24/09/2026 (Fase 2) · `ScreenHeader`

**Era la forma repetida más extendida del proyecto: 21 ficheros**, todos en `app/`, con la firma
idéntica (`flexDirection:'row'` · `justifyContent:'space-between'` · `borderBottomWidth: 1` · relleno
horizontal de 16 y vertical de 12). En `food` es literalmente la cabecera de sus seis pantallas.

**Las dos decisiones que pedía esta sección las tomó Bernardo el 24/09/2026, y las dos en el sentido de
unificar:** el dueño **vive en el kit**, como primitiva oficial, y las 21 variantes **se absorben en un
solo componente con props**.

**El nombre `StepHeader` no se pudo usar: ya estaba tomado.** Aquí había escrito que el kit tenía
`StepHeader` y `SearchHeader`; la segunda mitad era imprecisa — **`SearchHeader` no está en el kit**, es
un componente de la app (`components/SearchHeader.tsx`). Lo que sí está en el kit es `StepHeader`, la
cabecera de un paso de flujo con barra de progreso segmentada, **sin botón de volver**, importada por 5
pantallas de KYC. Así que el nuevo se llama **`ScreenHeader`** y el de KYC no se ha tocado.

**Lo que se midió antes de escribirlo, y que desmonta la idea de «21 copias iguales»:** la firma que
comparten es la de las *declaraciones*. Por dentro, el título se escribía de cuatro maneras y los
valores no coincidían:

| | Medición |
|---|---|
| Origen del estilo del título | clave `headerTitle` **13** · en línea **8** |
| Tamaño del título | **17** en 15 · **16** en 6 |
| Peso del título | **800** en 14 · **700** en 7 |
| Icono de volver | **24** en 14 · **22** en 7 |
| `numberOfLines` | lo ponen **8** · **13** no lo ponen (y la fila crece sola) |

**El componente** (`packages/ui-kit/src/primitives/ScreenHeader.tsx`): `titulo` · `subtitulo` ·
`alVolver` (sin ella no hay botón y queda un hueco, que es lo que necesita un esqueleto) ·
`etiquetaVolver` · `pistaVolver` · `accion` · `lineasTitulo` · `style`.

Las cinco decisiones que toma, todas por mayoría medida y ninguna por gusto: título **17**
(`tipografia.subCabecera`, el de 15 de 21 — **un token que 15 ficheros escribían a mano y ni uno usaba**),
peso **`maximo`** (14 de 21), icono **`icono.lg`** (14 de 21), **alto de fila 48 sin tocar**, y el toque
del volver por **`hitSlop` de 12 por lado** — porque subir la caja a `altura.punto` llevaría la fila a 68
y cambiaría el alto de las 21 pantallas. Es la regla de §7.3 aplicada al caso que **sí** la admite: un
control suelto, no una fila con `gap`.

**Un detalle que existe por medición y no por gusto: `lineasTitulo`.** Poner `numberOfLines={1}` en todas
evita que la fila crezca, pero puede truncar. El ancho disponible son **280 dp** y a 17 le caben **22
caracteres**, según la calibración de ancho de letra ya contrastada contra el teléfono (**7,9 dp por
carácter a tamaño 11**, de `_c9-medir-letra-rejilla.cjs`). Medidor nuevo: `_b2-medir-titulos-cabecera.py`.
Solo **2 de los 21 no caben** —«Alquileres en Guinea Ecuatorial» (31 caracteres, 378 dp) y el nombre del
comercio de `food-menu`, que es dinámico— y esos dos pasan `lineasTitulo={2}`.

**Estado: el componente está escrito y el piloto aplicado en `app/food-orders.tsx`**, con las tres
puertas en verde y el trinquete bajando `espaciado 5404 → 5402`, `fontSize 664 → 663`,
`fontWeight 1971 → 1970`, `borderWidth 549 → 548`. **Y la comprobación no es de previsualización: está
medida en el móvil de referencia.** `_b4-mide-cabecera.py` descodifica los dos pantallazos a mano y
encuentra que la fila del borde no se mueve (270 px), que el trazo del icono pasa de 44 a 48 px y que su
centro vertical queda donde estaba: el icono pasa de **22,0 a 24,0 dp creciendo alrededor de su centro**
—exactamente lo previsto, porque el trazo de `ArrowLeft` ocupa 16/24 de su caja—. El reemplazo de los 20
restantes espera el visto bueno de Bernardo. El acta, con la tabla de lo que cambia en cada uno, es
`TANDA-FASE2-CABECERA-PILOTO.md`.

**Y queda un suelo, medido, para el que venga:** la guardia cuenta los literales que están dentro de los
comentarios y **no** los distingue del código. En todo el proyecto hay **8 literales de espaciado** dentro
de un comentario (0,1 %), así que el trinquete de esa familia **nunca bajará de 8**. Es ruido, no deuda —
pero conviene saberlo antes de perseguir un cero que no existe.
