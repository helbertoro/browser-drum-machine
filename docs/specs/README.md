# Specs — Caja de Ritmos en el Navegador

Estos specs descomponen el [PRD](../prd/PRD.md) en **unidades pequeñas, implementables y revisables por separado**. La intención: **1 spec → 1 PR → 1 code review**.

Cada spec es independiente en lo posible, pero respeta un orden de dependencias (abajo). Todos comparten el mismo modelo de estado y el mismo mapa de módulos definidos aquí, para que cada spec individual quede corto.

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

| Spec | Título | Depende de |
|------|--------|------------|
| [SPEC-01](SPEC-01-esqueleto-audiocontext.md) | Esqueleto de la app y ciclo de vida del AudioContext | — |
| [SPEC-02](SPEC-02-cuadricula-secuenciador.md) | Cuadrícula del secuenciador (UI 4×8) | 01 |
| [SPEC-03](SPEC-03-sintesis-percusion.md) | Síntesis de percusión (kick, snare, hi-hat) | 01 |
| [SPEC-04](SPEC-04-scheduler-transporte.md) | Scheduler look-ahead + transporte (Play/Stop/Tempo/Clear) | 01,02,03 |
| [SPEC-05](SPEC-05-cabezal-visual.md) | Cabezal de reproducción visual (±20 ms) | 04 |
| [SPEC-06](SPEC-06-bajo-octavas.md) | Pista de bajo: síntesis + octavas por paso | 02,03,04 |
| [SPEC-07](SPEC-07-adsr.md) | Controles ADSR por pista | 03,06 |
| [SPEC-08](SPEC-08-persistencia.md) | Persistencia: Guardar/Cargar (localStorage, un slot) | todos |

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
