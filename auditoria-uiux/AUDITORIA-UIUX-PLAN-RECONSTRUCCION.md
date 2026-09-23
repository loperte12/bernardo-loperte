# Auditoría UI/UX — EG Route Plan · FASE 3b: Plan de Reconstrucción

**Regla que gobierna todo el plan, y sale de la medición:** esto **no es sustituir el sistema de diseño**.
El sistema existe, está razonado y es bueno (color 8/10, contraste de claro 16/16 en AA). Lo que hay que
hacer es **terminarlo: reconciliar las escalas declaradas con las que el producto usa, y después migrar.**
El plan es por eso más barato, más seguro y no toca la identidad visual que ya tienes.

**Anti-objetivos (lo que este plan NO hace):**

1. **No adoptar la paleta generada.** Verificado: es menos accesible que la actual (§1.4 del informe de
   tokens). Reintroduce cuatro fallos ya resueltos.
2. **No cambiar los valores de los tokens que ya funcionan.** Ningún token con adopción alta cambia de
   valor en las fases 1–2.
3. **No crear «un componente universal de tarjeta».** Las tres tarjetas tienen datos distintos.
4. **No hacer un big-bang.** Cada fase cierra con las tres puertas en verde y con la base del trinquete
   bajada. Si una fase no puede cerrarse verde, no se abre la siguiente.

**Las tres puertas de cada fase (hoy, todas en verde):**

```bash
npx tsc --noEmit                 # hoy: 0 errores
npm run diseno                   # hoy: OK, ninguna zona ha empeorado (peso 1971 · trazo 549 · espaciado 5423)
npm run rutas                    # hoy: mapa al día · 0 enlaces rotos · 0 pantallas huérfanas
```

---

## ESTADO DEL PLAN · traspaso del 24/09/2026

| Fase | Estado | Qué queda exactamente |
|---|---|---|
| **1 · Design Tokens** | **CERRADA** | Nada. Las tres puertas pasadas después de aplicar todo: `tsc` 0 · `diseno` OK · `rutas` al día |
| **2 · Componentes Base** | **EN CURSO — 4 de 5 piezas hechas (24/09/2026)** | Hecho: **`altura.*` adoptada** (12 usos → **37 en 13 ficheros**; `campo` y `boton` ya no están muertos) · **el corazón unificado** (3 versiones → 1) · **`app/ecomerse.tsx` con −28 literales** · la casilla del carrito a 44 dp. **Cerrado con decisión:** el stepper de `food-checkout` se queda como está. Queda: `lifebook`/`food` donde la forma se repita 3 veces (el censo táctil queda **cerrado con regla y decisión**, no con 38 parches — ver §7.3 del doc de tokens) |
| **3 · Pantallas** | **NO EMPEZADA** | — |
| **4 · Auditoría final** | **NO EMPEZADA** | — |

**Decisiones cerradas el 24/09/2026** (reemplazan a §1.1, que las pedía): **propuesta A** de tipografía ·
los 214 de `600` van a **`medio` (500)** · nombres de radio **`marca` · `chip` · `campo` · `tarjeta`**.
El detalle y el coste medido de cada una está en el bloque §0 del documento de tokens.

**Lo que la Fase 1 dejó fuera a propósito, y no se debe olvidar:** la migración de los literales a token
(1.975 de `fontWeight`, 664 de `fontSize`, 724 de `borderRadius`, 550 de `borderWidth`, 5.446 de
espaciado) **no es Fase 1 — es Fase 3.** La Fase 1 solo declaró las escalas y las puso al día; el código
sigue escribiendo los mismos números que antes. **Hoy la proporción token/literal es 1,02 y no se ha
movido ni un punto.** Quien lea «Fase 1 cerrada» y crea que la deuda bajó se equivoca: lo que bajó es la
*distancia entre lo declarado y lo usado*, que era el problema de verdad.

---

## FASE 1 · Design Tokens

**Objetivo:** que las escalas declaradas **cubran lo que el código ya escribe**, y que la guardia vea
todas las familias. **Cero cambios de aspecto en la app.**

### 1.1 Qué se toca

| Fichero | Cambio | Sitios |
|---|---|---|
| `packages/ui-kit/src/theme/colors.ts` | +6 tokens `*Dark` de texto · +rampa `neutro` de 11 · +`darkColors.sheet` | 18 líneas |
| `packages/ui-kit/src/theme/escalas.ts` | `peso.maximo: '800'` · `tipografia`: propuesta A o B · `espaciado` a base 2 · `radios` +4 · `trazoIcono` nuevo | 5 bloques |
| `pruebas/verifica-diseno.cjs` | **Añadir la regla de espaciado** (`(padding\|margin\|gap)[A-Za-z]*:\s*\d+`) y su base | ~10 líneas |
| `.diseno-baseline.json` | Regenerar con `npm run diseno -- --base` **después** de añadir la regla de espaciado | — |
| `packages/ui-kit/src/index.ts` | Exportar lo nuevo | 5 líneas |

**Decisiones que Bernardo tomó en esta fase — CERRADAS el 24/09/2026:**

- Tipografía: **propuesta A** — 9 peldaños, mueve `micro` −0,5 px y `display` −2 px, y **234 literales
  dejan de ser excepción**. (B también tenía 9 peldaños: el borrador decía 8 y 7, y era un error.)
- Peso: los **214 literales de `600`** van a **`medio` (500)**. Al aplicarlo apareció la razón técnica
  que lo refuerza: la app no declara ninguna `fontFamily` y nunca carga fuentes, así que **`600` es el
  único peso cuyo dibujo decide el aparato**. Y están **todos en Transporte y perfil — 0 en el Mercado**:
  la verificación se hace en un módulo, no en media app.
- Radios: nombres confirmados **`marca / chip / campo / tarjeta`**.

Las tres están aplicadas en el código y las tres puertas están pasadas. El bloque §0 del documento de
tokens tiene el detalle y las mediciones.

### 1.2 Esfuerzo

**Talla `M`.** Es trabajo de precisión, no de volumen: **~40 líneas de tokens** + la regla de la guardia.
Lo lento no es escribir los tokens, es **cerrar cada decisión mirando dos pantallas**.
Dependencia: ninguna. **Es la fase que desbloquea las otras tres.**

### 1.3 Riesgos

| Riesgo | Por qué | Mitigación |
|---|---|---|
| **Cambiar la base del espaciado rompe 432 usos** | `espaciado.xs` → `espaciado.e4` es un renombrado | Codemod mecánico + `tsc` lo caza todo: son nombres, no valores. **Cero cambio visual** |
| **La regla nueva de la guardia marca 5.446 literales de golpe** | El trinquete compara contra la base | Se añade la regla **y se regenera la base en el mismo commit**: la deuda de espaciado nace *registrada*, no como fallo |
| **Los `*Dark` se usan como relleno por error** | Dos tokens con el mismo papel aparente | Nombre explícito + comentario con la regla («solo texto/icono sobre oscuro»); la guardia no lo puede vigilar, lo vigila la revisión |
| Adoptar algo de la paleta generada por inercia | Es cómodo | La tabla de decisión §1.4 está escrita **con las mediciones al lado** |

---

## FASE 2 · Componentes Base

**Objetivo:** que lo que ya existe se use, y que lo que está copiado deje de estarlo. **Sin rediseñar
ninguna pantalla.**

### 2.1 Qué se toca

**Hecho el 24/09/2026 — la casilla del carrito (`app/lifebook-carrito.tsx`):** el `Pressable` pasa a
`altura.punto` (44) y el círculo de 21 va centrado dentro, así que el diseño no cambia y el `hitSlop={8}`
se retira. Es el caso **suelto**, donde el patrón funciona sin efectos laterales.

**Y el censo que faltaba, porque «los dos sitios conocidos» era una nota sin medir:** hay **152** objetos
de estilo con `width` **y** `height` explícitos por debajo de 44 dp, de los cuales **39 llevan nombre de
control** y no de decoración. El «micrófono del buscador» que citaba el borrador **no existe** —era un
dato inventado—; los casos reales están en la tabla §7.1 del documento de tokens y los concentra
**`lifebook` (15 de 39)**.

| Fichero | Cambio | Por qué |
|---|---|---|
| `packages/ui-kit/src/primitives/*` | Aplicar `altura.*` a los controles (campo 50, botón 52) | Los dos peldaños existen y **no se usan ni una vez** |
| Los **39 controles** del censo §7.1 por debajo de 44 dp | **Subir el elemento**, no el hitSlop. Y **uno a uno**: no distingue el Pressable del icono de dentro, así que hay que abrir cada caso | §7.1 del documento de tokens. Ya hecho: la `casilla` del carrito (21 dp) |
| El stepper de `food-checkout` (`stepBtn`/`iconBtn` 28 dp) — **pendiente de decisión** | Envolver en 44 dp **cambia la separación visible de 8 a 24** porque el `gap` se mide entre cajas de 44. Subir la caja visible a 44 sí altera el diseño | Es el caso que **no** se resuelve con el patrón: fila con `gap`. Se mira en pantalla antes de tocar |
| `components/ecomerse/useAccionesProducto.ts` | Extenderlo a `ecomerse-detail.tsx` y `ecomerse-tienda.tsx` | Hoy el favorito optimista está en **3 versiones** |
| `components/lifebook/*` y el bloque `food-*` | Aplicar la misma regla donde la forma se repita 3 veces | La deuda de estos módulos es la más alta del proyecto, y el censo táctil lo confirma: 15 de 39 en `lifebook` |
| `app/ecomerse.tsx` | `DIAM_FOTO_FAMILIA`, `ALTO_CELDA_FAMILIA`… pasar los números que ya son tokens | Es el módulo mejor migrado; se usa como patrón de referencia |

**Estado de cada fila, medido el 24/09/2026 — y son seis, no cinco:**

| Fichero | Estado |
|---|---|
| `packages/ui-kit/src/primitives/*` | **HECHO.** `PrimaryButton`: `height 52` → `altura.boton`, `16` → `radios.lg`, `'800'` → `peso.maximo`, los dos `#FFFFFF` → `brand.white`. `FormField`: `height 50` → `altura.campo`, `14` → `radios.campo`, `1.5` → `trazo.base`, los cinco espaciados y `12`/`15` → `tipografia.caption`/`cuerpo`. **Los dos peldaños que no se usaban ni una vez, usados.** |
| Los 39 controles del censo §7.1 | **CERRADO CON REGLA Y DECISIÓN, no con parches.** §7.3 del documento de tokens: en una fila con `gap` el alcance máximo de `hitSlop` es `tamaño + gap`, y solo la clase de 38 dp con `gap` 8 llega a 44. El resto exige cambiar el diseño y Bernardo decidió no cambiarlo. |
| El stepper de `food-checkout` | **DECIDIDO: se queda como está** (24/09/2026). |
| `components/ecomerse/useAccionesProducto.ts` | **HECHO.** `ecomerse-tienda` usa el módulo entero y `ecomerse-detail` el corazón. De la tienda desaparecen tres cosas: su `useEffect` de favoritos, su `alternarFavorito` **y su copia de la guarda de combinaciones**. El detalle gana lo que no tenía: la sincronización al montar, así que abrir una ficha por enlace directo ya no enseña el corazón vacío. |
| `app/ecomerse.tsx` | **HECHO (−28 literales).** 4 `fontWeight`, 1 `borderWidth` y 23 espaciados que ya tenían peldaño. Se quedan `40` y `48` (son anchos de caja, no huecos) y el `0` (un reinicio). |
| `components/lifebook/*` y `food-*` | **PENDIENTE.** Es lo que queda de la fase, y va por módulo. |

**La guardia ya lo cobra:** la base pasó de `fontWeight 1975 · borderWidth 550 · espaciado 5446` a
**`1971 · 549 · 5423`**, con las tres puertas en verde. **Es la primera vez que el trinquete baja por
trabajo planificado y no por arreglar un defecto concreto.**

### 2.2 Esfuerzo

**Talla `L`.** Es el trabajo más ingrato y el de mayor relación calidad/esfuerzo: **no se ve, y se nota
en todo.** Estimación: **una tanda por módulo** (`ecomerse` → `lifebook` → `food` → `conductor/taxi`),
**4–5 tandas**, cada una cerrando con las tres puertas y la base bajada.

### 2.3 Riesgos

| Riesgo | Por qué | Mitigación |
|---|---|---|
| **Subir un elemento de 21 a 44 dp descoloca la fila** | El layout se hizo contando con 21 | Se auditan primero **los dos casos conocidos**, se mira en el móvil, y después se busca el resto |
| **Migrar `fontWeight` cambia el peso de media app** | 1.975 literales; los 214 de `600` sí cambian | Fase por módulo, **empezando por el Mercado** (que ya está migrado) para validar el criterio antes de tocar `conductor.tsx` |
| **Extraer el hook del corazón a `ecomerse-detail` rompe su estado propio** | Esa pantalla guarda estado del producto | Se extrae **solo la acción compartida**, no el estado: mismo patrón que ya funcionó en `ecomerse-subcategoria` |
| Las tres puertas se quedan rojas a mitad de tanda | El trinquete es por fichero | **Nunca se deja una tanda a medias**: o el fichero queda entero, o no se empieza |

**Riesgos que NO se materializaron, y por qué (24/09/2026):** el hook del corazón se extrajo **sin tocar
el estado propio** de la ficha —el módulo presta `favIds` y `alternarFavorito`, y la ficha conserva
combinaciones, cantidad y «comprar ahora»—, así que la ficha no perdió nada. Y la tanda **no se dejó a
medias**: cerró con `tsc` 0 y la guardia verde.

**Y uno que sí apareció, y no estaba en la tabla: el codemod escribió `trazo.trazo.fino`.** El valor del
mapa ya llevaba el prefijo y la sustitución lo volvió a poner. **Lo cazó `tsc`; la guardia no podía**
—cuenta literales, y eso no es un literal—. De aquí sale una regla: **en un codemod el prefijo se pone en
un solo sitio**, y la verificación de un codemod no es que cambie lo que quieres, sino que `tsc` siga en 0.

---

## FASE 3 · Pantallas

**Objetivo:** que las pantallas **usen** lo anterior. Es donde el trabajo se ve.

### 3.1 Orden, y por qué ese orden

1. **`conductor.tsx` y `taxi.tsx`** — la deuda más alta del proyecto (74p/37l/34r y 71p/30l/20r). Son dos
   ficheros, muy grandes, y son el **peor caso**: arreglarlos primero fija el criterio.
2. **`profile.tsx` y `edit-profile.tsx`** — deuda alta en pantallas de uso diario.
3. **`lifebook-*`** (chat, post, carrito, user, hotel) — 6 ficheros, todos con la misma forma.
4. **`food-*`** (rider, orders, owner) — 3 ficheros.
5. **`work-detail.tsx`** y `components/DriverHomeSheet.tsx`.

### 3.2 Lo que hay que verificar en pantalla, no en el código

- **Modo oscuro con los 6 tokens nuevos**, en las tres pantallas donde el color de marca es texto:
  estados de pedido, precios y avisos. **Es el hallazgo crítico: hay que verlo, no razonarlo.**
- **`font_scale` distinto de 1,15.** Toda la geometría está calibrada con el 1,15 del móvil de
  referencia. A 1,3 la rejilla pide 102 dp de rótulo y crece. **No está mirado.**
- El **control subido a 44 dp** (la casilla del carrito) en el dedo, no en el código. Es el único que se
  subió: los demás del censo **se quedan como están** por decisión del 24/09/2026 (§7.3 del doc de tokens),
  porque en una fila con `gap` el alcance máximo es `tamaño + gap` y cambiarlo altera el diseño.

### 3.3 Esfuerzo

**Talla `XL`.** Es el 80 % del volumen: **192 ficheros con deuda**. Estimación: **una tanda por
pantalla o por par de pantallas**, con las tres puertas entre tandas. **No hay atajo**: el trinquete solo
permite bajar, así que el orden está garantizado, pero el tiempo lo pone el número de ficheros.

### 3.4 Riesgos

| Riesgo | Por qué | Mitigación |
|---|---|---|
| **El JS va dentro del APK: todo cambio exige recompilar** | Es el hecho central del proyecto | Agrupar cambios y compilar **por tanda**, no por fichero. `compilar-apk.ps1 -SoloCompilar` (≈5 min) |
| Pantallas sin captura posible (estados de error, flujos raros) | No se puede ver todo | **Lo que no se ha visto, se declara no verificado** — como se hizo en la pantalla de subcategoría |
| La tentación de «ya que estoy, lo rediseño» | Es donde se pierde el control | **Regla: en la fase 3 no se cambia ningún diseño**, solo se sustituyen valores por tokens. Un rediseño abre su propia fase |
| El trinquete se queda «verde» con la base alta | Un trinquete solo impide empeorar | Al cerrar cada tanda, `npm run diseno -- --base` **solo si hay mejora real**. La base es el marcador, no el objetivo |

---

## FASE 4 · Auditoría final

**Objetivo:** demostrar que el trabajo se hizo, con números, y dejarlo **protegido** para que no se
deshaga.

### 4.1 Qué se hace

| Paso | Cómo |
|---|---|
| Repetir el censo de adopción | `_a1-censo-diseno.py` → la proporción token/literal debe pasar de **1,02** a **> 4** |
| Repetir el contraste | `uiux_check_contrast` con las **44 parejas** (claro + oscuro + los 6 nuevos) → **0 fallos** |
| Repetir el censo de espaciado | el `grep` de `padding\|margin\|gap` → **% fuera de la escala** debe caer del 54 % a **< 10 %** |
| Recompilar y verificar en el móvil | `compilar-apk.ps1`, instalar con `push` + `pm install -r -d`, capturas por enlace `egrouteplan://` |
| Cerrar la guardia | `npm run diseno` verde, con la base **muy por debajo** de la de hoy |
| Acta de cierre | `RECONSTRUCCION-UIUX-CIERRE.md` con la tabla antes/después, y lo que **no** se verificó |

### 4.2 Esfuerzo

**Talla `M`.** Media jornada de medición + una de verificación en el móvil. **No se puede abreviar**: es
la fase que convierte «creo que ha mejorado» en «ha mejorado esto, medido así».

### 4.3 Riesgos

| Riesgo | Por qué | Mitigación |
|---|---|---|
| **Declarar la victoria con la guardia, no con el píxel** | La guardia mide literales, no diseño | La fase 4 exige **capturas del móvil** de las pantallas clave, no solo el verde de la guardia |
| La deuda vuelve a subir tras la reconstrucción | Sin presión, el trinquete se relaja | La guardia con la base nueva **es** la presión; y añadir la regla de espaciado cierra el último hueco |
| Sesiones de prueba de 1 h y OTP | Ya documentado en las trampas | Regenerar tokens justo antes de medir |

---

## Resumen de dependencias y esfuerzo

```
FASE 1 (M)  Design Tokens ──────────► desbloquea TODO
                │
                ├──► FASE 2 (L)  Componentes Base ──┐
                │                                    ├──► FASE 4 (M)  Auditoría final
                └──► FASE 3 (XL) Pantallas ─────────┘
```

| Fase | Talla | Volumen | Se puede paralelizar con |
|---|---|---|---|
| 1 · Design Tokens | **M** | ~40 líneas + 1 decisión grande | — (bloquea a las demás) |
| 2 · Componentes Base | **L** | 4–5 tandas por módulo | Fase 3, **módulo a módulo** |
| 3 · Pantallas | **XL** | 192 ficheros / 3.913 literales | Fase 2, **módulo a módulo** |
| 4 · Auditoría final | **M** | 1 medición + 1 verificación en móvil | — (es el cierre) |

**Las fases 2 y 3 se solapan a propósito por módulos** —se cierra un módulo entero (componente +
pantallas) antes de pasar al siguiente—, porque así el criterio se valida en el módulo más fácil
(`ecomerse`) antes de tocar `conductor.tsx`, que es el peor.

**Lo que hace esto sostenible no es la disciplina: es el trinquete.** `verifica-diseno.cjs` impide
empeorar aunque nadie mire, y con la regla de espaciado añadida en la fase 1 **ya no le queda ninguna
familia ciega**.
