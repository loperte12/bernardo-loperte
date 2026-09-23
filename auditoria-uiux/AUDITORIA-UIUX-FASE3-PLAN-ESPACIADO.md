# FASE 3 · El espaciado, por tandas — plan de ejecución

**Fecha:** 24/09/2026 · **Rama:** `refactor/ui-ux-reconstruction` · **Estado:** plan aprobado, sin empezar
**Herramienta de medida:** `_b1-censo-espaciado.py` · `_b3-desglose-lifebook.py`

---

## 0. Lo que hay que desaprender antes de empezar

**La auditoría arrastraba una cifra que ya es falsa: «el 54 % del espaciado está fuera de la escala».**
Esa medida se hizo contra la escala **vieja** —base 4, once peldaños—, cuando la cobertura era del
43,9 %. La Fase 1 la cambió por 20 peldaños (base 2 + los impares + los dos anchos de contenedor) y la
cobertura pasó al 82,3 % y luego al 95,7 %. **Medido hoy, con el mismo patrón que usa la guardia:**

```
5.459 literales de espaciado     →     245 fuera de la escala = 4,5 %
```

El objetivo que la Fase 4 se había puesto —«bajar del 54 % a menos del 10 %»— **ya está cumplido, y no
lo cumplió la Fase 3: lo cumplió la reconciliación de la Fase 1.** Quien lea el número viejo va a
diseñar una fase entera para un problema que ya no existe.

**Y de ahí sale el cambio de naturaleza de esta fase, que es lo importante:**

> **Migrar el espaciado no es arreglar 5.459 valores mal puestos. Es ponerle nombre a 5.214 valores que
ya son legales.** `espaciado.e8` **es** 8. El píxel no se mueve: cambia el nombre que lo escribe.

Tres consecuencias, y las tres ordenan el trabajo:

1. **Cero píxeles por construcción**, igual que en la tanda de la lámina inferior. La diferencia es que
   allí hubo que demostrarlo con un verificador; aquí es una propiedad del renombrado.
2. **El riesgo no está en el diseño, está en el codemod.** Las tres trampas ya pagadas (el final de
   línea, el literal dentro del comentario, el prefijo en dos sitios) son *trampas de herramienta*.
3. **El criterio no lo fija el gusto de nadie, lo fija la tabla de la escala.** Por eso el orden de las
   tandas **no** es «primero lo peor, para fijar el criterio» —como decía el plan de pantallas—: para
   un renombrado el criterio ya está fijado, y lo que hay que hacer primero es **probar la herramienta
   en el sitio más pequeño**.

---

## 1. El punto de partida, medido

### 1.1 Dos cifras que no coinciden, y por qué (esto se dice antes de que alguien lo pregunte)

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

**La lectura que manda: `lifebook` es el 45 % del espaciado del proyecto, en 93 ficheros.** No es «un
módulo más»: es casi la mitad, y por eso se lleva cinco tandas propias.

### 1.3 Los cinco ficheros peores

| Literales | Fichero |
|---:|---|
| 155 | `app/conductor.tsx` |
| 138 | `app/taxi.tsx` |
| 121 | `app/edit-profile.tsx` |
| 117 | `app/tienda/publicar.tsx` |
| 108 | `app/lifebook-chat/[id].tsx` |

### 1.4 Los valores más escritos (no son excepciones: son el idioma)

`8` → 836 usos · `10` → 795 · `12` → 761 · `6` → 569 · `14` → 400 · `16` → 348 · `4` → 338 · `2` → 275

Los ocho primeros suman **4.322 usos, el 79 %**, y los ocho están en la escala. La migración es
mayoritariamente un puñado de sustituciones repetidas miles de veces.

---

## 2. Las 245 que NO son un renombrado

No se migran «todas»: **245 literales no tienen peldaño**, y son tres clases distintas con tres
decisiones distintas. Meterlas en el mismo saco sería el error que el proyecto ya cometió una vez
(declarar una base que el producto no usa).

| Clase | Usos | Qué es de verdad | Decisión propuesta |
|---|---:|---|---|
| **`1`** | 69 | Un **nudo de alineación**: `gap: 1`, `marginTop: 1`. No separa nada, ajusta óptica. | **NO se toca.** No es espaciado. Se declara como excepción aceptada y se deja de contar como deuda |
| **`0`** | 19 | Un **reinicio**: `padding: 0`, `margin: 0`. | **NO se toca.** Ídem |
| **Anchos de caja ≥ 34** (34·36·38·40·44·46·48·50·52·56·60·70·76·80·90·96·108·110·116·120·130) | 91 | **No son huecos, son tamaños**: el ancho de una tarjeta, de una miniatura, de un contenedor. | **NO van a esta familia.** Esperan a una escala de **tamaños**, que hoy no existe. Migrarlos a `espaciado` sería mentir sobre lo que son |
| **Huecos de verdad sin peldaño** (30·28·26·15·17·3,5·2,5) | **66** | Esto sí es un hueco, y la escala no lo tiene. | **La única decisión real de la fase** (abajo) |

### 2.1 La única decisión: el hueco entre 24 y 32

La escala salta de **24 a 32**, y en ese hueco viven **26 (12 usos), 28 (19) y 30 (27) = 58 usos**.

| Opción | Qué implica |
|---|---|
| **A · Añadir `e26` · `e28` · `e30`** (recomendada) | La escala pasa a 23 peldaños y cubre el **99,8 %** de los huecos. Es coherente con lo que la propia escala ya declara de sí misma: *«esto ya no es una progresión, es el conjunto de los valores en uso, con nombre y contados»*. Cero píxeles |
| **B · Unificar a 28** | Tres valores se convierten en uno: **mueve 39 sitios** y hay que mirarlos uno a uno. Es un rediseño, no un renombrado |

Y quedan **15 (4), 17 (1), 3,5 (2), 2,5 (1) = 8 usos** sueltos. Con `e16`/`e18` y `e3`/`e4` al lado,
estos ocho **se revisan a mano uno por uno** (son ocho): o se ajustan al vecino o se justifica el valor.

**Los 245 en total: 3 se aceptan como excepción, 8 se deciden a mano, 58 esperan la decisión A o B.**

---

## 3. Las tandas

**Regla de oro: una tanda se cierra entera o no se empieza.** El trinquete es por fichero: dejar un
fichero a medias lo deja peor que como estaba.

| Tanda | Alcance | Literales | Ficheros | Talla | Por qué en este orden |
|---|---|---:|---:|---|---|
| **F3.0** | **La herramienta, y la decisión de §2.1** | — | — | **M** | Sin codemod probado y sin decidir el hueco 24–32, ninguna tanda es segura |
| **F3.1** | **Ensayo:** `billing` + `kyc` + `auth` + `core` | 130 | 9 | **S** | Es la tanda más pequeña que **ejercita todas las formas**: un checkout, un flujo de pasos, un fichero CRLF si cae. Prueba el codemod donde un fallo cuesta 9 ficheros, no 93 |
| **F3.2** | `ecomerse` | 300 | 14 | **M** | Es el módulo **mejor migrado** del proyecto: sirve para validar el criterio antes de tocar nada difícil |
| **F3.3** | `food` + `rental` + `intercity` | 558 | 14 | **M** | Módulos completos y medianos; ya llevan la cabecera unificada |
| **F3.4** | `work` + `alquiler` + `profile` | 747 | 14 | **L** | Aquí viven `edit-profile` (121) y `profile` (98). Tras 3 tandas el codemod ya es aburrido |
| **F3.5** | `conductor` + `taxi` + `monedero` | 432 | 8 | **L** | **Los dos peores ficheros del proyecto** (155 y 138). Se abordan cuando el codemod ya está probado en 51 ficheros |
| **F3.6** | `app` (raíz, los 21 sin módulo) | 437 | 21 | **L** | Mezcla de pantallas sueltas; ninguna forma dominante |
| **F3.7** | `components` (sin lifebook) | 319 | 22 | **L** | Componentes compartidos: lo que se rompa aquí se nota en varias pantallas, así que va **después** de que el patrón esté probado |
| **F3.8** | **lifebook** · mensajería | 241 | 7 | **S** | Primer trozo de la mitad del proyecto: el más pequeño, para entrar |
| **F3.9** | **lifebook** · hotel | 419 | 13 | **M** | |
| **F3.10** | **lifebook** · comercio y carrito | 449 | 12 | **M** | |
| **F3.11** | **lifebook** · hojas y editores | 673 | 42 | **L** | 42 ficheros con **media de 16 literales**: muchos ficheros, poco dentro. El trinquete por fichero ayuda: 42 ficheros pequeños que se cierran rápido |
| **F3.12** | **lifebook** · feed, perfil y resto | 699 | 19 | **L** | El mayor. Contiene `lifebook-chat/[id]` (108), `lifebook-post/[id]` (89), `lifebook-user` (86) |
| — | `packages/ui-kit` | 55 | 15 | **S** | **Fuera del trinquete.** Se hace al final, por coherencia, no por presión |

Comprobación aritmética: la suma de las tandas es **5.459**, el censo entero, sin solapes — los
módulos se reparten por nombre de fichero y ninguna ruta cae en dos tandas.

**Total: 13 tandas · 1 XL descompuesta en 5 (la regla de «no hacer un cambio masivo de golpe»), y
ninguna por encima de 750 literales.** La mayor, `lifebook` · feed, es el 12,8 % del total.

---

## 4. Cómo se cierra cada tanda (la puerta de salida)

Esto no es una sugerencia: es lo que ha funcionado cinco veces y lo que convirtió una tanda fallida en
una tanda cerrada.

| Paso | Comando | Qué tiene que pasar |
|---|---|---|
| 1 · Codemod en simulación | `_c1-codemod-espaciado.py` | Dice cuántos sitios va a tocar y **cuántos se salta**, con el motivo. **Cero saltos inesperados** |
| 2 · Respaldo | automático en el codemod | Un original por fichero, en `_respaldo-espaciado/`, **con su final de línea** |
| 3 · Aplicar | `--aplicar` | |
| 4 · Puerta 1 | `npx tsc --noEmit` | **0 errores.** Caza imports que falten y el prefijo duplicado |
| 5 · Puerta 2 | `npm run diseno` | `OK: ninguna zona ha empeorado` |
| 6 · Verificador | `_c2-verifica-espaciado.py` | **Declaración a declaración**, resolviendo `espaciado.eN` **leyendo `escalas.ts` del disco**: todos los sitios idénticos. Es la prueba de «0 píxeles», medida y no razonada |
| 7 · Puerta 3 | `npm run rutas` | Mapa al día, 0 enlaces rotos |
| 8 · Bajar la base | `npm run diseno -- --base` | **Solo si hay mejora real.** La base es el marcador, no el objetivo |

### 4.1 Qué NO se toca nunca con este codemod

- **Los comentarios.** Son prosa. La consecuencia hay que decirla: la guardia **los sigue contando**.
  Medido: **8 literales de espaciado** en todo el proyecto están dentro de un comentario (0,1 %), así
  que la guardia tiene un **suelo de 8** que nunca bajará de esta familia. Es un ruido, no un problema
  — pero conviene saberlo antes de perseguir un cero que no existe. (El detalle completo, y por qué
  pasa, está en el acta de la lámina inferior.)
- **Los valores de §2**: los 69 `1`, los 19 `0` y los 91 anchos de caja.
- **Las otras cinco familias.** Esta fase es de espaciado y solo de espaciado.

---

## 5. Riesgos, con lo que ya costó cada uno

| Riesgo | Por qué | Mitigación |
|---|---|---|
| **El codemod convierte CRLF en LF** | **11 de las 310 fuentes están en CRLF.** Ya pasó con `_a3` y por eso el `_a6` lo arregló: leer con traducción de saltos y escribir con `newline=''` convierte el fichero y produce un diff de 220 líneas por tocar dos | El codemod detecta el salto dominante **por fichero** y escribe el mismo. El verificador compara el fichero entero, no solo las declaraciones |
| **El prefijo o la llave se ponen en dos sitios** | `_a3` generó `trazo.trazo.fino`; `_a6` generó `{ { ...formaHoja } }` | Regla escrita: **el prefijo se pone en UN sitio**. Y `tsc` lo caza en ambos casos — el trinquete no |
| **Sube un literal en un fichero vigilado** | La guardia cuenta lo que está dentro de los comentarios | Si el fichero se documenta, **los valores se explican con palabras, no se copian**. Ya falló dos veces por esto |
| **Un import de `espaciado` a medias** | Muchos ficheros no lo importan hoy | El codemod usa el mismo `asegura_import` del `_a6`, que ya maneja imports en varias líneas y en línea |
| **La tanda se deja a medias** | El trinquete es por fichero | Una tanda se cierra entera o no se empieza |
| **Se cuela un rediseño** | Es la tentación de toda migración | **En la Fase 3 no se cambia ningún diseño.** Si un valor no tiene peldaño, **no se cambia el valor: se decide añadir el peldaño** (§2.1) |

---

## 6. Después del espaciado: el orden de las otras familias

El espaciado es la primera pasada de la Fase 3, no la única. **El orden de las siguientes sale de la
misma cuenta emparejada** (§4.0 del plan general), no del gusto:

| Familia | Usos de token | Literales | Proporción | Veredicto |
|---|---:|---:|---:|---|
| `espaciado` → `espaciado` | 455 | **5.404** | **0,08** | **esta fase** |
| `peso` → `fontWeight` | 179 | 1.971 | **0,09** | **siguiente**, y casi tan mal como el espaciado: 1.971 literales |
| `trazo` → `borderWidth` | 123 | 549 | 0,22 | después |
| `radios` → `borderRadius` | 702 | 688 | **1,02** | ya está equilibrada: se remata, no se migra |
| `tipografia` → `fontSize` | 2.462 | 664 | **3,71** | ya está bien: lo que queda son valores sin peldaño |

**`peso` es la sorpresa**: su proporción (0,09) es la peor del proyecto junto al espaciado, y solo se
veía mirando las dos mitades emparejadas. Son 1.971 literales en los que **1.759 ya tienen peldaño**
(`maximo` cubre el 800, que es el valor más escrito del proyecto con 727 literales) — así que es una
tanda **menos arriesgada que el espaciado** y de tamaño parecido al de `lifebook` entero.

---

## 7. Lo que esta fase NO es

1. **No es un rediseño.** Ni un píxel. Si algo hay que rediseñar, se abre su propia fase y se decide
   mirando la pantalla — como se hizo con el stepper de `food-checkout`.
2. **No es «poner tokens en todas partes».** Los 245 sin peldaño no se fuerzan: se clasifican.
3. **No entra `packages/ui-kit` en el trinquete.** Se migra al final por coherencia; la guardia no lo
   vigila y añadirlo es una decisión aparte (el kit es el único sitio donde un valor a mano es legítimo).
4. **No sustituye a la comprobación en el móvil.** 5.459 sustituciones que no mueven un píxel se
   demuestran con el verificador; el móvil se mira **una vez por tanda**, y no por fichero.

---

## 8. Lo que este documento NO decide

- **La opción A o B de §2.1.** Es de Bernardo: implica o no mover 39 sitios.
- **Si el trinquete debe dejar de contar los comentarios.** Suena a permiso, y no lo es: bajaría el
  contador de cinco familias de golpe. Es una decisión de método, no de diseño, y va con el resto.
- **Las cifras `5.402` del trinquete contra `5.459` del censo nunca serán iguales** mientras el kit no
  se vigile. Se declara la diferencia (57) en vez de forzar una coincidencia.
- **Qué se hace con `packages/ui-kit`**: se migra por coherencia al final, pero añadirlo al trinquete es
  otra decisión, porque **el kit es el único sitio donde un número a mano es legítimo** (es su trabajo
  definir el valor).
