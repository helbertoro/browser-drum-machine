# SPEC-04 · Scheduler look-ahead + transporte (Play/Stop/Tempo/Clear)

**Tamaño estimado:** M · **Depende de:** SPEC-01, SPEC-02, SPEC-03 · **Requisitos PRD:** RF-2, RNF-1, RNF-2

## Objetivo
Implementar el **scheduler de precisión** (patrón look-ahead anclado a `audio.ctx.currentTime`) y los controles de transporte: Reproducir, Detener, Tempo (60–180 BPM) y Limpiar.

## Alcance
**Incluye:**
- `scheduler.start()` / `scheduler.stop()`.
- Bucle look-ahead con `setInterval` (~25 ms) y ventana de anticipación (~100 ms).
- `nextNoteTime` (en segundos de `ctx.currentTime`) y `currentStep` (0–7, con wrap).
- `scheduler.scheduleStep(step, time)`: por cada pista con `steps[step] === true`, llama a su `trigger*(time)` correspondiente.
- `secondsPerStep = (60 / state.bpm) / 4`; cambio de BPM en vivo (recalcula el siguiente paso, sin reiniciar el bucle).
- Botones: **Play** (llama `audio.ensureContext()` + `scheduler.start()`), **Stop** (`scheduler.stop()`, resetea `currentStep=0`), **Clear** (vacía `steps` y `bass.octaves`, re-render).
- Control de Tempo (slider/input) con valor numérico visible.

**NO incluye:** disparo del bajo (lo añade SPEC-06 dentro de `scheduleStep`), resaltado visual (SPEC-05).

## Diseño técnico
Patrón "A Tale of Two Clocks":
```js
scheduler.tick = function () {
  const ctx = audio.ctx;
  while (this.nextNoteTime < ctx.currentTime + this.lookAhead) {   // lookAhead ≈ 0.1s
    this.scheduleStep(this.currentStep, this.nextNoteTime);
    this.nextNoteTime += (60 / state.bpm) / 4;
    this.currentStep = (this.currentStep + 1) % 8;
  }
};
scheduler.start = function () {
  this.nextNoteTime = audio.ctx.currentTime + 0.05;
  this.timer = setInterval(() => this.tick(), 25);
};
scheduler.stop = function () { clearInterval(this.timer); this.currentStep = 0; };
```

## Criterios de aceptación
- [ ] Programar 4 pasos de kick (1,3,5,7) y pulsar Play → suena un *four-on-the-floor* estable.
- [ ] El tempo entre 60–180 BPM cambia la velocidad **sin** reiniciar el patrón.
- [ ] Stop detiene y deja `currentStep` en 0; Clear deja el patrón vacío.
- [ ] Sin deriva audible tras varios minutos en bucle.

## Checklist de code review
- [ ] El timing del audio se ancla a `ctx.currentTime`; `setInterval` solo agenda, no temporiza el sonido.
- [ ] `scheduleStep` es agnóstico a la pista (itera el `state`), facilitando que SPEC-06 sume el bajo.
- [ ] El cambio de BPM no acumula error (recalcula desde `nextNoteTime`).
- [ ] `stop` limpia el `setInterval` (sin timers huérfanos).
