# Auditoría UI/UX — EG Route Plan · rama `refactor/ui-ux-reconstruction`

Esta rama reúne **la auditoría medida, los tokens reconciliados, el plan de reconstrucción y las actas de
lo que ya se ha aplicado**. No toca `main`: el trabajo llega como Pull Request.

> **DOS AVISOS ANTES DE LEER NADA, Y LOS DOS SON CORRECCIONES DE ERRORES PROPIOS:**
>
> 1. **La cifra «1,02» de §1 del informe está RETIRADA.** Divide todos los usos de token (10 familias)
>    entre los literales de cuatro, metiendo en el numerador los 455 usos de `espaciado` y dejando fuera
>    del denominador sus 5.404 literales. La comparación emparejada da **0,42**. El desarrollo está en
>    `AUDITORIA-UIUX-PLAN-RECONSTRUCCION.md` §4.0.
> 2. **El «micrófono del buscador a 28 dp» del §3.5 NO EXISTE.** Era una nota sin comprobar. El censo real
>    es 152 objetos por debajo de 44 dp, 39 de ellos con nombre de control.

## Qué es instantánea y qué está al día

| | |
|---|---|
| **Instantánea del 24/09/2026 por la mañana** (no se reescribe: es la medición de ese día) | `AUDITORIA-UIUX-INFORME.md` · `AUDITORIA-UIUX-TOKENS.md` · `AUDITORIA-UIUX-SALIDA-HERRAMIENTAS.md` · `AUDITORIA-UIUX-GITHUB-BLOQUEO.md` |
| **Al día** | `AUDITORIA-UIUX-PLAN-RECONSTRUCCION.md` (estado de las fases y §4.0) · `TANDA-FASE2-LIFEBOOK-CIERRE.md` y `TANDA-FASE2-CABECERA-PILOTO.md` (lo aplicado) · `AUDITORIA-UIUX-FASE3-PLAN-ESPACIADO.md` (lo que viene) · este índice |

Consecuencia práctica: **si una cifra de la instantánea contradice el plan o un acta, manda el acta.** Las
dos contradicciones conocidas están avisadas arriba y desarrolladas en el plan (§4.0) y en el acta (§0).

**La sección §9 del documento de tokens —«formas repetidas»— ya está subida**, y su §9.2, que decía
«medido, no tocado», está **reescrita**: el encabezado de pantalla pasó de hallazgo a pieza cerrada
(`ScreenHeader`). El censo completo sigue también en el §0 del acta de la lámina inferior.

## Por qué hay copias de código aquí dentro

**El código de la aplicación no está versionado en ninguna parte.** `D:\egapp` no es un repositorio git, y
el único repositorio del proyecto (`D:\Routeplan`) rastrea 11 ficheros: `git ls-files` da **0
coincidencias** para `packages/ui-kit` y para `app/`.

Consecuencia, y es la razón de la carpeta `tokens/`: los cambios de la Fase 1 (los 6 tokens de texto sobre
fondo oscuro, la rampa de neutros, las escalas reconciliadas) y los de la Fase 2 (los primitivos, los
campos y la geometría de la lámina inferior) **solo existen en el disco de `D:\egapp`**. Para que viajen
con la auditoría se copian aquí **tal cual**, conservando su ruta real: aplicarlos es copiarlos a su sitio.

## Qué hay

| Fichero | Qué es |
|---|---|
| `AUDITORIA-UIUX-INFORME.md` | La auditoría. Cómo se midió, el perfil real (310 ficheros · 4.009 usos de token), los 7 hallazgos con su «por qué», y lo que **no** se midió |
| `AUDITORIA-UIUX-TOKENS.md` | Los tokens, peldaño a peldaño, **con el número de literales que pide cada uno al lado**. §9 es nueva: **formas repetidas**, el hueco que ni el trinquete ni el censo de adopción veían |
| `AUDITORIA-UIUX-PLAN-RECONSTRUCCION.md` | El plan en 4 fases: estado real, esfuerzo, riesgos y las tres puertas de cada fase. §4.0 corrige el objetivo de la Fase 4 |
| `TANDA-FASE2-LIFEBOOK-CIERRE.md` | **Acta de la tanda del 24/09/2026:** la lámina inferior unificada en 19 sitios, 0 píxeles movidos y cómo se demuestra |
| `TANDA-FASE2-CABECERA-PILOTO.md` | **Acta de la cabecera de pantalla:** el componente `ScreenHeader` del kit, el piloto aplicado y **la comprobación en el móvil medida al píxel** (icono 22,0 → 24,0 dp, fila sin mover) |
| `AUDITORIA-UIUX-FASE3-PLAN-ESPACIADO.md` | **El plan de la Fase 3 en 13 tandas.** Y la corrección que lo cambia todo: el «54 % fuera de escala» era de la escala vieja — hoy es el **4,5 %**, así que migrar espaciado es **ponerle nombre a lo que ya es legal**, no arreglar nada |
| `_f2-piloto-cabecera-ANTES.png` · `_f2-piloto-cabecera-food-orders.png` | Los dos pantallazos del piloto **en el móvil de referencia** (PKK110): el build de ayer y el de hoy. Se comparan con `_b4`, no a ojo |
| `AUDITORIA-UIUX-SALIDA-HERRAMIENTAS.md` | Anexo crudo: lo que devolvió cada herramienta, incluido **lo que se descartó y por qué** |
| `AUDITORIA-UIUX-GITHUB-BLOQUEO.md` | El bloqueo del conector con las pruebas literales, y la ruta de salida |
| `lienzo-uiux.html` | El lienzo visual: cuatro pantallas a tamaño real con los tokens reales, el antes/después de modo oscuro, la comparación tipográfica A/B y el peso `medio` frente a `fuerte` |
| `_a1-censo-diseno.py` | Censo reproducible de adopción de tokens. Imprime la proporción histórica **y la emparejada** |
| `_a2-codemod-espaciado.py` | Renombrado de las claves de `espaciado` (450 usos, 34 ficheros) |
| `_a3-codemod-altura.py` | Adopción de `altura.*` en la app y migración de `app/ecomerse.tsx` (−28 literales) |
| `_a5-censo-formas-repetidas.py` | **Nuevo.** Encuentra la misma forma escrita en varios ficheros comparando solo las declaraciones. 110 formas en 3+ ficheros |
| `_a6-codemod-forma-hoja.py` | **Nuevo.** Unifica la geometría de la lámina inferior. Respeta el final de línea y escribe respaldo |
| `_a7-verifica-forma-hoja.py` | **Nuevo.** Comprueba, contra el respaldo, que los 19 sitios dejaron las mismas declaraciones: **0 píxeles movidos, medido** |
| `_b1-censo-espaciado.py` | **Nuevo.** El censo de espaciado por módulo, **con el mismo patrón que la guardia**. Dice qué parte de los 5.459 está fuera de la escala (245) y agrupa lo que queda fuera |
| `_b2-medir-titulos-cabecera.py` | **Nuevo.** Qué títulos de cabecera caben en una línea a 17 px, con la calibración de ancho de letra ya contrastada contra el teléfono |
| `_b3-desglose-lifebook.py` | **Nuevo.** Parte `lifebook` —el 45 % del espaciado del proyecto— en cinco sub-áreas reales, para que sus cinco tandas tengan sentido |
| `_b4-mide-cabecera.py` | **Nuevo.** Descodifica un PNG de `screencap` a mano (no hay Pillow ni numpy) y mide la cabecera en la captura: la fila del borde, el trazo del icono y su centro. Deduce el tamaño del icono desde los píxeles |
| `tokens/` | Los ficheros de código tal como quedaron, porque la app no está versionada |

## AVISO IMPORTANTE: qué falta aquí, y qué no se puede subir por esta vía

**El conector de GitHub ya escribe** desde el 24/09/2026: `push_files` sube varios ficheros en un solo
commit, pero **con el contenido en línea en la llamada**. La vía directa por la API sigue cerrada —la
credencial de la configuración local del conector es un marcador, no el token real: `401 Bad credentials`—.

**Lo que está en esta rama:** los seis documentos de la auditoría, las **tres actas** de lo aplicado, el
plan de la Fase 3 y este índice. Todo eso es texto y viaja bien.

**Lo que NO puede viajar por el conector, y por qué:** el juego completo son ~20 ficheros. De ellos,

- los **dos pantallazos del piloto** (`_f2-piloto-cabecera-*.png`, 125 KB cada uno) son **binarios**: el
  conector exige texto, así que no entran ni con `push_files` ni con `create_or_update_file`;
- los **codemods y censos** (`_a*`, `_b*`) y las **copias de `tokens/`** (73 ficheros de código, porque la
  app no está versionada) sí son texto, y se subirán fichero a fichero.

**Todo ello está en `D:\Routeplan\auditoria-uiux\`** — commit local `0e693b7`, rama
`refactor/ui-ux-reconstruction`. Para empujarlo de una vez:

```bash
cd /d/Routeplan && git push -u origin refactor/ui-ux-reconstruction
```

Nada de esto es una pérdida: **todo existe en el disco de la máquina** (`C:\Users\nisang12\WorkBuddy\
2026-09-18-21-02-51\auditoria-uiux\` y `D:\Routeplan\auditoria-uiux\`). Lo que falta es el transporte.

## Lo que esta rama NO trae

**La migración masiva de literales a token** (1.970 `fontWeight`, 663 `fontSize`, 688 `borderRadius`,
548 `borderWidth`, 5.402 de espaciado). Es la **Fase 3** y no ha empezado: la Fase 1 declaró las escalas
—que era el problema de verdad—, no las aplicó. La proporción emparejada está en **0,42**. **El plan de
la Fase 3, en 13 tandas, sí está aquí** (`AUDITORIA-UIUX-FASE3-PLAN-ESPACIADO.md`), y su §0 explica por
qué migrar espaciado es ponerle nombre a valores legales y no arreglar un 54 % que ya no existe.

## Cómo se comprueba

```bash
cd D:\egapp
npx tsc --noEmit        # 0 errores
npm run diseno          # OK: ninguna zona ha empeorado. El trinquete solo permite bajar
npm run rutas           # mapa de rutas al día, 0 enlaces rotos
```

Estado del trinquete el 24/09/2026, **tras la tanda de la cabecera de pantalla** (el piloto del
`ScreenHeader`, que bajó −5 en un solo fichero):
`hex 189 · fontSize 663 · borderRadius 688 · fontWeight 1970 · borderWidth 548 · espaciado 5402 · precioFigura 12`

Es la única de las tres puertas que **puede** bajar sola: `tsc` y `npm run rutas` no cuentan deuda, la
detectan. Por eso la base del trinquete se baja **solo cuando hay mejora real** y es el marcador, no el
objetivo.
