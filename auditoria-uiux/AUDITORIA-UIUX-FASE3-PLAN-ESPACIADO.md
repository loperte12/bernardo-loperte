# FASE 3 · El espaciado, por tandas — plan de ejecución

**Fecha:** 24/09/2026 · **Rama:** `refactor/ui-ux-reconstruction` · **Estado:** plan aprobado, **decisión de §2.1 tomada y aplicada**; falta el codemod
**Herramienta de medida:** `_b1-censo-espaciado.py` · `_b3-desglose-lifebook.py` · `_b5-hueco-24-32.py`

---

## 0. Lo que hay que desaprender antes de empezar

**La auditoría arrastraba una cifra que ya es falsa: «el 54 % del espaciado está fuera de la escala».**
Esa medida se hizo contra la escala **vieja** —base 4, once peldaños—, cuando la cobertura era del
43,9 %. La Fase 1 la cambió por 20 peldaños y la cobertura pasó al 82,3 % y luego al 95,7 %. **Medido
hoy, con el mismo patrón que usa la guardia:**

```
5.459 literales de espaciado     ->     245 fuera de la escala = 4,5 %
```

El objetivo que la Fase 4 se había puesto —«bajar del 54 % a menos del 10 %»— **ya está cumplido, y no
lo cumplió la Fase 3: lo cumplió la reconciliación de la Fase 1.**

**Y de ahí sale el cambio de naturaleza de esta fase, que es lo importante:**

> **Migrar el espaciado no es arreglar 5.459 valores mal puestos. Es ponerle nombre a 5.214 valores que
ya son legales.** `espaciado.e8` **es** 8. El píxel no se mueve: cambia el nombre que lo escribe.

Tres consecuencias: (1) **cero píxeles por construcción**; (2) **el riesgo no está en el diseño, está en
el codemod** —las tres trampas ya pagadas son trampas de herramienta—; y (3) **el criterio no lo fija
el gusto de nadie, lo fija la tabla de la escala**, así que el orden de las tandas **no** es «primero lo
peor»: para un renombrado el criterio ya está fijado y hay que **probar la herramienta en el sitio más
pequeño**.

---

## 1. El punto de partida, medido

### 1.1 Dos cifras que no coinciden, y por qué

| Herramienta | Cuenta | Zonas que mira |
|---|---:|---|
| Censo `_b1` | **5.459** | 9: `app` · `components` · `packages/ui-kit/src` · `core` · `state` · `api` · `utils` · `constants` · `pruebas` |
| Trinquete `npm run diseno` | **5.402** | 7: las mismas menos `packages/ui-kit` (excluido a propósito) y `pruebas` |

La diferencia —**57**— es de **alcance, no de patrón**: los dos usan exactamente el mismo
`(padding|margin|gap)[A-Za-z]*: <número>`. El `packages/ui-kit` aporta 55 y `pruebas` el resto.

### 1.2 Reparto por módulo

| Módulo | Literales | Fuera | % fuera | Ficheros |
|---|---:|---:|---:|---:|
| **lifebook** | **2.481** | 101 | 4,1 % | **93** |
| app (raíz, sin módulo) | 437 | 20 | 4,6 % | 21 |
| food | 393 | 17 | 4,3 % | 6 |
| components (sin lifebook) | 319 | 21 | 6,6 % | 22 |
| ecomerse | 300 | 15 | 5,0 % | 14 |
| work | 281 | 8 | 2,8 % | 6 |
| profile | 254 | 15 | 5,9 % | 4 |
| alquiler | 212 | 11 | 5,2 % | 4 |
| conductor | 182 | 11 | 6,0 % | 2 |
| taxi | 138 | 5 | 3,6 % | 1 |
| monedero | 112 | 6 | 5,4 % | 5 |
| intercity | 96 | 3 | 3,1 % | 3 |
| billing | 76 | 2 | 2,6 % | 2 |
| rental | 69 | 0 | 0,0 % | 5 |
| packages/ui-kit | 55 | 5 | 9,1 % | 15 |
| kyc | 29 | 1 | 3,4 % | 4 |
| core | 16 | 3 | 18,8 % | 2 |
| auth | 9 | 1 | 11,1 % | 1 |
| **TOTAL** | **5.459** | **245** | **4,5 %** | **~309** |

**La lectura que manda: `lifebook` es el 45 % del espaciado del proyecto, en 93 ficheros.**

### 1.3 Los cinco ficheros peores

| Literales | Fichero |
|---:|---|
| 155 | `app/conductor.tsx` |
| 138 | `app/taxi.tsx` |
| 121 | `app/edit-profile.tsx` |
| 117 | `app/tienda/publicar.tsx` |
| 108 | `app/lifebook-chat/[id].tsx` |

### 1.4 Los valores más escritos (no son excepciones: son el idioma)

`8` -> 836 usos · `10` -> 795 · `12` -> 761 · `6` -> 569 · `14` -> 400 · `16` -> 348 · `4` -> 338 · `2` -> 275

Los ocho primeros suman **4.322 usos, el 79 %**, y los ocho están en la escala.

---

## 2. Las 245 que NO son un renombrado

| Clase | Usos | Qué es de verdad | Decisión |
|---|---:|---|---|
| **`1`** | 69 | Un **nudo de alineación**: `gap: 1`, `marginTop: 1`. No separa nada, ajusta óptica. | **NO se toca.** No es espaciado |
| **`0`** | 19 | Un **reinicio**: `padding: 0`, `margin: 0`. | **NO se toca** |
| **Anchos de caja >= 34** | 91 | **No son huecos, son tamaños**: el ancho de una tarjeta, de una miniatura. | **NO van a esta familia.** Esperan a una escala de **tamaños**, que hoy no existe |
| **Huecos de verdad sin peldaño** (30·28·26·15·17·3,5·2,5) | **66** | Esto sí es un hueco, y la escala no lo tiene. | **Decidido: opción A** (§2.1) |

### 2.1 El hueco entre 24 y 32 — **RESUELTA el 24/09/2026**

La escala saltaba de **24 a 32**, y en ese hueco viven **26 (12 usos), 28 (19) y 30 (27) = 58 usos**.

| Opción | Qué implica |
|---|---|
| **A · Añadir `e26` · `e28` · `e30`** — **ELEGIDA** | La escala pasa a **22 peldaños** y cubre el **98,5 %** de los huecos enteros. Coherente con lo que la escala ya declara de sí misma: *«esto ya no es una progresión, es el conjunto de los valores en uso, con nombre y contados»*. **Cero píxeles** |
| **B · Unificar a 28** | Tres valores se convierten en uno: **mueve 39 sitios** y hay que mirarlos uno a uno. Es un rediseño, no un renombrado |

**DECIDIDA: A** (24/09/2026). Aplicada a `packages/ui-kit/src/theme/escalas.ts`. Las tres puertas
pasadas, y el delta es **exactamente 0** porque el cambio es de la escala y ningún literal la usa
todavía: `hex 189 · fontSize 663 · borderRadius 688 · fontWeight 1970 · borderWidth 548 ·
espaciado 5402 · precioFigura 12`, idéntico al de antes de tocarla. **Eso prueba que es una decisión
de nombres y no de diseño.**

#### 2.1.1 La segunda auditoría: por qué el plan decía 58 y el conteo dijo 57

Al medir los literales que la decisión A nombra salieron **57**, y el plan (medido con el censo `_b1`)
decía **58**. Un medidor que no cuadra con su propia frase es lo que este proyecto decidió no tolerar,
así que se abrió `_b5-hueco-24-32.py`. **No la tenía nadie:**

| Conteo | Zonas | 26 | 28 | 30 | Total |
|---|---|---:|---:|---:|---:|
| Censo `_b1` · el del plan | **9** (incluye `packages/ui-kit` y `pruebas`) | 12 | 19 | 27 | **58** |
| Guardia `npm run diseno` | **7** | 12 | **18** | 27 | **57** |

**El `28` que falta vive en `packages/ui-kit`**, que la guardia no vigila a propósito. Es **el mismo
alcance, otra vez** — el mismo motivo por el que 5.459 != 5.402 (§1.1). Y el desglose del plan
(12/19/27) era exacto: el error era mío, por comparar dos cifras de alcances distintos sin decir cuál
era cuál.

#### 2.1.2 Y un dato que cambia la forma de la tanda

Los 58 **no están concentrados**: están **en 37 ficheros distintos**, 8 de ellos en `components` y el
resto de uno en uno (`lifebook-player` 3, `lifebook-user` 3, y así hasta 35 ficheros con un solo uso).

Consecuencia práctica: **no es una tanda de renombrado, es una tanda de excursión.** El valor está una
vez por fichero, así que el trabajo no es "sustituir 58 sitios" sino **abrir 37 ficheros y decidir en
cada uno si ese `28` es de verdad un hueco de espaciado** —o un ancho de caja, que es el error de
clase que §2 ya identificó—. Va donde va la migración de su módulo, no como tanda propia.

Y quedan **15 (4), 17 (1), 3,5 (2), 2,5 (1) = 8 usos** sueltos. Con `e16`/`e18` y `e3`/`e4` al lado,
estos ocho **se revisan a mano uno por uno**.

**Los 245 en total: 85 se aceptan como excepción (los 69 `1`, los 19 `0` y los 8 sin vecino), 58 ya
tienen peldaño por la decisión A, y 91 esperan a una escala de tamaños que hoy no existe.**

**F3.0 queda cerrada con esto.** De las dos cosas que la bloqueaban —la herramienta y la decisión—, la
decisión está tomada y aplicada (§2.1). Queda el codemod.

---

## 3. Las tandas

**Regla de oro: una tanda se cierra entera o no se empieza.** El trinquete es por fichero: dejar un
fichero a medias lo deja peor que como estaba.

| Tanda | Alcance | Literales | Ficheros | Talla | Por qué en este orden |
|---|---|---:|---:|---|---|
| **F3.0** | **La herramienta** — la decisión de §2.1 ya está tomada y aplicada | — | — | **M** | Sin codemod probado ninguna tanda es segura. La decisión ya no bloquea |
| **F3.1** | **Ensayo:** `billing` + `kyc` + `auth` + `core` | 130 | 9 | **S** | Es la tanda más pequeña que **ejercita todas las formas**: un checkout, un flujo de pasos, un fichero CRLF si cae. Prueba el codemod donde un fallo cuesta 9 ficheros, no 93 |
| **F3.2** | `ecomerse` | 300 | 14 | **M** | Es el módulo **mejor migrado** del proyecto |
| **F3.3** | `food` + `rental` + `intercity` | 558 | 14 | **M** | Módulos completos y medianos; ya llevan la cabecera unificada |
| **F3.4** | `work` + `alquiler` + `profile` | 747 | 14 | **L** | Aquí viven `edit-profile` (121) y `profile` (98). Tras 3 tandas el codemod ya es aburrido |
| **F3.5** | `conductor` + `taxi` + `monedero` | 432 | 8 | **L** | **Los dos peores ficheros del proyecto** (155 y 138) |
| **F3.6** | `app` (raíz, los 21 sin módulo) | 437 | 21 | **L** | Mezcla de pantallas sueltas |
| **F3.7** | `components` (sin lifebook) | 319 | 22 | **L** | Lo que se rompa aquí se nota en varias pantallas: va **después** |
| **F3.8** | **lifebook** · mensajería | 241 | 7 | **S** | Primer trozo de la mitad del proyecto |
| **F3.9** | **lifebook** · hotel | 419 | 13 | **M** | |
| **F3.10** | **lifebook** · comercio y carrito | 449 | 12 | **M** | |
| **F3.11** | **lifebook** · hojas y editores | 673 | 42 | **L** | 42 ficheros con **media de 16 literales**: muchos ficheros, poco dentro |
| **F3.12** | **lifebook** · feed, perfil y resto | 699 | 19 | **L** | El mayor. Contiene `lifebook-chat/[id]` (108), `lifebook-post/[id]` (89), `lifebook-user` (86) |
| — | `packages/ui-kit` | 55 | 15 | **S** | **Fuera del trinquete.** Se hace al final, por coherencia |

Comprobación aritmética: la suma de las tandas es **5.459**, el censo entero, sin solapes.

**Total: 13 tandas · 1 XL descompuesta en 5, y ninguna por encima de 750 literales.**

---

## 4. Cómo se cierra cada tanda (la puerta de salida)

| Paso | Comando | Qué tiene que pasar |
|---|---|---|
| 1 · Codemod en simulación | `_c1-codemod-espaciado.py` | Dice cuántos sitios va a tocar y **cuántos se salta**, con el motivo |
| 2 · Respaldo | automático | Un original por fichero, en `_respaldo-espaciado/`, **con su final de línea** |
| 3 · Aplicar | `--aplicar` | |
| 4 · Puerta 1 | `npx tsc --noEmit` | **0 errores.** Caza imports que falten y el prefijo duplicado |
| 5 · Puerta 2 | `npm run diseno` | `OK: ninguna zona ha empeorado` |
| 6 · Verificador | `_c2-verifica-espaciado.py` | **Declaración a declaración**, resolviendo `espaciado.eN` **leyendo `escalas.ts` del disco** |
| 7 · Puerta 3 | `npm run rutas` | Mapa al día, 0 enlaces rotos |
| 8 · Bajar la base | `npm run diseno -- --base` | **Solo si hay mejora real.** La base es el marcador, no el objetivo |

### 4.1 Qué NO se toca nunca con este codemod

- **Los comentarios.** Son prosa. **La guardia los sigue contando.** Medido: **8 literales de espaciado**
  en todo el proyecto están dentro de un comentario (0,1 %), así que la guardia tiene un **suelo de 8**
  que nunca bajará de esta familia. Es un ruido, no un problema — pero conviene saberlo antes de
  perseguir un cero que no existe.
- **Los valores de §2**: los 69 `1`, los 19 `0` y los 91 anchos de caja.
- **Las otras cinco familias.**

---

## 5. Riesgos, con lo que ya costó cada uno

| Riesgo | Por qué | Mitigación |
|---|---|---|
| **El codemod convierte CRLF en LF** | **11 de las 310 fuentes están en CRLF.** Ya pasó con `_a3` y por eso el `_a6` lo arregló | El codemod detecta el salto dominante **por fichero**. El verificador compara el fichero entero |
| **El prefijo o la llave se ponen en dos sitios** | `_a3` generó `trazo.trazo.fino`; `_a6` generó `{ { ...formaHoja } }` | **El prefijo se pone en UN sitio**. Lo caza `tsc`, no el trinquete |
| **Sube un literal en un fichero vigilado** | La guardia cuenta lo que está dentro de los comentarios | **Los valores se explican con palabras, no se copian** |
| **Un import de `espaciado` a medias** | Muchos ficheros no lo importan hoy | El codemod usa el mismo `asegura_import` del `_a6` |
| **La tanda se deja a medias** | El trinquete es por fichero | Una tanda se cierra entera o no se empieza |
| **Se cuela un rediseño** | Es la tentación de toda migración | **En la Fase 3 no se cambia ningún diseño.** Si un valor no tiene peldaño, **no se cambia el valor: se decide añadir el peldaño** |

---

## 6. Después del espaciado: el orden de las otras familias

| Familia | Usos de token | Literales | Proporción | Veredicto |
|---|---:|---:|---:|---|
| `espaciado` | 455 | **5.404** | **0,08** | **esta fase** |
| `peso` | 179 | 1.971 | **0,09** | **siguiente**: casi tan mal como el espaciado |
| `trazo` | 123 | 549 | 0,22 | después |
| `radios` | 702 | 688 | **1,02** | ya equilibrada: se remata |
| `tipografia` | 2.462 | 664 | **3,71** | ya está bien |

**`peso` es la sorpresa**: 1.971 literales en los que **1.759 ya tienen peldaño** (`maximo` cubre el 800,
que es el valor más escrito). Es una tanda **menos arriesgada que el espaciado**.

---

## 7. Lo que esta fase NO es

1. **No es un rediseño.** Ni un píxel.
2. **No es «poner tokens en todas partes».** Los 245 sin peldaño no se fuerzan: se clasifican.
3. **No entra `packages/ui-kit` en el trinquete.** Se migra al final por coherencia; añadirlo es una
   decisión aparte.
4. **No sustituye a la comprobación en el móvil.** El móvil se mira **una vez por tanda**.

---

## 8. Lo que este documento NO decide

- **Si el trinquete debe dejar de contar los comentarios.** Suena a permiso, y no lo es.
- **Las cifras `5.402` del trinquete contra `5.459` del censo nunca serán iguales** mientras el kit no
  se vigile. Se declara la diferencia (57) en vez de forzar una coincidencia.
- **Qué se hace con `packages/ui-kit`**: se migra por coherencia al final, pero añadirlo al trinquete es
  otra decisión, porque **el kit es el único sitio donde un número a mano es legítimo**.

---

## 9. Un aviso sobre `D:\egapp`: los 79 ficheros `_*` de la raíz

El `git init` de `D:\egapp` (24/09/2026, commit `93db2d3`) **incluye a propósito los 79 ficheros `_*` y
`.build-*` de la raíz** — los scripts de medición y las capturas. Se prefirió que entraran a decidir a
mano qué es basura: un `git add .` con un `.gitignore` a medias es como se pierde trabajo.

**Consecuencia que hay que saber:** sacarlos después **no los borra de la historia**. Hay que reescribir
el commit (`git rm --cached` + `filter-branch` o `rebase`), y eso es una operación que conviene hacer
**antes de que la historia crezca**. La decisión es de Bernardo, y está en la lista de abiertos.
