# SPEC-01 · Esqueleto de la app y ciclo de vida del AudioContext

**Tamaño estimado:** S · **Depende de:** — · **Requisitos PRD:** RNF-3, RNF-4

## Objetivo
Crear el archivo único `index.html` con la estructura base (HTML/CSS/JS embebidos), el objeto `state` inicial y el `audio` engine que crea y **reanuda** el `AudioContext` en el primer gesto del usuario.

## Alcance
**Incluye:**
- `index.html` con un único `<script>` que declara los namespaces vacíos: `state`, `audio`, `scheduler`, `playhead`, `ui`, `storage`.
- `state` inicializado con la forma del [modelo compartido](README.md#modelo-de-estado-compartido-state) y un patrón por defecto (puede ser todo vacío salvo BPM=120).
- `audio.ctx` + `audio.master` (un `GainNode` maestro conectado a `destination`).
- `audio.ensureContext()`: crea el `AudioContext` de forma perezosa la primera vez y llama a `ctx.resume()` (cumple la política de autoplay).
- Estructura/contenedores HTML vacíos: header, barra de transporte, contenedor de grilla, contenedor de controles.
- CSS base mínimo (layout, tipografía).

**NO incluye:** síntesis de sonido (SPEC-03), grilla funcional (SPEC-02), scheduling (SPEC-04).

## Diseño técnico
```js
const audio = {
  ctx: null,
  master: null,
  ensureContext() {
    if (!this.ctx) {
      this.ctx = new (window.AudioContext || window.webkitAudioContext)();
      this.master = this.ctx.createGain();
      this.master.gain.value = 0.8;
      this.master.connect(this.ctx.destination);
    }
    if (this.ctx.state === 'suspended') this.ctx.resume();
    return this.ctx;
  }
};
```
Llamar a `audio.ensureContext()` desde el primer gesto del usuario (lo usará SPEC-04 al pulsar Play).

## Criterios de aceptación
- [ ] Abrir `index.html` (doble clic) no muestra errores en consola.
- [ ] El `AudioContext` **no** se crea al cargar la página, solo tras `ensureContext()`.
- [ ] Tras invocar `ensureContext()` por un gesto, `audio.ctx.state === 'running'`.
- [ ] `state` existe y cumple la forma del modelo compartido.

## Checklist de code review
- [ ] El `AudioContext` se crea de forma perezosa (no en el load), con fallback `webkitAudioContext`.
- [ ] Existe un único master gain; las voces futuras se conectarán a él.
- [ ] Sin dependencias externas, sin `<audio>`, sin samples.
- [ ] `state` es un objeto plano serializable (sin nodos del DOM ni de audio dentro).
