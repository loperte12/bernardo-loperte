# Respuesta a los cuatro puntos — 24/09/2026

> **Resumen en una linea por punto.** (1) El reemplazo de las 20 cabeceras **empieza hoy**, y antes de
> tocar nada medi las 20 una a una: **la tabla del plan no cuadraba**. (2) La decision A **ya esta
> aplicada y verificada** — delta 0 en la guardia. (3) El push **no es un problema de permisos**: es
> que en mi entorno **no hay credencial git**; te doy el comando exacto. (4) El `git init` de
> `D:\egapp` **ya esta hecho**, con los comandos exactos y lo que entro a proposito.

> **NOTA:** el fichero completo (15.738 caracteres) viaja por el commit de `D:\Routeplan`; esta copia
> se sube por el conector, que tiene limite de tamano. El original con acentos esta en la rama.

---

## 1 · Las 20 cabeceras — la tabla del acta estaba mal

Aprobaste el reemplazo. Lo primero que hice **no** fue tocar ficheros: fue comprobar que la lista y
los cambios prometidos en `TANDA-FASE2-CABECERA-PILOTO.md` §7 fueran ciertos. **No lo eran del todo.**

### Hallazgo 1: la lista de «las otras 20» no estaba escrita en ningun sitio

El acta dice «las otras 20» pero no las enumera, y el censo que las encontro (`_a5`) **corta la lista
con «... y 12 mas»**. Asi que escribi `_c0-lista-cabeceras.py`, que reproduce la clave del censo y
devuelve **exactamente 20 ficheros** con la forma `header`. El numero del plan es correcto. Reparto:
5 `ecomerse` · 5 `food` · 3 `alquiler` · 3 `work` · 2 `billing` · 1 `landlord-profile` · 1 `intercity`.

Avisa ademas de **56 ficheros mas que tienen un `header:` pero NO esa forma** (los `lifebook-*`, los
`monedero-*`, las laminas...). Esos no entran: unos usan `hairlineWidth` en vez de `1`, otros llevan
`gap`. Meterlos en la misma tanda seria un rediseno disfrazado de unificacion.

### Hallazgo 2: la tabla de §7 tiene dos errores

`_c1-mide-20-cabeceras.py` lee, fichero a fichero, el valor real de cada decision del componente:

| Lo que decia §7 | Lo que dice la medicion |
|---|---|
| «El titulo sube **16 -> 17** en **6**» | **Ninguno de los 20 tiene 16.** 14 a **17**; los otros 6 usan `tipografia.subtitle` |
| «**Nada** cambia en 4: `billing-checkout`, `billing-status`, `food-checkout`, `work-*`» | Los cuatro **si cambian** (tope de linea, icono y/o peso) |
| «El titulo sube 700 -> 800 en 7» | Correcto: **7** |
| «El icono sube 22 -> 24 en 11» | Correcto: **11** |
| «El volver responde al toque en 19» | **Los 20 ya navegaban** — ver abajo |

**Como se colaron.** Mi primera version del medidor buscaba el primer `fontSize` del fichero, y en
`billing-status` encuentra un **38** (un emoji de estado) en vez del 17 del titulo. Y para leer los
atributos del volver use `[^>]*`, que **se corta en el `>` de `() =>`** y devolvia `onPress={() =`
— por eso «media» que los 20 volveres no tenian rol cuando **19 si lo tienen**.

### La correccion que cambia el trabajo: el volver ya funcionaba

El docblock decia «**19 botones de volver con respuesta al toque.** Hoy son `Pressable` secos: no
cambian nada al pulsarse». **Es falso.** Los 20 navegan:

```
router.back()      17 ficheros
ir.atras()         billing-status, ecomerse-checkout, food-checkout
(dirty ? confirmLeave() : router.back())    food-owner
```

Lo que **si** falta es la **senal visual**: ninguno de los 20 usa `Tactil` y ninguno tiene `pressed`.
Tocar y no ver nada durante 200 ms se lee como «no ha cogido el toque». Esa es la mejora real, y ya
esta corregida en el docblock.

### Cuarta cosa: cuatro titulos van al borde

| Caracteres | Ficheros |
|---:|---|
| 31 · variable | `alquiler` · `food-menu` — **no caben** → `lineasTitulo={2}` |
| **22** | `billing-status`, `intercity-planes` — en el borde |
| **21** | `alquiler-planes`, `landlord-profile` — a un caracter |

El modelo da 22,9 caracteres. **Honestidad sobre ese numero:** los 7,9 dp/caracter estan medidos en el
telefono, pero las 22,9 son esa constante **escalada de 11 a 17**, no una medicion a 17. Los cuatro de
21-22 caben **por poco**. **Esa comprobacion la haces tu**: estos cuatro titulos son donde mirar.

### Orden de las tandas

1. **Tanda 1** — `alquiler` (3) + `billing` (2) + `intercity` (1). El cambio visual mas grande.
2. **Tanda 2** — `work` (3). Los otros tres del peso 700 -> 800.
3. **Tanda 3** — `ecomerse` (5).
4. **Tanda 4** — `food` (5).
5. **Tanda 5** — `landlord-profile` + cierre.

Cada tanda con las tres puertas abiertas (`tsc`, guardia, mapa de rutas). **Los pixeles los compruebas
tu**; yo garantizo que el trinquete no sube y que `tsc` no se queja.

---

## 2 · Decision A — aplicada y verificada, delta 0

En disco y comprobada: `e26: 26` · `e28: 28` · `e30: 30` entre `e24` y `e32`. **La prueba de que es una
decision de nombres y no un cambio de diseno: el trinquete no se movio ni un digito** (`espaciado 5402`
antes y despues). La escala pasa de 20 a **22 peldaños** y cubre el **98,5 %** de los huecos enteros.

Y una nota que cambia la naturaleza del trabajo: **no es una tanda de renombrado, es una tanda de
excursion.** Los 58 literales del hueco viven **en 37 ficheros distintos**, 8 en `components`.

---

## 3 · El push — no eran los permisos

Gracias por actualizar el token, pero **el push no se cuelga por permisos**: se cuelga porque **en mi
entorno no hay ninguna credencial de git**. Y `gh` **no esta instalado**.

```
$ git push
fatal: could not read Username for 'https://gitee.com': terminal prompts disabled
```

**Y un detalle que yo mismo tenia mal:** `D:\Routeplan` **NO apunta a GitHub, apunta a Gitee**
(`gitee.com/Bernardo12/eg-route-plan.git`). El token de GitHub que actualizaste **no es el que hace
falta para estos commits**. El comando exacto:

```bash
cd /d/Routeplan
git push -u origin refactor/ui-ux-reconstruction
```

Son **8 commits** sin subir (114 ficheros, ~46.000 lineas).

**Que he subido por el conector mientras tanto:** la rama `refactor/ui-ux-reconstruction` de
`loperte12/bernardo-loperte` tiene los documentos al dia. El conector **solo acepta texto en linea**, no
binarios: por eso las capturas PNG no viajan por ahi y los `.md` si.

**Y un aviso que me encontre a mi mismo:** al preparar este commit, un `git add -A` metio **50 ficheros
basura** (`.idea/`, `mobile-app/`, `mobile-app.zip`) en un commit cuyo mensaje hablaba de las
cabeceras. Lo deshice con `git reset --soft HEAD~1` (los cambios se conservan) y lo rehice con los 11
ficheros correctos. Anadi al `.gitignore` los residuos, con una salvedad: **`mobile-app/` NO se ignora
en bloque**, porque tiene **dos ficheros rastreados que si interesan** (`package.json` y el parche de
`expo-gaode-map-navigation`); ignorar la carpeta habria dejado de rastrear cambios en ellos sin avisar.

---

## 4 · El `git init` de `D:\egapp` — ya esta hecho

```
$ git -C /d/egapp log --oneline
7ecc10b Fase 2: corrige el docblock de ScreenHeader con la medicion de las 20 cabeceras
93db2d3 Punto de partida del proyecto EG Route Plan
$ git -C /d/egapp status --short
(vacio)                     # 1.129 ficheros rastreados
```

**Los comandos exactos:**

```bash
cd /d/egapp
git init -q -b main
git config user.name  "loperte12"
git config user.email "bienvenidondongnkogonnang65@gmail.com"
# .gitignore ANTES de cualquier `git add`
git add -A
# comprobar el indice antes de commitear
```

La identidad local hizo falta porque la global de la maquina era el marcador `you@example.comm` (con
dos emes) y habria firmado los commits con un correo falso.

**El `.gitignore` salvo la operacion, y se hizo en dos pasadas.** El primer `git add -A` iba a meter
**187 MB de APK de `pruebas/`** y la cache de npm. Se amplio y se volvio a comprobar:
**1.129 ficheros, 1,4 MB**. Diferencia entre las dos cifras = todo lo que el `.gitignore` salvo.

**Dos cosas con consecuencias:**

1. **Los 79 ficheros `_*` de la raiz SI entraron, a proposito.** Preferi que entraran a decidir yo que
es basura. Si quieres que salgan se puede, pero **sacarlos despues NO los borra de la historia**: hay
que reescribir el commit. Dime y lo hacemos antes de que la historia crezca.

2. **`D:\egapp` no tiene remoto que funcione.** Le puse `origin`, pero no puedo subirlo por lo del
punto 3. La historia es **local**: puedes volver a `93db2d3` de un comando, pero **no es copia de
seguridad**. Si el disco muere, muere.

---

## Lo que queda abierto, y quien lo cierra

| Punto | Quien | Estado |
|---|---|---|
| Las 20 cabeceras, tanda a tanda | Yo | Empiezo por la tanda 1 (6 ficheros) |
| Los 8 commits de `D:\Routeplan` | **Tu** (Gitee) | Comando dado arriba |
| El remoto de `D:\egapp` | **Tu** | Comando dado arriba |
| Los 4 titulos al borde (21-22 car.) | **Tu**, en el movil | Donde mirar: dicho arriba |
| ¿Sacar los 79 `_*` de la historia? | **Tu** decides | Se puede, cuesta reescribir |

*Documento de trabajo — 24/09/2026. Kai*
