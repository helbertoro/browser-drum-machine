EL RETO
Una caja de ritmos en el navegador para cursos de producción musical

Los cursos de producción musical de Platzi carecen de herramientas prácticas interactivas. Una sencilla caja de ritmos en el navegador junto con un sintetizador de bajo permitiría a los estudiantes experimentar con ritmos sin instalar una DAW (estación de trabajo de audio digital) y permanecería como complemento permanente de la clase.

Construye un secuenciador de una sola página utilizando únicamente la Web Audio API:
Secuenciador de 8 pasos y 4 pistas: bombo (kick), caja (snare), hi-hat y bajo.
Control de tempo (60–180 BPM), botones de reproducir, detener y limpiar.
La pista de bajo tiene 3 octavas seleccionables por paso.
Controles ADSR (Ataque, Decaimiento, Sostenido y Liberación) para cada pista.
Guardar y cargar patrones usando localStorage.
Cabezal de reproducción visual sincronizado con el audio (precisión de ±20 ms). Sin etiquetas <audio>. Sin archivos de muestras (samples).

TERMINADO CUANDO
Un estudiante abre la página, programa 4 pasos en el bombo y pulsa reproducir → escucha un patrón four-on-the-floor. Cambia a la pista de bajo, crea un patrón y lo escucha. Guarda el proyecto, actualiza la página y vuelve a cargarlo. El sonido proviene de OscillatorNodes que tú mismo programaste manualmente.

STACK: Web Audio API · JavaScript Vanilla