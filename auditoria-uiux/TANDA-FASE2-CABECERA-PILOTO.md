# Acta · FASE 2 — la cabecera de pantalla (`ScreenHeader`)

**Fecha:** 24/09/2026 · **Rama:** `refactor/ui-ux-reconstruction` · **Fase:** 2 (Componentes Base), 6.ª pieza
**Estado:** **componente escrito y piloto aplicado.** El reemplazo de las 21 queda pendiente de la
comprobación visual de Bernardo, que es como él lo pidió.

---

## 0. La decisión, y el nombre que no se pudo usar

Bernardo decidió el 24/09/2026: **(1) el dueño vive en el kit**, como primitiva oficial, y
**(2) se unifican las 21 variantes** en un solo componente con props.

**Pero el nombre `StepHeader` ya estaba tomado.** No es una nota: existe
`packages/ui-kit/src/primitives/StepHeader.tsx` —la cabecera de un paso de flujo, con barra de progreso
segmentada y sin botón de volver— exportada en el barrel desde antes, y **5 pantallas de KYC la
importan** (`app/kyc/index.tsx`, `capture.tsx`, `liveness.tsx`, `status.tsx`). Dos componentes distintos
con el mismo nombre en el mismo barrel es un error de importación esperando a ocurrir, así que **el
nuevo se llama `ScreenHeader`** y el `StepHeader` de KYC no se toca.

Si el nombre no convence, es un renombrado de una línea (`ScreenHeader.tsx` + el `export` del barrel) y
nada más: **no hay ningún otro sitio que lo importe todavía** salvo el piloto.

---

## 1. Lo que se midió antes de escribir nada: no eran 21 copias iguales

El censo de formas (`_a5-censo-formas-repetidas.py`) encontró la firma idéntica en **21 ficheros, todos
en `app/`**. Pero la firma es de las **declaraciones**: por dentro, el título se escribía de cuatro
maneras y los valores no coincidían entre sí.

| | Medición | Ficheros |
|---|---|---|
| Origen del estilo del título | clave local `headerTitle` **13** · en línea **8** | 21 |
| Tamaño del título | **17** en 15 · **16** en 6 | 21 |
| Peso del título | **800** en 14 · **700** en 7 | 21 |
| Tamaño del icono de volver | **24** en 14 · **22** en 7 | 21 |
| `numberOfLines` | lo ponen **8** · no lo ponen **13** | 21 |
| Ancho del hueco derecho | **24** en 18 · **22** en 3 | 21 |

**La lectura:** lo que se repetía no era un diseño, era **la misma fila con cinco decisiones tomadas
por separado en cada sitio**. Y la más cargada de consecuencias es la última fila: **13 cabeceras
dejan que un título largo parta en dos líneas y la fila crezca sola**, aunque debajo tenga un borde
que la delimita.

---

## 2. El componente

`packages/ui-kit/src/primitives/ScreenHeader.tsx` — nuevo, exportado desde el barrel.

| Prop | Qué hace |
|---|---|
| `titulo` | El texto. Obligatorio |
| `subtitulo` | Segunda línea en `textSecondary`. Lo usa `alquiler.tsx` para el contador de anuncios |
| `alVolver` | **Sin ella no hay botón** y queda un hueco del ancho del icono — es lo que necesita un esqueleto de carga |
| `etiquetaVolver` | «Volver» por defecto |
| `pistaVolver` | `accessibilityHint`. Existe porque una pantalla ya lo tenía: quitar un aviso de accesibilidad es perder algo |
| `accion` | El contenido de la derecha: un icono, un botón, dos textos |
| `lineasTitulo` | **1 por defecto.** Ver §4 |
| `style` | Para el `paddingTop` de la zona segura, que no es igual en las 21 |

**Las cinco decisiones que toma, y con qué se justifican:**

| Decisión | Valor | Por qué |
|---|---|---|
| Tamaño del título | `tipografia.subCabecera` (**17**) | Es el de **15 de los 21**. Se eligió el mayoritario, no el que me gustara |
| Peso | `peso.maximo` (**800**) | El de **14 de los 21** |
| Icono | `icono.lg` (**24**) | El de **14 de los 21** |
| Alto de la fila | **48 dp** (12 + 24 + 12) | Es el de hoy. **No se toca** |
| Toque del volver | `hitSlop` **12 por lado** → 48 dp de dedo | El alto de la cabecera lo marca el icono. Subir la caja a `altura.punto` (44) la llevaría a **68** y cambiaría el alto de las 21 pantallas: eso es un rediseño, no una unificación. Es la regla de §7.3 del documento de tokens aplicada al caso que **sí** la admite —un control suelto, no una fila con `gap`— |

**Y dos tokens que llevaban meses muertos entran en uso:** `tipografia.subCabecera` (17) lo escribían a
mano 15 ficheros y **ni uno solo usaba el token**; `icono.lg` no se usaba en ninguna cabecera.

---

## 3. El piloto: `app/food-orders.tsx`

El elegido es la cabecera **canónica**: volver, título y hueco a la derecha, sin acción. Está en el
módulo del que salió la decisión (`food`) y es una pantalla de uso diario.

```tsx
// antes — 12 líneas, 5 literales vigilados
<View style={s.header}>
  <Pressable onPress={() => router.back()} hitSlop={12} accessibilityRole="button" accessibilityLabel="Volver">
    <ArrowLeft size={22} color={colors.textPrimary} />
  </Pressable>
  <Text style={s.headerTitle}>{isOwner ? 'Pedidos recibidos' : 'Mis pedidos'}</Text>
  <View style={{ width: 24 }} />
</View>

// después — 4 líneas, 0 literales
<ScreenHeader
  titulo={isOwner ? 'Pedidos recibidos' : 'Mis pedidos'}
  alVolver={() => router.back()}
/>
```

Las dos claves locales (`header`, `headerTitle`) se retiran del `StyleSheet` y el `ArrowLeft` del
import. **Lo que cambia en pantalla en este fichero, y es todo lo que cambia:**

| | Antes | Después |
|---|---|---|
| Título | 17 · peso 800 · sin tope de líneas | **idéntico**, con tope de 1 línea (cabe: 17 caracteres de 22 posibles) |
| Icono de volver | 22 | **24** (+2 px) |
| Al pulsar el volver | **nada** | el icono baja a 0,6 de opacidad |
| Alto de la fila | 48 | **48** |

---

## 4. Por qué `lineasTitulo` existe, y por qué solo dos lo necesitan

Poner `numberOfLines={1}` en todas arregla que la fila crezca, pero puede **truncar** un título. Eso no
se decide a ojo: se mide, y con la calibración que ya está **contrastada contra el teléfono** en
`_c9-medir-letra-rejilla.cjs` — **7,9 dp por carácter a tamaño 11**, sacada de que «Supermercados» (13,
letras anchas) **no** cabe en 80 dp y «Servicios del» (13, estrechas) **sí**.

El ancho disponible para el título es **280 dp** (360 − 16 de relleno por lado − 24 de cada extremo), y
a tamaño 17 eso son **22 caracteres**. Medidor nuevo: **`_b2-medir-titulos-cabecera.py`**.

```
Títulos que NO caben en una línea a 17: 2 de 21
   ✗ app/alquiler.tsx      «Alquileres en Guinea Ecuatorial»   31 car · 378 dp
   ✗ app/food-menu.tsx     el NOMBRE DEL COMERCIO              dinámico
```

Los dos pasan `lineasTitulo={2}`. **Los otros 19 caben en una, y el más justo es
«Mis compras y derechos» con 22 caracteres exactos de 22.** Esos son los que hay que recordar si algún
día se cambia el tamaño del título.

> **Y una trampa que costó un intento, aquí mismo:** la primera versión del medidor puso como título la
> expresión ternaria entera (`'Pedidos recibidos / Mis pedidos'`) y **dio por truncado un título que
> nunca se dibuja así**. Un medidor que inventa el dato que mide es peor que no medirlo: se corrige a
> la rama más larga que realmente se pinta.

---

## 5. Las tres puertas, con los números

```
npx tsc --noEmit     → 0 errores
npm run diseno       → OK: ninguna zona ha empeorado
npm run rutas        → mapa al día · 0 enlaces rotos · 0 pantallas huérfanas
```

**Y el trinquete ha bajado otra vez, por lo previsto y exactamente lo previsto:**

| Familia | Antes | Después |
|---|---:|---:|
| `espaciado` | 5.404 | **5.402** (−2) |
| `fontSize` | 664 | **663** (−1) |
| `fontWeight` | 1.971 | **1.970** (−1) |
| `borderWidth` | 549 | **548** (−1) |
| `borderRadius` · `hex` | 688 · 189 | sin cambio |

−5 literales en **un** fichero de los 21: `paddingHorizontal: 16` y `paddingVertical: 12`, más
`fontSize: 17`, `fontWeight: '800'` y `borderBottomWidth: 1`. **La aritmética se puede comprobar sola.**

### 5.1 La comprobación en el móvil — HECHA, y medida

El piloto no se ha aprobado mirando una previsualización: **se ha instalado y se ha mirado el píxel**.

```
compilación   gradlew assembleRelease        → BUILD SUCCESSFUL in 2m 46s
instalación   adb install -r                 → Success   (PKK110, el aparato de referencia)
captura       arranque en frío con el enlace egrouteplan://food-orders
```

| | |
|---|---|
| Antes | `_f2-piloto-cabecera-ANTES.png` — el build **de ayer 22:42** (`com.egrouteplan.app.prueba`), o sea el código **sin** el cambio |
| Después | `_f2-piloto-cabecera-food-orders.png` — el build de hoy (02:14) con `ScreenHeader` |

**Y no se comparan a ojo, porque a ojo son idénticos.** `_b4-mide-cabecera.py` descodifica los dos PNG
a mano —el entorno no tiene Pillow ni numpy— y mide la cabecera buscando sus propias referencias (el
borde inferior es la primera fila que destaca sobre el fondo en el lado derecho, que está vacío). Lo
que devuelve:

| | Antes | Después | Diferencia |
|---|---|---|---|
| Fila del borde inferior | 270 | **270** | **0 px → la fila mide lo mismo** |
| Trazo de la flecha | 44×44 px = **14,7 dp** | 48×48 px = **16,0 dp** | **+4 px = +1,3 dp** |
| Centro vertical de la flecha | 194,5 px | **194,5 px** | **0 px → crece alrededor de su centro** |
| **Tamaño de icono que implica** | **22,0 dp** | **24,0 dp** | el previsto |

**La comprobación tiene predicción exacta, y por eso vale:** el trazo de `ArrowLeft` de Lucide ocupa
16/24 de su caja, así que un icono de 22 dp dibuja un trazo de 14,67 dp y uno de 24 dibuja 16,00. Lo
medido es 14,7 y 16,0, y el medidor **deduce el tamaño del icono desde los píxeles sin saberlo**:
22,0 → 24,0. Es el cambio exacto, ni uno más.

**Dos intentos fallidos del medidor, que quedan escritos** porque son la misma trampa dos veces: acotar
la ventana de medida a ojo. La v1 (umbral bajo) metió el texto del chip «Todos» y midió una «flecha»
de 148 px; la v2 metió el reloj del sistema. La que funciona se apoya en el dibujo, no en coordenadas
elegidas a mano.

**Un aviso sobre la captura, para que no confunda a nadie:** el cuerpo de la pantalla enseña el estado
de error «No pudimos cargar los pedidos». Eso **no lo ha causado el cambio** —el piloto solo toca la
cabecera— y es el mismo error en las dos capturas: el arranque en frío deja la sesión caducada. Lo que
se verifica aquí es la cabecera, y la cabecera se ve igual en las dos.

---

## 6. Una trampa nueva, y es de sintaxis

**Un comentario JSX dentro del bloque de documentación cierra el bloque antes de tiempo.** En la
cabecera de `ScreenHeader.tsx` había escrito, como ejemplo de uso, un `<ScreenHeader/>` seguido de un
comentario de llave-barra-asterisco. Ese `*/` **termina el docblock**, y las 30 líneas siguientes de
prosa se leyeron como código: `tsc` escupió 44 errores de golpe.

La cazó **`tsc`, no la guardia** — la guardia cuenta valores, no sintaxis. Es la **misma familia** que
la trampa del literal dentro del comentario y que el prefijo duplicado: *el texto que documenta se
convierte en código si no se le quitan los símbolos que el lenguaje reconoce*. Queda escrita en el
propio fichero.

---

## 7. Lo que la unificación cambiará en las otras 20 (medido, antes de tocarlas)

Esto es lo que Bernardo tiene que aprobar, y está contado fichero a fichero para que no haya sorpresas:

| Cambio | Ficheros | Cuáles |
|---|---:|---|
| El título sube **16 → 17** | **6** | `alquiler`, `ecomerse-checkout`, `ecomerse-detail`, `ecomerse-favorites`, `ecomerse-orders`, `ecomerse-planes` |
| El título sube **700 → 800** | **7** | `alquiler-planes`, `alquiler-publicar`, `intercity-planes`, `landlord-profile`, `work-detail`, `work-planes`, `work-publish` |
| El icono sube **22 → 24** | **11** | los `ecomerse-*` y los `food-*`, más `alquiler` |
| El título queda **topado a 1 línea** | **13** | los que hoy no lo ponen. Es **protección**: evita que la fila crezca |
| El volver **responde al toque** | **19** | todos. Es el hallazgo que creó `Tactil`: de 1.163 `Pressable` del proyecto solo 156 hacían algo al pulsarse, y los 19 volveres estaban en el grupo que no |
| El volver **gana etiqueta y rol** | **1** | `landlord-profile.tsx`, que era el único sin ninguno de los dos |
| **Nada** cambia | 4 | los que ya estaban en 17 · 800 · 24 · con tope: `billing-checkout`, `billing-status`, `food-checkout`, `work-*` (según fila) |

**Ningún fichero cambia de alto de fila. Ninguno cambia de color. Ninguno cambia de estructura.**

---

## 8. Lo que queda abierto

1. **La aprobación del piloto por Bernardo.** La comprobación en el móvil está hecha y medida (§5.1);
   lo que falta es su visto bueno para tocar los otros 20.
2. **El reemplazo de los 20 restantes**, con `lineasTitulo={2}` en `alquiler.tsx` y `food-menu.tsx`.
3. **La `accion` de cada uno**, que se pasa tal cual: en `alquiler.tsx` un `Plus`, en `work-detail` el
   `Bookmark`, en `food.tsx` los dos iconos y en `food-owner` los dos textos. **El componente no
   interpreta la acción: la coloca.**
4. **Los dos `StepHeader` siguen siendo dos.** Si en algún momento se quiere un solo nombre, hay que
   renombrar el de KYC, y eso es una decisión de producto, no de diseño.
