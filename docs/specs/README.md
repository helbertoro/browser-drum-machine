# Specs — Caja de Ritmos en el Navegador

Estos specs descomponen el [PRD](../prd/PRD.md) en **unidades pequeñas, implementables y revisables por separado**. La intención: **1 spec → 1 PR → 1 code review**.

Cada spec es independiente en lo posible, pero respeta un orden de dependencias (abajo). Todos comparten el mismo modelo de estado y el mismo mapa de módulos definidos aquí, para que cada spec individual quede corto.

---

## 📌 Estado de desarrollo

> **Este README es el control de avance del proyecto.** Antes de empezar, lee *Estado actual* y la última entrada de la *Bitácora*. Al terminar tu trabajo, **actualiza el tablero y añade una entrada en la bitácora** para que cualquier otra persona pueda continuar desde donde lo dejaste.

**Estado actual:** 🟢 SPEC-01 hecho. **Siguiente spec a tomar: `SPEC-02` o `SPEC-03`** (paralelos, ambos dependen solo de 01).

### Tablero de progreso
| Spec | Estado | Depende de | Responsable | PR | Últ. actualización | Notas |
|------|--------|------------|-------------|----|--------------------|-------|
| [SPEC-01](SPEC-01-esqueleto-audiocontext.md) · Esqueleto + AudioContext | ✅ Hecho | — | Helber Toro | — | 2026-06-23 | `index.html` creado |
| [SPEC-02](SPEC-02-cuadricula-secuenciador.md) · Cuadrícula UI 4×8 | ⬜ Pendiente | 01 ✅ | — | — | 2026-06-23 | Listo para tomar |
| [SPEC-03](SPEC-03-sintesis-percusion.md) · Síntesis percusión | ⬜ Pendiente | 01 ✅ | — | — | 2026-06-23 | Listo para tomar (paralelo con 02) |
| [SPEC-04](SPEC-04-scheduler-transporte.md) · Scheduler + transporte | ⬜ Pendiente | 01,02,03 | — | — | 2026-06-23 | Bloqueado hasta que 02 y 03 estén listos |
| [SPEC-05](SPEC-05-cabezal-visual.md) · Cabezal visual ±20 ms | ⬜ Pendiente | 04 | — | — | 2026-06-23 | — |
| [SPEC-06](SPEC-06-bajo-octavas.md) · Bajo + octavas | ⬜ Pendiente | 02,03,04 | — | — | 2026-06-23 | — |
| [SPEC-07](SPEC-07-adsr.md) · ADSR por pista | ⬜ Pendiente | 03,06 | — | — | 2026-06-23 | — |
| [SPEC-08](SPEC-08-persistencia.md) · Persistencia localStorage | ⬜ Pendiente | todos | — | — | 2026-06-23 | Último; cierra el reto |

**Leyenda de estados:** ⬜ Pendiente · 🟡 En progreso · 🔵 En revisión (PR abierto) · ✅ Hecho (merge en `main`) · ⛔ Bloqueado

### Cómo continuar (handoff)
1. Lee **Estado actual** y la última entrada de la **Bitácora**.
2. En el **Tablero**, toma el primer spec ⬜ *Pendiente* cuyas dependencias estén ✅ *Hecho*.
3. Márcalo 🟡 *En progreso*: pon tu nombre en *Responsable* y la fecha en *Últ. actualización*.
4. Implementa **solo** el alcance del spec; abre PR y cámbialo a 🔵 *En revisión* con el link del PR.
5. Tras el merge, márcalo ✅ *Hecho* y **añade una entrada en la bitácora** con: qué quedó, cualquier desvío del spec y cuál es el siguiente paso natural.

### Bitácora de avance
> Entrada más reciente arriba. Formato: `fecha — autor: qué se hizo / dónde continuar`.

- **2026-06-23 — Helber Toro:** `SPEC-01` implementado. Creado `index.html` con los 6 namespaces (`state`, `audio`, `scheduler`, `playhead`, `ui`, `storage`), el `state` con el modelo completo (4 pistas, ADSR defaults, octavas del bajo en 1), `audio.ensureContext()` lazy, contenedores HTML (`#header`, `#transport`, `#grid`, `#controls`, `#persistence`) y CSS base con tokens de color. **Próximo paso:** SPEC-02 (cuadrícula) y SPEC-03 (síntesis de percusión) se pueden tomar en paralelo — ambos solo dependen de 01.
- **2026-06-23 — Helber Toro:** Creado el [PRD](../prd/PRD.md) y los 8 specs. Aún no hay código.

---

## Convenciones compartidas

### Entregable
Un **único archivo** `index.html` (HTML + CSS + JS embebidos). Sin frameworks, sin bundler, sin dependencias externas. Sin `<audio>` ni archivos de samples. Todo el audio se sintetiza con la Web Audio API.

### Mapa de módulos (objetos dentro del único `<script>`)
| Objeto | Responsabilidad |
|--------|-----------------|
| `state` | Fuente única de verdad del patrón (en memoria) |
| `audio` | `AudioContext`, master gain y voces sintetizadas (`trigger*`) |
| `scheduler` | Agendado look-ahead anclado a `audio.ctx.currentTime` |
| `playhead` | Bucle `requestAnimationFrame` que resalta el paso en curso |
| `ui` | Render de la cuadrícula y los controles desde `state` |
| `storage` | (de)serialización a `localStorage` |

### Modelo de estado compartido (`state`)
```js
state = {
  version: 1,
  bpm: 120,
  isPlaying: false,
  currentStep: 0,
  tracks: {
    kick:  { steps: [/* 8 bool */], adsr: { a, d, s, r } },
    snare: { steps: [/* 8 bool */], adsr: { a, d, s, r } },
    hihat: { steps: [/* 8 bool */], adsr: { a, d, s, r } },
    bass:  { steps: [/* 8 bool */], octaves: [/* 8 ints 0|1|2 */], adsr: { a, d, s, r } },
  }
}
```
- `adsr`: `a`, `d`, `r` en **segundos**; `s` como **nivel 0–1**.
- `octaves` (solo `bass`): entero `0|1|2` por paso; relevante solo si el paso está activo.

### Definición de "paso"
1 paso = 1 corchea. `secondsPerStep = (60 / bpm) / 4`.

---

## Orden de dependencias / build order

```
SPEC-01 (esqueleto + AudioContext)
   ├─ SPEC-02 (cuadrícula UI)
   ├─ SPEC-03 (síntesis percusión)
   │     └─ SPEC-04 (scheduler + transporte)  ← requiere 02 y 03
   │            └─ SPEC-05 (cabezal visual)
   ├─ SPEC-06 (bajo + octavas)                ← requiere 02, 03, 04
   ├─ SPEC-07 (ADSR por pista)                ← requiere 03, 06
   └─ SPEC-08 (persistencia localStorage)     ← requiere todo el state
```

El detalle de cada spec (título, estado, dependencias, responsable) vive en el [Tablero de progreso](#tablero-de-progreso).

---

## Cómo revisar (proceso de equipo)
1. Toma un spec libre cuyas dependencias ya estén en `main`.
2. Implementa **solo** su alcance; deja lo marcado "NO incluye" para su spec.
3. Verifica los **Criterios de aceptación**.
4. Abre PR enlazando el spec; el revisor usa la **Checklist de code review** del spec.

## Reglas transversales (aplican a todos los PRs)
- Cero dependencias externas, cero `<audio>`, cero archivos de sample.
- `state` es la única fuente de verdad; la UI deriva de `state` (no guardar estado en el DOM).
- El audio nunca se temporiza con `setTimeout`/`Date.now()`; solo con tiempos de `audio.ctx.currentTime`.
- Las envolventes usan rampas de `AudioParam` (no cortes abruptos → evita clicks/pops).
- Los nodos de audio efímeros deben quedar desconectados tras sonar (sin fugas).
