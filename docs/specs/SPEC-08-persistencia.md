# SPEC-08 · Persistencia: Guardar / Cargar (localStorage, un slot)

**Tamaño estimado:** S · **Depende de:** todos (necesita el `state` completo) · **Requisitos PRD:** RF-7, RNF-6, O4

## Objetivo
Guardar y cargar el patrón completo en `localStorage` usando **un único slot**, de forma robusta ante datos ausentes o corruptos.

## Alcance
**Incluye:**
- `storage.save()`: serializa `state` (steps, `bass.octaves`, `bpm`, `adsr` de cada pista) con `JSON.stringify` a la clave `platzi.drummachine.pattern`.
- `storage.load()`: lee y parsea; valida `version` y forma; hace merge con los valores por defecto antes de aplicar.
- Botones **Guardar** y **Cargar** en la UI.
- Tras cargar: re-render completo (grilla, octavas del bajo, sliders ADSR, valor de BPM).
- Manejo de errores: `try/catch`; slot vacío o JSON corrupto → arranca/permanece con el patrón por defecto sin romper la app.

**NO incluye:** múltiples patrones nombrados (explícitamente fuera de alcance del PRD v1.0).

## Diseño técnico
```js
const KEY = 'platzi.drummachine.pattern';
storage.save = function () {
  try { localStorage.setItem(KEY, JSON.stringify(state)); }
  catch (e) { ui.notify('No se pudo guardar'); }
};
storage.load = function () {
  try {
    const raw = localStorage.getItem(KEY);
    if (!raw) return false;
    const data = JSON.parse(raw);
    if (!data || data.version !== state.version) return false; // validación defensiva
    Object.assign(state, mergeWithDefaults(data));
    ui.renderAll();
    return true;
  } catch (e) { return false; }   // corrupto → se ignora, se queda el default
};
```

## Criterios de aceptación
- [ ] **Escenario del reto:** crear patrón → Guardar → F5 (recargar) → Cargar → el patrón se restaura **idéntico** (incluye octavas del bajo, BPM y ADSR).
- [ ] Con `localStorage` vacío, la app arranca con el patrón por defecto.
- [ ] Con `localStorage` corrupto/manipulado, Cargar no rompe la app (se mantiene el default).

## Checklist de code review
- [ ] Se persiste un esquema versionado (`version`) para futura compatibilidad.
- [ ] Validación defensiva: no se confía ciegamente en el JSON leído.
- [ ] Tras `load`, se re-renderiza **todo** el estado visible (no quedan controles desincronizados).
- [ ] Un único slot/clave; sin lógica de múltiples patrones (fuera de alcance).
