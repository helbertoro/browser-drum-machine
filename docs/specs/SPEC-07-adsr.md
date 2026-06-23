# SPEC-07 · Controles ADSR por pista

**Tamaño estimado:** M · **Depende de:** SPEC-03, SPEC-06 · **Requisitos PRD:** RF-4

## Objetivo
Dar a cada una de las 4 pistas controles **ADSR** (Ataque, Decaimiento, Sostenido, Liberación) que modelen la envolvente de amplitud de sus golpes.

## Alcance
**Incluye:**
- UI: 4 sliders (A, D, S, R) por pista, ligados a `state.tracks[t].adsr`, con valores visibles.
- Helper único `audio.applyADSR(gainParam, time, adsr, sustainDur)` que programa la envolvente sobre un `AudioParam` de ganancia.
- Las voces de SPEC-03 y SPEC-06 dejan de usar envolventes hardcoded y leen `state.tracks[t].adsr` al disparar.
- Rangos: A/D/R en segundos (0–~2 s); S nivel 0–1.

**NO incluye:** persistencia (SPEC-08; aquí solo el estado en memoria reacciona en vivo).

## Diseño técnico
```js
audio.applyADSR = function (param, t0, { a, d, s, r }, sustainDur) {
  const peak = 1.0;
  param.setValueAtTime(0.0001, t0);
  param.linearRampToValueAtTime(peak, t0 + a);            // Attack
  param.linearRampToValueAtTime(Math.max(s, 0.0001), t0 + a + d); // Decay → Sustain
  const releaseStart = t0 + a + d + sustainDur;
  param.setTargetAtTime(0.0001, releaseStart, r / 3);     // Release
};
```
- Cada `trigger*` pasa su `gain.gain` y el `adsr` de su pista a este helper.
- Validar/clampar entradas: A/D/R ≥ 0; S ∈ [0, 1].

## Criterios de aceptación
- [ ] Mover los sliders ADSR de una pista cambia audiblemente su sonido (p. ej. un release largo en el bajo lo alarga).
- [ ] Cada pista mantiene su ADSR independiente.
- [ ] A/D/R se interpretan en segundos y S como nivel 0–1.

## Checklist de code review
- [ ] Existe **un solo** helper `applyADSR` reutilizado por todas las voces (sin duplicar la lógica de envolvente).
- [ ] Entradas validadas/clampadas (sin valores negativos; S acotado).
- [ ] El release no provoca clicks (decae a un valor pequeño, no a 0 absoluto con corte).
- [ ] Los valores ADSR viven en `state` (no en el DOM), listos para SPEC-08.
