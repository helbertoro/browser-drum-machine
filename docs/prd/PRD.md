# PRD — Caja de Ritmos en el Navegador

> **Documento de Requisitos de Producto (PRD)**
> Proyecto: Secuenciador / caja de ritmos web para cursos de producción musical de Platzi
> Versión: 1.0 · Fecha: 2026-06-23 · Estado: Aprobado para desarrollo

---

## 1. Resumen ejecutivo

Construir un **secuenciador de patrones de una sola página**, 100 % en el navegador, usando exclusivamente la **Web Audio API** y **JavaScript vanilla**. Todo el sonido se genera en tiempo real con `OscillatorNode` (y ruido sintetizado en código) — **sin etiquetas `<audio>` y sin archivos de muestras (samples)**.

El resultado es una herramienta práctica e interactiva que acompaña a los cursos de producción musical: los estudiantes experimentan con ritmos y líneas de bajo sin instalar una DAW.

---

## 2. Contexto y problema

Los cursos de producción musical de Platzi carecen de herramientas prácticas interactivas dentro de la clase. Hoy un estudiante que quiere experimentar con un ritmo debe instalar una DAW (estación de trabajo de audio digital), lo cual es una barrera de entrada significativa.

Una caja de ritmos en el navegador, junto con un sintetizador de bajo, elimina esa fricción y queda como **complemento permanente** de la clase: abre una pestaña, programa y escucha.

---

## 3. Objetivos

| # | Objetivo | Métrica de éxito |
|---|----------|------------------|
| O1 | Permitir programar ritmos sin instalar software | El estudiante produce un patrón audible en < 1 min desde que abre la página |
| O2 | Generar todo el audio por síntesis propia | 0 archivos de sample y 0 etiquetas `<audio>` en el proyecto |
| O3 | Sincronía audio–visual precisa | El cabezal visual coincide con el audio dentro de **±20 ms** |
| O4 | Persistencia local | Un patrón guardado se recupera intacto tras recargar la página |

### No objetivos (fuera de alcance v1.0)
- Múltiples patrones nombrados / gestión de proyectos (solo **un slot** de guardado).
- Exportar audio (WAV/MP3) o MIDI.
- Swing/shuffle, automatización, efectos (reverb, delay), más de 8 pasos.
- Backend, cuentas de usuario o sincronización en la nube.
- Soporte móvil/táctil dedicado (objetivo: escritorio con navegador moderno).

---

## 4. Usuario objetivo

**Persona:** Estudiante de un curso de producción musical de Platzi, principiante o intermedio, sin DAW instalada, usando un laptop con un navegador moderno (Chrome/Firefox/Edge/Safari recientes).

**Necesidad principal:** Entender de forma tangible qué es un secuenciador por pasos, un patrón rítmico (p. ej. *four-on-the-floor*), una envolvente ADSR y cómo se relaciona la altura (octava) en una línea de bajo.

---

## 5. Stack y restricciones técnicas

- **Audio:** Web Audio API únicamente. Síntesis con `OscillatorNode` programados manualmente.
- **Lenguaje:** JavaScript vanilla (sin frameworks, sin librerías de audio, sin bundler).
- **Entrega:** **Un único archivo HTML** que contiene HTML, CSS y JS embebidos. Portable: se abre con doble clic.
- **Prohibido:** etiquetas `<audio>`, archivos de samples (`.wav`, `.mp3`, etc.), librerías externas de audio (Tone.js, Howler, etc.).
- **Permitido:** ruido (white noise) generado en tiempo de ejecución llenando un `AudioBuffer` con valores aleatorios. Esto **no** es un sample — no se carga ningún archivo — y es la técnica estándar para sintetizar caja y hi-hat.

---

## 6. Requisitos funcionales

### RF-1 · Secuenciador de 8 pasos y 4 pistas
- Cuadrícula de **4 pistas × 8 pasos** = 32 celdas.
- Pistas: **Bombo (kick)**, **Caja (snare)**, **Hi-hat**, **Bajo (bass)**.
- Cada celda se activa/desactiva con clic (toggle on/off).
- Estado visual claro de celda activa vs. inactiva.

### RF-2 · Transporte y tempo
- **Tempo:** control de 60 a 180 BPM (slider o input numérico) con valor visible.
- **Reproducir (Play):** inicia la reproducción en bucle del patrón.
- **Detener (Stop):** detiene la reproducción y resetea el cabezal al paso 1.
- **Limpiar (Clear):** vacía todas las celdas activas (deja el patrón en blanco).
- Cambiar el BPM durante la reproducción ajusta el tempo sin reiniciar el bucle.

### RF-3 · Pista de bajo con octavas por paso
- Cada paso activo de la pista de **bajo** tiene **3 octavas seleccionables** (p. ej. Baja / Media / Alta).
- La selección de octava es por paso (cada paso puede sonar en distinta octava).
- El bajo se sintetiza con `OscillatorNode` afinado a la nota/octava elegida.

### RF-4 · Controles ADSR por pista
- Cada una de las 4 pistas tiene sus propios controles **A**taque, **D**ecaimiento, **S**ostenido, **L**iberación (release).
- Los valores ADSR modelan la envolvente de amplitud (vía `GainNode`) de cada golpe de esa pista.
- Rango sugerido: A/D/R en segundos (0–~2 s), S en nivel (0–1).

### RF-5 · Síntesis de sonido (solo Web Audio)
- **Bombo (kick):** oscilador `sine` con barrido descendente de frecuencia (~150 Hz → ~50 Hz) y envolvente de amplitud.
- **Caja (snare):** mezcla de ruido sintetizado (filtrado) + componente tonal de oscilador.
- **Hi-hat:** ruido sintetizado pasado por filtro paso-altos con envolvente muy corta (o conjunto de osciladores `square` filtrados, estilo 808).
- **Bajo (bass):** oscilador (`sawtooth`/`square`) afinado por octava, con envolvente ADSR.
- Toda envolvente usa la rampa del audio (`setValueAtTime`, `linearRampToValueAtTime`, `setTargetAtTime`) — nunca `setTimeout` para la amplitud.

### RF-6 · Cabezal de reproducción visual
- Indicador visual (resaltado de columna) que avanza paso a paso, sincronizado con el audio.
- **Precisión requerida: ±20 ms** respecto al evento de audio.
- Implementado con bucle de `requestAnimationFrame` que lee el paso actual programado, **no** con `setInterval` ingenuo sobre el reloj de JS.

### RF-7 · Guardar y cargar (localStorage, un solo slot)
- Botón **Guardar:** serializa el estado completo del patrón a `localStorage` en un único slot.
- Botón **Cargar:** restaura el patrón desde `localStorage`.
- El estado persistido incluye: celdas activas por pista, octava por paso del bajo, BPM y parámetros ADSR de cada pista.
- Tras recargar la página (F5), **Cargar** reconstruye exactamente el patrón guardado.

---

## 7. Requisitos no funcionales

| ID | Requisito |
|----|-----------|
| RNF-1 | **Precisión temporal:** desviación cabezal-audio ≤ ±20 ms. |
| RNF-2 | **Latencia de inicio:** Play comienza a sonar en < 100 ms tras el clic. |
| RNF-3 | **Sin dependencias externas:** funciona offline, sin red, abriendo el archivo. |
| RNF-4 | **Política de autoplay:** el `AudioContext` se crea/reanuda (`resume()`) en el primer gesto del usuario (clic en Play), cumpliendo las políticas del navegador. |
| RNF-5 | **Compatibilidad:** últimas versiones de Chrome, Firefox, Edge y Safari de escritorio. |
| RNF-6 | **Robustez de persistencia:** si `localStorage` está vacío o corrupto, la app arranca con un patrón por defecto sin romperse. |

---

## 8. Arquitectura técnica

### 8.1 Componentes lógicos (dentro del único archivo HTML)
1. **AudioEngine** — administra el `AudioContext`, la síntesis por pista y las envolventes ADSR.
2. **Scheduler (look-ahead)** — agenda los eventos de audio por adelantado para precisión de muestra.
3. **State** — modelo en memoria del patrón (pistas, pasos, octavas, BPM, ADSR).
4. **UI / Render** — construye la cuadrícula, los controles y refleja el estado.
5. **Playhead** — bucle `requestAnimationFrame` que ilumina el paso en curso.
6. **Storage** — (de)serialización a `localStorage`.

### 8.2 Scheduler de precisión (clave para RNF-1)
Patrón recomendado (*"A Tale of Two Clocks"*):
- Un temporizador "lento" (`setInterval` ~25 ms) actúa como *look-ahead*.
- En cada tick, mientras el siguiente paso caiga dentro de una ventana de anticipación (~100 ms), se agenda el audio con tiempos absolutos de `audioContext.currentTime` (mediante `start(time)` / rampas programadas).
- El reloj de muestra del audio (`currentTime`) es la **única** fuente de verdad temporal; el reloj de JS solo decide *cuándo agendar*, no *cuándo sonar*.
- El cabezal visual compara `audioContext.currentTime` contra los tiempos agendados para resaltar el paso correcto.

### 8.3 Cálculo de tiempo por paso
- Duración de un paso = `(60 / BPM) / 4` segundos (8 pasos = 2 compases de corcheas, o 8 corcheas según rejilla; se define 1 paso = 1 corchea).
- El cambio de BPM recalcula la duración del siguiente paso agendado.

---

## 9. Diseño de UI/UX

Una sola pantalla, sin navegación. Layout vertical:

```
┌──────────────────────────────────────────────┐
│  Caja de Ritmos — Platzi                       │
├──────────────────────────────────────────────┤
│  [▶ Play] [■ Stop] [🗑 Clear]                  │
│  Tempo: [====|====] 120 BPM                    │
│  [💾 Guardar] [📂 Cargar]                       │
├──────────────────────────────────────────────┤
│         1  2  3  4  5  6  7  8                  │
│  Kick  [ ][ ][ ][ ][ ][ ][ ][ ]   ADSR ▸       │
│  Snare [ ][ ][ ][ ][ ][ ][ ][ ]   ADSR ▸       │
│  Hihat [ ][ ][ ][ ][ ][ ][ ][ ]   ADSR ▸       │
│  Bass  [▾][ ][▾][ ][ ][▾][ ][ ]   ADSR ▸       │
│         (cada paso de Bass: selector de octava)│
└──────────────────────────────────────────────┘
```

- **Cabezal:** la columna del paso activo se resalta al reproducir.
- **ADSR por pista:** 4 controles (sliders) por pista, agrupados/colapsables para no saturar.
- **Bajo:** cada celda activa muestra su octava (Baja/Media/Alta) seleccionable.
- Estética alineada a la identidad de Platzi (acentos en verde) — opcional, no bloqueante.

---

## 10. Modelo de datos

Estado en memoria y formato de `localStorage` (un único slot, p. ej. clave `platzi.drummachine.pattern`):

```json
{
  "version": 1,
  "bpm": 120,
  "tracks": {
    "kick":  { "steps": [true,false,false,false,true,false,false,false],
               "adsr": { "a": 0.001, "d": 0.15, "s": 0.0, "r": 0.1 } },
    "snare": { "steps": [false,false,false,false,true,false,false,false],
               "adsr": { "a": 0.001, "d": 0.2, "s": 0.0, "r": 0.15 } },
    "hihat": { "steps": [true,true,true,true,true,true,true,true],
               "adsr": { "a": 0.001, "d": 0.05, "s": 0.0, "r": 0.03 } },
    "bass":  { "steps": [true,false,true,false,false,true,false,false],
               "octaves": [1,0,2,0,0,1,0,0],
               "adsr": { "a": 0.01, "d": 0.2, "s": 0.6, "r": 0.2 } }
  }
}
```

- `steps`: array de 8 booleanos (paso activo/inactivo).
- `octaves` (solo bajo): array de 8 enteros `0|1|2` (octava por paso; se usa cuando el paso está activo).
- `adsr`: `a`, `d`, `r` en segundos; `s` como nivel 0–1.

---

## 11. Criterios de aceptación (Definition of Done)

Escenario end-to-end del reto:

1. ✅ El estudiante abre la página (un archivo HTML).
2. ✅ Programa **4 pasos en el bombo** y pulsa **Play** → escucha un patrón *four-on-the-floor*.
3. ✅ Cambia a la **pista de bajo**, crea un patrón (con octavas por paso) y lo escucha.
4. ✅ **Guarda** el proyecto.
5. ✅ **Actualiza la página (F5)** y vuelve a **Cargar** el patrón → se restaura intacto.
6. ✅ El sonido proviene de `OscillatorNode` programados manualmente (verificable en el código; sin `<audio>` ni samples).

Criterios adicionales:

- [ ] Tempo ajustable 60–180 BPM y refleja el cambio en la reproducción.
- [ ] Botones Play / Stop / Clear funcionan según RF-2.
- [ ] Bajo con 3 octavas seleccionables por paso (RF-3).
- [ ] ADSR independiente y audible por cada una de las 4 pistas (RF-4).
- [ ] Cabezal visual sincronizado dentro de ±20 ms (RNF-1).
- [ ] Persistencia robusta ante `localStorage` vacío/corrupto (RNF-6).
- [ ] Cero etiquetas `<audio>` y cero archivos de sample en el proyecto.

---

## 12. Riesgos y mitigaciones

| Riesgo | Impacto | Mitigación |
|--------|---------|------------|
| Deriva temporal con `setInterval` ingenuo | Cabezal desincronizado (>20 ms) | Usar scheduler look-ahead anclado a `audioContext.currentTime` (§8.2) |
| Política de autoplay bloquea el audio | No suena al cargar | Crear/`resume()` el `AudioContext` en el primer gesto (Play) |
| Clicks/pops en los golpes | Calidad de audio pobre | Envolventes con rampas (ataque/decay) en vez de cortes abruptos |
| Hi-hat/caja "no son osciladores puros" | Duda sobre la restricción | Ruido sintetizado en runtime (sin archivo) cumple "sin samples"; documentarlo |
| `localStorage` lleno o deshabilitado | Falla al guardar | Capturar excepción y avisar al usuario sin romper la app |

---

## 13. Referencias

- Reto original: `docs/challenge.md`.
- Web Audio API — scheduling preciso: patrón *look-ahead* ("A Tale of Two Clocks").
- Síntesis de batería 808 con osciladores y ruido (kick con pitch-envelope, hi-hat con osciladores `square` filtrados).
