# Auditoría UI/UX — EG Route Plan · rama `refactor/ui-ux-reconstruction`

Esta rama reúne **la auditoría medida, los tokens reconciliados, el plan de reconstrucción y las actas de
lo que ya se ha aplicado**. No toca `main`: el trabajo llega como Pull Request.

> **AVISOS ANTES DE LEER NADA, Y TODOS SON CORRECCIONES DE ERRORES PROPIOS:**
>
> 1. **La cifra «1,02» de §1 del informe está RETIRADA.** Divide todos los usos de token (10 familias)
>    entre los literales de cuatro, metiendo en el numerador los 455 usos de `espaciado` y dejando fuera
>    del denominador sus 5.404 literales. La comparación emparejada da **0,42**. Está en el plan §4.0.
> 2. **El «micrófono del buscador a 28 dp» del §3.5 NO EXISTE.** Era una nota sin comprobar. El censo real
>    es 152 objetos por debajo de 44 dp, 39 de ellos con nombre de control.
> 3. **La tabla §7 del acta de la cabecera estaba MAL, y es la corrección más incómoda.** Decía «el título
>    sube 16 → 17 en 6» (ninguno de los 20 tiene 16), «nada cambia en 4» (los cuatro cambian) y, en el
>    docblock del componente, «19 volveres no hacían nada al pulsarse» (**los 20 navegaban**; lo que
>    faltaba era la señal visual). La escribí **leyendo el componente en vez de midiendo los ficheros**.
>    Corregida con `_c0` y `_c1`, que la reproducen. El desarrollo está en el §7 y el §9 del acta.
> 4. **En `AUDITORIA-UIUX-TOKENS.md`, el título de §4.1 se contradecía con su propio texto** (decía
>    «bases de 4 y 2 compatibles con los impares» cuando el texto explica que una base 2 no puede
>    expresar un impar). Corregido, con la ley que lo resuelve. **Ese fichero es de 38 KB y no cabe por
>    el conector: la corrección viaja en el commit de `D:\Routeplan`.**

## Qué es instantánea y qué está al día

| | |
|---|---|
| **Instantánea del 24/09/2026** (no se reescribe: es la medición de ese día) | `AUDITORIA-UIUX-INFORME.md` · `AUDITORIA-UIUX-TOKENS.md` · `AUDITORIA-UIUX-SALIDA-HERRAMIENTAS.md` · `AUDITORIA-UIUX-GITHUB-BLOQUEO.md` |
| **Al día** | `AUDITORIA-UIUX-PLAN-RECONSTRUCCION.md` · `TANDA-FASE2-LIFEBOOK-CIERRE.md` y `TANDA-FASE2-CABECERA-PILOTO.md` · `AUDITORIA-UIUX-FASE3-PLAN-ESPACIADO.md` · `RESPUESTA-24SEP-CUATRO-PUNTOS.md` · este índice |

Consecuencia práctica: **si una cifra de la instantánea contradice el plan o un acta, manda el acta.**

## Por qué hay copias de código aquí dentro

> **ACTUALIZACIÓN DEL 24/09/2026: `D:\egapp` YA TIENE HISTORIA.** Hasta hoy no era un repositorio git en
> absoluto; ahora tiene el commit **`93db2d3` «Punto de partida del proyecto EG Route Plan»** (**1.129
> ficheros, 1,4 MB, árbol limpio**), rama `main`, y un segundo commit `7ecc10b`. **Se puede volver atrás
> de un comando.**
>
> **Pero esto NO vuelve innecesaria la carpeta `tokens/`**, por dos razones: (1) **la historia es local**
> —no hay credencial git en mi entorno (`fatal: could not read Username for 'https://gitee.com'`) y `gh`
> no está instalado—, así que **no se ha podido subir a ningún sitio**; si el disco muere, muere; y (2)
> `93db2d3` es un **punto de partida**, no una red: recupera el estado de hoy, no los 73 ficheros de
> trabajo con su ruta real, que es lo que permite aplicar un cambio sin buscarlo.

**El código de la aplicación sigue sin estar versionado en el remoto.** El único repositorio con remoto
es `D:\Routeplan`, y **apunta a Gitee** (`gitee.com/Bernardo12/eg-route-plan.git`), no a GitHub.

Consecuencia, y es la razón de la carpeta `tokens/`: los cambios de la Fase 1 y los de la Fase 2 (los
primitivos, los campos, la geometría de la lámina inferior y la cabecera) **solo existen en el disco de
`D:\egapp`** y en esa carpeta. Para que viajen con la auditoría se copian aquí **tal cual**, conservando
su ruta real: aplicarlos es copiarlos a su sitio.

## Qué hay

| Fichero | Qué es |
|---|---|
| `AUDITORIA-UIUX-INFORME.md` | La auditoría: cómo se midió, el perfil real (310 ficheros · 4.009 usos de token) y los 7 hallazgos |
| `AUDITORIA-UIUX-TOKENS.md` | Los tokens, peldaño a peldaño, **con el número de literales que pide cada uno al lado**. §9: **formas repetidas** |
| `AUDITORIA-UIUX-PLAN-RECONSTRUCCION.md` | El plan en 4 fases: estado real, esfuerzo, riesgos y las tres puertas. §4.0 corrige el objetivo de la Fase 4 |
| `TANDA-FASE2-LIFEBOOK-CIERRE.md` | La lámina inferior unificada en 19 sitios, 0 píxeles movidos y cómo se demuestra |
| `TANDA-FASE2-CABECERA-PILOTO.md` | El componente `ScreenHeader`, el piloto aplicado, **la comprobación en el móvil medida al píxel** (icono 22,0 → 24,0 dp, fila sin mover) y **§7 corregida** |
| `RESPUESTA-24SEP-CUATRO-PUNTOS.md` | La respuesta a las cuatro decisiones de Bernardo, y la medición de las 20 cabeceras |
| `AUDITORIA-UIUX-FASE3-PLAN-ESPACIADO.md` | El plan de la Fase 3 en 13 tandas. Y la corrección que lo cambia todo: el «54 % fuera de escala» era de la escala vieja — hoy es el **4,5 %** |
| `_f2-piloto-cabecera-ANTES.png` · `_f2-piloto-cabecera-food-orders.png` | Los dos pantallazos del piloto en el PKK110. Se comparan con `_b4`, no a ojo |

## Lo que esta rama NO trae

**La migración masiva de literales a token** (1.970 `fontWeight`, 663 `fontSize`, 688 `borderRadius`,
548 `borderWidth`, 5.402 de espaciado). Es la **Fase 3** y no ha empezado: la Fase 1 declaró las
escalas, que era el problema de verdad, no las aplicó. La proporción emparejada está en **0,42**.

## Cómo se comprueba

```bash
cd D:\egapp
npx tsc --noEmit        # 0 errores
npm run diseno           # OK: ninguna zona ha empeorado. El trinquete solo permite bajar
npm run rutas           # mapa de rutas al día, 0 enlaces rotos
```

Estado del trinquete el 24/09/2026, **tras el piloto del `ScreenHeader`** (−5 en un solo fichero):
`hex 189 · fontSize 663 · borderRadius 688 · fontWeight 1970 · borderWidth 548 · espaciado 5402 · precioFigura 12`

Es la única de las tres puertas que **puede** bajar sola: `tsc` y `npm run rutas` no cuentan deuda, la
detectan. Por eso la base del trinquete se baja **solo cuando hay mejora real**.
