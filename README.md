# Partitura Live

PWA local para profesores de música: carga una partitura (MusicXML o Guitar Pro), activa el
micrófono y la partitura resalta y desplaza automáticamente la posición por la que va el alumno.
Pensada para compartir pantalla por Zoom. Todo el procesamiento es local; no hay servidor,
cuentas ni envío de audio.

## Ejecutar en local
El micrófono y el service worker requieren un origen seguro: `localhost` o HTTPS.
Abrir el archivo con doble clic (`file://`) NO funciona.

    npx serve .           # o: python3 -m http.server 8080
    # abrir [localhost](http://localhost:3000) (o :8080) en Chrome/Edge

La primera carga necesita internet para descargar la biblioteca de notación (alphaTab) desde
el CDN; después queda en caché y la app funciona sin conexión.

## Instalar como aplicación
En Chrome: icono «Instalar» en la barra de direcciones → se abre en ventana propia,
ideal para compartirla en Zoom («Compartir pantalla → Ventana»).

## Desplegar en GitHub Pages
1. Crea un repositorio y sube `index.html`, `manifest.json`, `sw.js`, `icon.svg`, `README.md`.
2. Settings → Pages → Source: *Deploy from a branch* → rama `main`, carpeta `/ (root)`.
3. La app quedará en `[tu_usuario.github.io](https://TU_USUARIO.github.io/NOMBRE_REPO/)` (HTTPS, válido para micrófono).
Las rutas son relativas, así que funciona en subcarpetas sin cambios.

## Uso en clase
1. Cargar partitura → elegir pista si hay varias → (opcional) escribir compás de inicio → «Ir».
2. 🎤 Micrófono → el alumno toca. La nota resaltada avanza cuando la detección es estable.
3. «Ocultar notas futuras» muestra solo el compás actual + N siguientes (ajustable).
4. «Modo presentación» oculta la barra superior. Esc para salir.
5. Flechas ← → avanzan/retroceden; espacio pausa. En «Modo manual» el micrófono no mueve la posición.

## Formatos
- MusicXML: `.xml`, `.musicxml`, `.mxl`
- Guitar Pro: `.gp3`, `.gp4`, `.gp5`, `.gpx`, `.gp`
Se usa la primera voz del primer pentagrama de la pista elegida.

## Acordes
Un tiempo con varias notas se trata como un acorde. La app comprueba en el espectro si están
presentes las notas esperadas (por defecto, al menos el 75 %). En Ajustes se puede relajar a
«aceptar cualquiera de sus notas» o ignorar la octava. No se intenta transcribir polifonía libre.

## Simulación y depuración
Ajustes → Simulación: `C4,D4,E4,G4` o `C4+E4+G4` para acordes. Ajustes → Panel de depuración
muestra pitch, frecuencia, confianza, nota esperada, índice, compás y estado del tracker.

## Arquitectura (clases dentro de index.html, separables a módulos)
PitchDetector (audio → PitchEvent) → App (estabilidad/confirmación) → ScoreTracker (posición)
→ ScoreRenderer (alphaTab + cursor/máscaras). ScoreModel convierte la partitura en eventos
`{index, measure, tick, midis, names, duration, voice}`. Parámetros centralizados en `CONFIG`.

## Limitaciones conocidas
- Reconocimiento no perfecto con ruido, reverberación o micrófonos pobres: la app prioriza
  quedarse esperando antes que avanzar mal.
- Notas repetidas consecutivas requieren un ataque audible o un breve silencio entre ellas.
- Polifonía libre (dos voces independientes) no está soportada.
