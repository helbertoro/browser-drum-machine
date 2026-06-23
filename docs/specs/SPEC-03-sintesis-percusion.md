# SPEC-03 · Síntesis de percusión (kick, snare, hi-hat)

**Tamaño estimado:** M · **Depende de:** SPEC-01 · **Requisitos PRD:** RF-5, O2

## Objetivo
Implementar las voces sintetizadas de **bombo, caja y hi-hat** con `OscillatorNode` y ruido generado en tiempo de ejecución. Exponer una API `trigger*(time)` que reproduce un golpe en un instante absoluto del reloj de audio.

## Alcance
**Incluye:**
- `audio.triggerKick(time)`: oscilador `sine` con barrido de frecuencia descendente (~150 Hz → ~50 Hz) + envolvente de amplitud.
- `audio.triggerSnare(time)`: ruido filtrado (paso-banda/altos) + componente tonal de oscilador.
- `audio.triggerHihat(time)`: ruido con filtro paso-altos y envolvente muy corta (o varios `square` filtrados, estilo 808).
- `audio.makeNoiseBuffer()`: genera un `AudioBuffer` con `Math.random()` (ruido blanco) — **en runtime, sin archivo**.
- Envolventes con valores ADSR **por defecto hardcoded** (SPEC-07 los hará configurables).
- Hook opcional de audición para review: clic en la etiqueta de la pista dispara la voz una vez con `audio.ctx.currentTime`.

**NO incluye:** bajo (SPEC-06), scheduler (SPEC-04), ADSR configurable por usuario (SPEC-07).

## Diseño técnico
- Cada `trigger` crea sus nodos, los conecta a `audio.master`, programa rampas con `time` y llama `start(time)`/`stop(time + dur)`.
- El ruido **no** es un sample: se construye un buffer corto con datos aleatorios y se reproduce con `AudioBufferSourceNode`. Cumple "sin samples" porque no se carga ningún archivo.
- Frecuencias, duraciones y tipos de onda nombrados como constantes, no números mágicos.

```js
audio.triggerKick = function (time) {
  const ctx = this.ensureContext();
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  osc.type = 'sine';
  osc.frequency.setValueAtTime(150, time);
  osc.frequency.exponentialRampToValueAtTime(50, time + 0.12);
  gain.gain.setValueAtTime(1, time);
  gain.gain.exponentialRampToValueAtTime(0.001, time + 0.2);
  osc.connect(gain).connect(this.master);
  osc.start(time); osc.stop(time + 0.2);
};
```

## Criterios de aceptación
- [ ] Cada voz produce el sonido percusivo esperado, sin clicks/pops audibles.
- [ ] El ruido se genera en runtime (no hay archivos `.wav`/`.mp3` ni `<audio>`).
- [ ] Disparar varias voces en el mismo `time` no genera distorsión por suma excesiva (gain razonable).

## Checklist de code review
- [ ] Los nodos efímeros se descartan tras `stop` (sin acumulación/fugas).
- [ ] Envolventes con rampas de `AudioParam` (no asignaciones abruptas de `.value`).
- [ ] Constantes nombradas para frecuencias/tiempos.
- [ ] Todas las voces se conectan al master gain de SPEC-01.
