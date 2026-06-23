# SPEC-02 · Cuadrícula del secuenciador (UI 4×8)

**Tamaño estimado:** S · **Depende de:** SPEC-01 · **Requisitos PRD:** RF-1

## Objetivo
Renderizar la cuadrícula de **4 pistas × 8 pasos** a partir de `state` y permitir activar/desactivar cada celda con clic. Sin audio todavía.

## Alcance
**Incluye:**
- `ui.renderGrid()`: construye 4 filas (kick, snare, hihat, bass) × 8 celdas leyendo `state.tracks[t].steps`.
- Etiqueta de cada pista y numeración de pasos (1–8).
- Toggle: clic en celda invierte `state.tracks[t].steps[i]` y actualiza la clase visual (`active`).
- Indicación visual de "tiempos fuertes" cada 4 pasos (opcional, estético).

**NO incluye:** selector de octava del bajo (SPEC-06), sonido (SPEC-03), resaltado del cabezal (SPEC-05).

## Diseño técnico
- Render declarativo desde `state`: la celda refleja `steps[i]`, no se guarda estado en el DOM.
- Usar `data-track` y `data-step` en cada celda + delegación de eventos en el contenedor.
- `ui.setCell(track, step)` actualiza solo la clase de esa celda tras el toggle.

```html
<div class="grid">
  <div class="row" data-track="kick"> <span class="label">Kick</span>
    <button class="cell" data-track="kick" data-step="0" aria-pressed="false"></button>
    ...
  </div>
</div>
```

## Criterios de aceptación
- [ ] Se ven 4 pistas × 8 celdas = 32 celdas.
- [ ] Clic alterna el estado visual y `state.tracks[t].steps[i]` queda sincronizado.
- [ ] Re-render desde un `state` arbitrario pinta exactamente las celdas activas (preparado para SPEC-08).

## Checklist de code review
- [ ] La UI deriva de `state` (single source of truth); el DOM no es la fuente de verdad.
- [ ] Eventos por delegación o listeners correctamente acotados (sin duplicados).
- [ ] Marcado accesible: `<button>` + `aria-pressed` (deseable).
- [ ] No hay lógica de audio filtrada en este spec.
