# Historial de cambios — Laguna Media

Todos los cambios notables de Laguna Media se documentan en este archivo.

El formato sigue [Keep a Changelog](https://keepachangelog.com/es-ES/1.1.0/) y
el proyecto usa [Versionado Semántico](https://semver.org/lang/es/).

---

## [3.0.2] — 2026-10-20

### Corregido (Android)
- **Errores de descarga sin explicación**: la tarjeta mostraba la última línea
  de la salida del motor (casi siempre un *warning*) y a veces quedaba vacía.
  Ahora se usa la línea `ERROR:` y se traducen los casos frecuentes:
  - video privado, eliminado o restringido por país;
  - TikTok bloqueando el video para tu conexión
    (*IP address is blocked from accessing this post*);
  - link que no corresponde a un video; error 404.
- Si el mensaje quedara vacío, se muestra el tipo de error en lugar de nada.

---

## [3.0.1] — 2026-10-20

Parche de estabilidad: la app de escritorio abre siempre y Android vuelve a
descargar sin tocar nada.

### Corregido
- **Escritorio: la app no abría** (la ventana aparecía y se cerraba sola). La
  ventana se creaba dentro del callback de un `QTimer` en una variable local;
  al terminar la función, Python la recolectaba y Qt cerraba la última ventana.
  Ahora `MainWindow` se crea de forma síncrona en `main()`.
- **Android: las descargas fallaban**. La detección de antigüedad del motor
  nunca funcionaba porque `LocalDate.parse("2025.11.12")` no acepta el formato
  de yt-dlp (puntos en vez de guiones). El motor nunca se actualizaba y YouTube
  lo rechazaba (SABR / *missing a url*). Ahora:
  - la edad del motor se parsea correctamente;
  - si tiene **45 días o más**, se actualiza **antes** de descargar;
  - si la descarga falla por un cambio de YouTube, se actualiza el motor y se
    **reintenta una vez**.
- **Android: links con `HTTPS://` en mayúsculas** (como los escribe el teclado
  del teléfono) eran rechazados. Ahora el esquema se compara sin distinguir
  mayúsculas.
- **Android: auto-actualización del motor** integrada en el flujo de descarga,
  sin intervención del usuario.

### Cambiado
- **Android: YouTube y TikTok separados**, con selector de plataforma, un
  campo de link propio para cada una y validación por dominio. El link
  compartido desde otra app elige la plataforma automáticamente. (En escritorio
  ya eran dos pestañas independientes).
- **Android: se eliminó la opción de cookies**; la app funciona sin configurar
  nada.
- Actualizada la librería del motor de descarga de Android
  (`youtubedl-android` 0.18.1).

---

## [3.0.0] — 2026-10-17

Revisión grande: se arreglan las descargas de YouTube/TikTok y las conversiones
que estaban rotas, se agrega soporte **Android** y la variante **Lite**, y se
optimiza la velocidad de descarga y conversión.

### Corregido
- **YouTube/TikTok ya no descargaban**: se eliminó la opción `hls_prefer_native`
  (deprecada por yt-dlp) y el motor se puede actualizar desde la app.
- **Conversiones rotas / sin progreso**: se reescribió el núcleo de conversión;
  el progreso real se lee de `ffmpeg -progress pipe:1`.
- **Crash al abrir Configuración**: una pestaña recibía tres argumentos y su
  constructor aceptaba dos.
- **Bucle infinito** al recorrer el layout (`itemAt(0)` sin avanzar).
- **"Abrir archivo"** usaba el widget real de una pestaña todavía no cargada.
- **Caché de biblioteca**: comparaba contra `~/Música` en vez de la carpeta
  elegida por el usuario.
- **Cierre con *segmentation fault***: se reemplazó `QThread.terminate()` por
  cancelación cooperativa.

### Agregado
- **Versión Android nativa** (Kotlin + Jetpack Compose): descarga, conversión y
  servicio en primer plano.
- **Laguna Media Lite**: variante mínima de escritorio (descarga + conversión).

### Rendimiento
- **8 fragmentos en paralelo** en las descargas y **aria2c** opcional
  (16 conexiones).
- **Conversión en paralelo** con `ThreadPoolExecutor` y `-c:a copy` cuando no
  hace falta recomprimir.
- **Arranque ~3 s** en escritorio (pestañas diferidas) y biblioteca instantánea
  con caché SQLite.
- **AV1** con `libsvtav1` (mucho más rápido que `libaom-av1`) y fallback
  automático.

---

## Versiones anteriores

El historial completo desde la 0.6.0 está documentado en el repositorio del
proyecto. Cambios principales de las versiones previas a la 3.0.0:

- **2.0.0** — el proyecto pasó de llamarse *ZYMax* a **Laguna Media** (sin
  cambios funcionales).
- **1.x** — reproductor de música y video, biblioteca con caché, temas,
  atajos configurables y conversión de códec.
