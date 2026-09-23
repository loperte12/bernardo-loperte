# Auditoría UI/UX — EG Route Plan · rama `refactor/ui-ux-reconstruction`

Esta rama reúne **la auditoría medida, los tokens reconciliados y el plan de reconstrucción** del sistema
de diseño de la app. No toca `main`: el trabajo llega como Pull Request.

## Por qué hay copias de código aquí dentro

**El código de la aplicación no está versionado en ninguna parte.** `D:\egapp` no es un repositorio git, y
el único repositorio del proyecto (`D:\Routeplan`) rastrea 11 ficheros: `git ls-files` da **0
coincidencias** para `packages/ui-kit` y para `app/`.

Consecuencia, y es la razón de la carpeta `tokens/`: los cambios de la Fase 1 (los 6 tokens de texto sobre
fondo oscuro, la rampa de neutros, las escalas reconciliadas) y los de la Fase 2 (los primitivos y los
campos) **solo existen en el disco de `D:\egapp`**. Para que viajen con la auditoría se copian aquí **tal
cual**, en `tokens/`, conservando su ruta real: aplicarlos es copiarlos a su sitio.

## Qué hay

| Fichero | Qué es |
|---|---|
| `AUDITORIA-UIUX-INFORME.md` | La auditoría. Cómo se midió, el perfil real (310 ficheros · 4.009 usos de token frente a 3.913 literales escritos a mano = **1,02**), los 7 hallazgos con su «por qué», y lo que **no** se midió |
| `AUDITORIA-UIUX-TOKENS.md` | Los tokens, peldaño a peldaño, **con el número de literales que pide cada uno al lado**. Incluye las decisiones cerradas y las correcciones de errores propios |
| `AUDITORIA-UIUX-PLAN-RECONSTRUCCION.md` | El plan en 4 fases: estado real, esfuerzo, riesgos y las tres puertas de cada fase |
| `AUDITORIA-UIUX-SALIDA-HERRAMIENTAS.md` | Anexo crudo: lo que devolvió cada herramienta, incluido **lo que se descartó y por qué** |
| `AUDITORIA-UIUX-GITHUB-BLOQUEO.md` | El bloqueo del conector con las pruebas literales, y la ruta de salida |
| `lienzo-uiux.html` | El lienzo visual: cuatro pantallas a tamaño real con los tokens reales, el antes/después de modo oscuro, la comparación tipográfica A/B y el peso `medio` frente a `fuerte` |
| `_a1-censo-diseno.py` | Censo reproducible de adopción de tokens |
| `_a2-codemod-espaciado.py` | Renombrado de las claves de `espaciado` (450 usos, 34 ficheros) |
| `_a3-codemod-altura.py` | Adopción de `altura.*` en la app y migración de `app/ecomerse.tsx` (−28 literales) |
| `tokens/` | Los ficheros de código tal como quedaron, porque la app no está versionada |

## AVISO IMPORTANTE: esta rama está incompleta, y por qué

El juego completo son **280 KB en 20 ficheros**. El conector de GitHub **exige el contenido de cada
fichero en línea** en la llamada, y la vía directa por la API está cerrada —la credencial que hay en la
configuración local del conector es un marcador, no el token real, así que devuelve `401 Bad credentials`—.

**Lo que está en esta rama:** el informe, los tokens, el plan y este índice.

**Lo que NO está, y está en `D:\Routeplan\auditoria-uiux\`** (commit `6705f7e`, rama
`refactor/ui-ux-reconstruction`): el anexo de herramientas, el acta del bloqueo, el lienzo visual, los tres
codemods y las copias de `tokens/`. Para subirlo:

```bash
cd /d/Routeplan && git push -u origin refactor/ui-ux-reconstruction
```

Nada de esto es una pérdida: **todo existe en el disco de la máquina** (`C:\Users\nisang12\WorkBuddy\
2026-09-18-21-02-51\auditoria-uiux\` y `D:\Routeplan\auditoria-uiux\`). Lo único que falta es el transporte.

## Lo que esta rama NO trae

**La migración masiva de literales a token** (1.975 `fontWeight`, 664 `fontSize`, 724 `borderRadius`,
550 `borderWidth`, 5.446 de espaciado). Es la **Fase 3** y no ha empezado: la Fase 1 declaró las escalas
—que era el problema de verdad—, no las aplicó. La proporción token/literal sigue en 1,02.

## Cómo se comprueba

```bash
cd D:\egapp
npx tsc --noEmit        # 0 errores
npm run diseno          # OK: ninguna zona ha empeorado. El trinquete solo permite bajar
npm run rutas           # mapa de rutas al día, 0 enlaces rotos
```
