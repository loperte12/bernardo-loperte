# Acta · TANDA FASE 2 — la lámina inferior (`lifebook`)

**Fecha:** 24/09/2026 · **Rama:** `refactor/ui-ux-reconstruction` · **Fase:** 2 (Componentes Base), 5.ª pieza
**Puertas:** `tsc` 0 errores · `npm run diseno` OK (base **bajada**) · `npm run rutas` todo en orden

---

## 0. El censo que faltaba: las FORMAS repetidas

**Esta tanda estrena una dimensión que la auditoría no medía, y era un hueco del método.** Las siete
familias del documento de tokens (color, tamaño, peso, espaciado, trazo, radio, altura) son familias de
*valores*. El trinquete cuenta literales y el censo `_a1` cuenta adopción de tokens: **ninguno de los dos ve
una forma repetida**, porque una forma repetida no gasta ningún literal — gasta **sitios**. Diecisiete
ficheros escribiendo siete valores idénticos suman cero deuda a mano y, sin embargo, son diecisiete sitios
que hay que editar a la vez.

Medidor nuevo: **`_a5-censo-formas-repetidas.py`**. Extrae cada `StyleSheet.create` del proyecto, parte
cada entrada `nombre: { … }` y compara **solo las declaraciones** —no el nombre de la clave—, con espacios
y orden normalizados. Así `handle` y `barra` con el mismo cuerpo cuentan como la misma forma.

**Resultado global, medido el 24/09/2026:** 310 ficheros · 2.831 entradas de estilo ·
**110 formas que aparecen en 3 o más ficheros**. Las cuatro mayores:

| Forma | Ficheros | Qué es | Estado |
|---|---:|---|---|
| `alignItems:'center' flex:1 justifyContent:'center'` | 28 | el centrado de un estado vacío o de carga | sin dueño |
| **`header`** — `flexDirection:'row' justifyContent:'space-between' borderBottomWidth:1 paddingHorizontal:16 paddingVertical:12` | **21** | **el encabezado de pantalla** | **medido, SIN TOCAR** (§5) |
| **`sheet`** — absoluta pegada abajo, dos radios de 22, relleno de 18 | **17** | **la lámina inferior** | **CERRADA en esta tanda** (§1–3) |
| `topBar` — como `header` pero con `gap:10` y `paddingHorizontal:12` | 10 | la barra superior de lifebook | sin dueño |

---

## 1. Qué se ha cerrado

**La forma que se repetía en `lifebook` no era un componente: era una geometría.** La lámina inferior
—posición absoluta pegada abajo, ancho completo, dos esquinas de arriba redondeadas y un relleno— estaba
escrita **a mano en 17 ficheros**, idéntica byte a byte, bajo cuatro nombres distintos (`sheet`,
`menuSheet`, `userMenuSheet`, `sheetCard`). El tirador estaba en 3.

Y **uno de los 17 era su propio dueño**: `components/lifebook/ui/Sheet.tsx` ya declaraba ese mismo cuerpo
en `sheetStyles.sheet`, **exportado y sin que lo importara nadie**. Es el mismo patrón que `altura.boton`
en la pieza anterior: lo que hacía falta no había que crearlo, había que usarlo.

### Cómo se ha cerrado

No creando un componente —eso obligaría a tocar el marcado de 17 pantallas y no es una tanda, es un
rediseño— sino **exportando la geometría** desde el fichero que ya la poseía, para que cada sitio la
extienda:

```ts
export const formaHoja = {
  position: 'absolute', left: 0, right: 0, bottom: 0,
  borderTopLeftRadius: 22, borderTopRightRadius: 22, padding: 18,
} as const;

export const formaTirador = {
  width: 40, height: 4, borderRadius: 2, alignSelf: 'center', marginBottom: 14,
} as const;
```

y en cada sitio, una línea: `sheet: { ...formaHoja }` · `handle: { ...formaTirador }`.

| | |
|---|---|
| Sitios unificados | **19** (17 hojas + 2 tiradores) en **16 ficheros** + el dueño |
| De ellos en `lifebook` | **16 de 17**; el restante es `app/settings.tsx` (`sheetCard`), misma firma |
| Variante conservada | `ChatOptionsSheet` tenía una declaración más: `{ ...formaHoja, maxHeight: '84%' }` |
| Píxeles movidos | **0 — medido, no razonado** (§3) |

---

## 2. Las tres puertas, con los números

```
npx tsc --noEmit     → 0 errores
npm run diseno       → OK: ninguna zona ha empeorado
npm run rutas        → mapa al día · 0 enlaces rotos · 0 pantallas huérfanas
```

**Y el trinquete ha bajado, que es la prueba de que se trabajó:**

| Familia | Antes | Después |
|---|---:|---:|
| `borderRadius` | 724 | **688** (−36) |
| `espaciado` | 5.423 | **5.404** (−19) |
| `fontSize` · `fontWeight` · `borderWidth` · `hex` | 664 · 1.971 · 549 · 189 | sin cambio |

La aritmética cuadra y se puede comprobar sola: 18 hojas × 2 radios = 36; 17 hojas + 2 tiradores = 19
espaciados. Los ficheros no ganaron literales porque **los que se quitaron no se movieron a otro sitio**:
se sustituyeron por una extensión que no contiene ningún número.

---

## 3. Cómo se demuestra que no se movió un píxel

Decir «es el mismo valor» es exactamente el tipo de afirmación que se cree sin comprobar. Se comprueba:

**`_a7-verifica-forma-hoja.py`** — para cada uno de los 19 sitios recupera el cuerpo **original** desde el
respaldo que el codemod escribe antes de tocar nada, y lo compara declaración a declaración con el cuerpo
actual, **resolviendo `...formaHoja` leyendo la constante real del disco** (no lo que el script crea
recordar).

```
OK  app/lifebook-chat/[id].tsx (sheet)          — 7 declaraciones, idénticas
…
OK  components/lifebook/messaging-sheets.tsx (handle) — 5 declaraciones, idénticas
OK  components/lifebook/ChatOptionsSheet.tsx (sheet)  — 8 declaraciones, idénticas

identicos: 19 · distintos: 0 · sin respaldo: 0
```

---

## 4. Dos trampas que costaron un intento cada una

Van aquí porque las dos son del mismo tipo: **cosas que la guardia y el compilador no ven.**

### 4.1 El trinquete cuenta los literales que están DENTRO de los comentarios

El primer intento **falló la puerta de diseño** por un solo fichero, y el fichero era el mío: el bloque de
comentario que explicaba el cambio citaba una declaración de relleno **con su cifra**. El trinquete la
contó como deuda (`espaciado 4 → 5`). Se arregló… y el aviso que advertía de eso **la volvió a citar**, así
que falló una segunda vez. La regla queda escrita en el propio fichero: **en un fichero vigilado, los
valores se explican con palabras, no se copian.** El `_a3` ya lo había aprendido por su lado.

> Consecuencia para la Fase 3: migrar literales dentro de comentarios **no** es trabajo pendiente, es ruido
> del medidor. Cuando se migre en serio, conviene decidir si la guardia debe ignorar los comentarios —
> hoy cuenta de más, y quien lo descubra por su cuenta va a perder el mismo rato.

### 4.2 Un codemod que no respeta el final de línea convierte CRLF en LF

**11 de las 310 fuentes del proyecto están en CRLF**, y una de ellas (`messaging-sheets.tsx`) es de esta
tanda. El primer codemod leía con traducción de saltos y escribía con `newline=''`: habría convertido el
fichero a LF y el cambio saldría como un diff de **220 líneas** donde se tocaron **dos**. Ahora el codemod
detecta el salto dominante del fichero y escribe el mismo que tenía.

**Y hay que decir que el `_a3` (la pieza anterior) sí tenía ese fallo.** No está comprobado que dañara
ningún fichero —no se guardó respaldo de los finales de línea, así que **no se puede saber**—, y por eso se
enuncia como riesgo y no como hecho. El respaldo del `_a6` sí conserva el original, y por eso esta vez sí
se puede afirmar.

---

## 5. Lo que se midió y NO se tocó (y por qué)

| Candidato | Medición | Decisión |
|---|---|---|
| **`ACCENT` en 6 ficheros de `food`** | No es una copia de color: es `const ACCENT = brand.primary` — **apunta al token** | **Cero deuda.** No se toca. Se midió para no «arreglar» algo que estaba bien |
| **El encabezado de pantalla** | **21 ficheros** (6 en `food`: `header` ×6, `headerTitle` ×5). Es la forma más repetida del proyecto. Firma idéntica: `flexDirection:'row' justifyContent:'space-between' borderBottomWidth:1 borderBottomColor:c.border paddingHorizontal:16 paddingVertical:12` | **NO SE TOCA.** No tiene dueño y su cuerpo referencia el tema (`c.border`), así que no puede exportarse como constante plana. Necesita **dos decisiones de Bernardo**: **(1) dónde vive el dueño** —¿primitiva nueva en `packages/ui-kit/src/primitives/`, coherente con `StepHeader`, o módulo compartido de la app?— y **(2) si se unifica o se respeta** — puede que las 21 variantes sean legítimamente distintas: unas llevan acción a la derecha, otras no, unas llevan subtítulo. Que las **declaraciones** coincidan no significa que el control sea el mismo |
| `food-rider` / `food-orders` / `food-owner` | Deuda alta (34p/9l/13r, 28p/9l/14r, 27p/11l/8r), pero **es deuda de literales** | **Fase 3**, no Fase 2 — y Bernardo ya dijo que la migración literal espera a la Fase 3 |

---

## 6. Lo que esta tanda deja abierto

1. **La decisión del encabezado de pantalla** (21 ficheros). Las dos preguntas concretas están en §5 de
   esta misma acta y en §9.2 del documento de tokens (esa sección vive en la copia local, aún sin subir:
   ver el aviso del `LEEME.md`). Es el siguiente trabajo natural de la Fase 2 y **está bloqueado por
   decisión, no por capacidad**.
2. **Fase 3 sin empezar**, y con una prioridad que ha cambiado tras la corrección de la proporción
   (§4.0 del plan): **`espaciado` es el trabajo grande** — 5.404 literales frente a 455 usos de token —
   no `fontWeight`.
3. **El respaldo de esta tanda** está en `_respaldo-forma-hoja/` del workspace de auditoría, con los 17
   originales. **`D:\egapp` no está versionado**: mientras siga así, este respaldo es la única copia
   recuperable de esos ficheros.
4. **Ojo al leer los documentos grandes de la rama:** el informe, los tokens y el anexo de herramientas
   son la instantánea del 24/09 por la mañana. **Lo aplicado después está en las actas**, y las cifras
   que hayan quedado desfasadas están corregidas aquí y en el §4.0 del plan.
