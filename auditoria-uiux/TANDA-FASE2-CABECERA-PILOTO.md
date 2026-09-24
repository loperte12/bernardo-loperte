# Acta · FASE 2 — la cabecera de pantalla (`ScreenHeader`)

> **AVISO (24/09/2026, tarde): la §7 de este acta estaba mal y esta corregida.** El fichero integro
> con las tablas nuevas esta en `D:\\Routeplan` commit `211bd03`; esta copia se sube por el conector de
> GitHub, que tiene limite de tamano. Lo que hay que saber:
>
> · Decia «el titulo sube **16 -> 17** en 6 ficheros». **Ninguno de los 20 tiene 16**: son 14 a 17 y 6
>   con `tipografia.subtitle`.
> · Decia «**nada** cambia en 4». **Los cuatro cambian.**
> · El docblock del componente decia «19 volveres no hacen nada al pulsarse». **Los 20 navegan.** Lo
>   que faltaba era la senal visual: cero usan `Tactil`.
>
> Causa: medidores que resumian antes de leer. Ver el final de este documento.

**Fecha:** 24/09/2026 · **Rama:** `refactor/ui-ux-reconstruction` · **Fase:** 2 (Componentes Base), 6.ª pieza
**Estado:** componente escrito, piloto aplicado y **medido en el PKK110**. Aprobado por Bernardo; el
reemplazo de las otras 20 empieza ahora.

---

## 0. La decision, y el nombre que no se pudo usar

Bernardo decidio: **(1) el dueño vive en el kit**, como primitiva oficial, y **(2) se unifican las 21
variantes** en un solo componente con props.

**Pero el nombre `StepHeader` ya estaba tomado.** Existe
`packages/ui-kit/src/primitives/StepHeader.tsx` —la cabecera de un paso de flujo, con barra de progreso
segmentada y sin boton de volver— exportada en el barrel desde antes, y **5 pantallas de KYC la
importan**. Dos componentes con el mismo nombre en el mismo barrel es un error de importacion
esperando a ocurrir, asi que **el nuevo se llama `ScreenHeader`**.

---

## 1. Lo que se midio antes de escribir nada: no eran 21 copias iguales

El censo de formas (`_a5`) encontro la firma identica en **21 ficheros, todos en `app/`**. Pero la firma
es de las **declaraciones**: por dentro, el titulo se escribia de cuatro maneras.

| | Medicion |
|---|---|
| Origen del estilo del titulo | clave `headerTitle` **13** · en linea **8** |
| Tamaño del titulo | **17** en 15 · **16** en 6 |
| Peso del titulo | **800** en 14 · **700** en 7 |
| Icono de volver | **24** en 14 · **22** en 7 |
| `numberOfLines` | lo ponen **8** · no lo ponen **13** |
| Ancho del hueco derecho | **24** en 18 · **22** en 3 |

**Lo que se repetia no era un diseño, era la misma fila con cinco decisiones tomadas por separado.**

---

## 2. El componente

`packages/ui-kit/src/primitives/ScreenHeader.tsx`

| Prop | Que hace |
|---|---|
| `titulo` | El texto. Obligatorio |
| `subtitulo` | Segunda linea en `textSecondary` |
| `alVolver` | **Sin ella no hay boton** y queda un hueco del ancho del icono |
| `etiquetaVolver` | «Volver» por defecto |
| `pistaVolver` | `accessibilityHint` |
| `accion` | El contenido de la derecha |
| `lineasTitulo` | **1 por defecto** |
| `style` | Para el `paddingTop` de la zona segura |

**Las cinco decisiones, y con que se justifican:**

| Decision | Valor | Por que |
|---|---|---|
| Tamaño del titulo | `tipografia.subCabecera` (**17**) | El de **15 de los 21**; se eligio el mayoritario |
| Peso | `peso.maximo` (**800**) | El de **14 de los 21** |
| Icono | `icono.lg` (**24**) | El de **14 de los 21** |
| Alto de la fila | **48 dp** | Es el de hoy. No se toca |
| Toque del volver | `hitSlop` **12 por lado** -> 48 dp de dedo | Subir la caja a 44 la llevaria a **68** y cambiaria el alto de las 21 pantallas |

**Dos tokens que llevaban meses muertos entran en uso:** `tipografia.subCabecera` (17) lo escribian a
mano 15 ficheros sin que ninguno usara el token, y `icono.lg` no se usaba en ninguna cabecera.

---

## 3. El piloto: `app/food-orders.tsx`

```tsx
// antes — 12 lineas, 5 literales vigilados
<View style={s.header}>
  <Pressable onPress={() => router.back()} hitSlop={12} accessibilityRole="button" accessibilityLabel="Volver">
    <ArrowLeft size={22} color={colors.textPrimary} />
  </Pressable>
  <Text style={s.headerTitle}>{isOwner ? 'Pedidos recibidos' : 'Mis pedidos'}</Text>
  <View style={{ width: 24 }} />
</View>

// despues — 4 lineas, 0 literales
<ScreenHeader
  titulo={isOwner ? 'Pedidos recibidos' : 'Mis pedidos'}
  alVolver={() => router.back()}
/>
```

| | Antes | Despues |
|---|---|---|
| Titulo | 17 · peso 800 · sin tope | **identico**, con tope de 1 linea |
| Icono de volver | 22 | **24** (+2 px) |
| Al pulsar el volver | **nada** | el icono baja a 0,6 de opacidad |
| Alto de la fila | 48 | **48** |

---

## 4. Por que `lineasTitulo` existe

Poner `numberOfLines={1}` en todas arregla que la fila crezca, pero puede **truncar** un titulo. Se
mide con la calibracion ya contrastada contra el telefono en `_c9-medir-letra-rejilla.cjs` — **7,9 dp
por caracter a tamaño 11**.

El ancho disponible es **280 dp** (360 − 16 por lado − 24 por extremo), y a 17 eso son **22
caracteres**.

```
Titulos que NO caben en una linea a 17: 2 de 21
   x app/alquiler.tsx      «Alquileres en Guinea Ecuatorial»   31 car · 378 dp
   x app/food-menu.tsx     el NOMBRE DEL COMERCIO              dinamico
```

Los dos pasan `lineasTitulo={2}`.

> **Advertencia honesta, anadida el 24/09:** el limite es **22,9**, y los cuatro titulos de 21–22
> caracteres (`billing-status`, `intercity-planes`, `alquiler-planes`, `landlord-profile`) caben **por
> poco**. Y las 22,9 son el 7,9 dp/caracter del movil **escalado** de 11 a 17, no una medicion a 17.
> **Esos cuatro son el sitio donde mirar en el telefono.**

---

## 5. Las tres puertas

```
npx tsc --noEmit     -> 0 errores
npm run diseno       -> OK: ninguna zona ha empeorado
npm run rutas        -> 0 enlaces rotos · 0 pantallas huerfanas
```

| Familia | Antes | Despues |
|---|---:|---:|
| `espaciado` | 5.404 | **5.402** (−2) |
| `fontSize` | 664 | **663** (−1) |
| `fontWeight` | 1.971 | **1.970** (−1) |
| `borderWidth` | 549 | **548** (−1) |

−5 literales en **un** fichero de los 21. La aritmetica se puede comprobar sola.

### 5.1 La comprobacion en el movil — HECHA, y medida

```
compilacion   gradlew assembleRelease   -> BUILD SUCCESSFUL in 2m 46s
instalacion   adb install -r            -> Success   (PKK110)
captura       arranque en frio con egrouteplan://food-orders
```

| | Antes | Despues | Diferencia |
|---|---|---|---|
| Fila del borde inferior | 270 | **270** | **0 px -> la fila mide lo mismo** |
| Trazo de la flecha | 44x44 px = **14,7 dp** | 48x48 px = **16,0 dp** | **+4 px = +1,3 dp** |
| Centro vertical | 194,5 px | **194,5 px** | **0 px -> crece alrededor de su centro** |
| Icono que implica | **22,0 dp** | **24,0 dp** | el previsto |

**La comprobacion tiene prediccion exacta, y por eso vale:** el trazo de `ArrowLeft` ocupa 16/24 de su
caja, asi que 22 dp dibuja 14,67 y 24 dibuja 16,00. Lo medido es 14,7 y 16,0, y el medidor **deduce el
tamaño del icono desde los pixeles**: 22,0 -> 24,0. El cambio exacto, ni uno mas.

**Dos intentos fallidos del medidor, que quedan escritos:** acotar la ventana a ojo. La v1 metio el
texto del chip «Todos» (midio una «flecha» de 148 px); la v2 metio el reloj del sistema. La que
funciona se apoya en el dibujo, no en coordenadas elegidas a mano.

**Aviso:** el cuerpo de la captura enseña el error «No pudimos cargar los pedidos». **No lo ha causado
el cambio** —el piloto solo toca la cabecera— y sale igual en las dos: el arranque en frio deja la
sesion caducada.

---

## 6. Una trampa nueva, y es de sintaxis

**Un comentario JSX dentro del bloque de documentacion cierra el bloque antes de tiempo.** En la
cabecera de `ScreenHeader.tsx` habia un `<ScreenHeader/>` seguido de un comentario de llave-barra-
asterisco. Ese `*/` **termina el docblock**, y las 30 lineas siguientes de prosa se leyeron como
codigo: `tsc` escupio 44 errores de golpe.

La cazo **`tsc`, no la guardia** — la guardia cuenta valores, no sintaxis.

---

## 7. La tabla corregida de las otras 20

Antes de tocar nada se midio con `_c0-lista-cabeceras.py` (reproduce la clave del censo y devuelve
**exactamente 20**) y `_c1-mide-20-cabeceras.py` (resuelve el estilo del titulo siguiendo el JSX).

| Cambio | Ficheros | Cuales |
|---|---:|---|
| El titulo se queda en **17** | 14 | no hay subida: el 17 es el valor de 14 de 20 |
| El titulo usa `tipografia.subtitle` | 6 | `alquiler`, los cinco `ecomerse-*` |
| El titulo sube **700 -> 800** | **7** | `alquiler-planes`, `alquiler-publicar`, `intercity-planes`, `landlord-profile`, `work-detail`, `work-planes`, `work-publish` |
| El icono sube **22 -> 24** | **11** | `alquiler`, los cinco `ecomerse-*`, los cinco `food-*` |
| El icono ya estaba en **24** | 9 | los 3 `work-*`, los 2 `billing-*`, `alquiler-planes`, `alquiler-publicar`, `intercity-planes`, `landlord-profile` |
| El titulo **gana tope de linea** | 11 | los que hoy no lo ponen |
| El volver **gana respuesta visual** | **20** | todos |
| El volver **gana etiqueta y rol** | **1** | `landlord-profile.tsx`, el unico sin ninguno de los dos |

**`_c0` avisa ademas de 56 ficheros mas con un `header:` que NO es la misma forma** (los `lifebook-*`,
los `monedero-*`, las laminas). Unos usan `hairlineWidth` en vez de `1`, otros llevan `gap`. **No
entran**: meterlos en la misma tanda seria un rediseño disfrazado de unificacion.

---

## 8. Lo que queda abierto

1. **El reemplazo de los 20 restantes**, en 5 tandas, con `lineasTitulo={2}` en `alquiler.tsx` y
   `food-menu.tsx`.
2. **La `accion` de cada uno**, que se pasa tal cual: en `alquiler.tsx` un `Plus`, en `work-detail` el
   `Bookmark`, en `food.tsx` los dos iconos, en `food-owner` los dos textos. **El componente no
   interpreta la accion: la coloca.**
3. **Los cuatro titulos al borde**, que comprueba Bernardo en el movil.
4. **Los dos `StepHeader` siguen siendo dos.** Unificarlos es una decision de producto.

---

## 9. La leccion de este acta: un medidor que resume antes de leer inventa el dato

Las tres correcciones de esta tarde son el mismo error tres veces:

1. **`fontSize` del fichero en vez del titulo.** `billing-status` titula a 17; el primer `fontSize` del
   fichero es un **38** de un emoji de estado.
2. **`[^>]*` para leer atributos.** Se corta en el `>` de `() =>`, asi que devolvia `onPress={() =` y
   **perdia `accessibilityRole`**. De ahi el «19 sin rol» que era falso: 19 **si** lo tienen.
3. **El titulo mas largo en vez del titulo.** En `ecomerse-orders` el titulo es un ternario
   (`{enZona ? 'Pedidos' : 'Mis pedidos'}`), y contarlo entero daba por truncado un titulo que nunca
   se dibuja asi.

Los tres se arreglaron midiendo contra el **fichero real**, no contra una idea del fichero. El medidor
final no da ni un aviso en los 20, y su salida se puede reproducir.

*Acta de trabajo — 24/09/2026. Kai*
