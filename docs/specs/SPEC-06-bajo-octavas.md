# SPEC-06 · Pista de bajo: síntesis + octavas por paso

**Tamaño estimado:** M · **Depende de:** SPEC-02, SPEC-03, SPEC-04 · **Requisitos PRD:** RF-3, RF-5

## Objetivo
Añadir la voz de **bajo** sintetizada con `OscillatorNode` y permitir elegir entre **3 octavas por paso** en la pista de bajo.

## Alcance
**Incluye:**
- `audio.triggerBass(time, octave)`: oscilador `sawtooth`/`square` afinado según la octava.
- Mapa de 3 octavas a frecuencias: `octave ∈ {0,1,2}` → frecuencia base × `2^octave` (relación de octava 2×). P. ej. raíz 55 Hz (A1) → 55 / 110 / 220 Hz.
- UI: cada celda activa de la pista de bajo permite seleccionar su octava (3 estados; p. ej. ciclo de clic o mini-selector), reflejando `state.tracks.bass.octaves[i]`.
- `scheduler.scheduleStep` dispara `triggerBass(time, octaves[step])` cuando el paso de bajo está activo.

**NO incluye:** ADSR configurable (SPEC-07; aquí se usan defaults), persistencia (SPEC-08).

## Diseño técnico
```js
audio.triggerBass = function (time, octave) {
  const ctx = this.ensureContext();
  const freq = 55 * Math.pow(2, octave);   // 55 / 110 / 220 Hz
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  osc.type = 'sawtooth';
  osc.frequency.setValueAtTime(freq, time);
  // envolvente ADSR por defecto (SPEC-07 la parametriza)
  osc.connect(gain).connect(this.master);
  osc.start(time); osc.stop(time + 0.5);
};
```
- La octava por paso se guarda en `state.tracks.bass.octaves[i]` (entero `0|1|2`).
- La UI de octava se integra en la fila de bajo de SPEC-02 sin romper su render.

## Criterios de aceptación
- [ ] Un patrón de bajo es audible y distinguible del resto de pistas.
- [ ] Cambiar la octava de un paso cambia audiblemente su altura (relación 2×).
- [ ] La octava elegida se refleja en `state` y en la UI.

## Checklist de code review
- [ ] Frecuencias por octava correctas (cada octava duplica la frecuencia).
- [ ] La UI de octava es clara y por paso; no afecta a las otras pistas.
- [ ] `triggerBass` sigue el mismo patrón de voz que SPEC-03 (rampas, conexión al master, sin fugas).
