# SPEC-05 · Cabezal de reproducción visual (±20 ms)

**Tamaño estimado:** S · **Depende de:** SPEC-04 · **Requisitos PRD:** RF-6, RNF-1, O3

## Objetivo
Mostrar un cabezal visual que resalta la columna del paso en curso, **sincronizado con el audio dentro de ±20 ms**.

## Alcance
**Incluye:**
- Cola de notas agendadas: el scheduler hace `playhead.notesInQueue.push({ step, time })` por cada paso programado.
- `playhead.run()`: bucle `requestAnimationFrame` que, mientras la nota al frente de la cola tenga `time <= ctx.currentTime`, la descarta y fija `playhead.currentDrawStep`.
- Resaltado de la columna `currentDrawStep` (clase CSS) y limpieza del resaltado al detener.
- Cancelar el `requestAnimationFrame` en `stop`.

**NO incluye:** cambios en el scheduler salvo el `push` a la cola (mínimo acople con SPEC-04).

## Diseño técnico
> Clave: el scheduler va **adelantado** (look-ahead ~100 ms). Por eso el cabezal **no** debe usar `scheduler.currentStep` directamente, sino la **cola time-stamped**, comparando contra `ctx.currentTime`. Así el resalte coincide con lo que realmente se oye.

```js
playhead.run = function () {
  const ctx = audio.ctx;
  while (this.notesInQueue.length && this.notesInQueue[0].time <= ctx.currentTime) {
    this.currentDrawStep = this.notesInQueue.shift().step;
    ui.highlightColumn(this.currentDrawStep);
  }
  this.raf = requestAnimationFrame(() => this.run());
};
```

## Criterios de aceptación
- [ ] El resalte de columna coincide con el golpe audible dentro de ±20 ms (verificación percibida; opcional medir).
- [ ] Al detener, se limpia el resalte y se cancela el `rAF` (sin bucles huérfanos).
- [ ] Tras cambiar de pestaña y volver, el cabezal se resincroniza (no queda desfasado permanentemente).

## Checklist de code review
- [ ] Usa la cola time-stamped, **no** `scheduler.currentStep` (que está adelantado por el look-ahead).
- [ ] `requestAnimationFrame` se cancela en `stop`.
- [ ] El resaltado es puramente visual; no toca `state` ni el audio.
