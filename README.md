<h1 align="center">Laguna Media</h1>

<p align="center">
  <strong>Descargá de YouTube y TikTok, convertí audio y video, y reproducí tu biblioteca.<br/>
  Todo en una sola aplicación, para Linux, Windows y Android.</strong>
</p>

<p align="center">
  <a href="../../releases/latest"><img src="https://img.shields.io/github/v/release/JonathanDevHurtado/LagunaMedia-App?label=versi%C3%B3n&color=blue" alt="Última versión" /></a>
  <a href="../../releases"><img src="https://img.shields.io/github/downloads/JonathanDevHurtado/LagunaMedia-App/total?label=descargas&color=success" alt="Descargas" /></a>
  <img src="https://img.shields.io/badge/Plataformas-Linux%20%7C%20Windows%20%7C%20Android-lightgrey" alt="Plataformas" />
  <a href="LICENSE"><img src="https://img.shields.io/badge/Licencia-Propietaria-red" alt="Licencia propietaria" /></a>
</p>

---

> ### 📌 Este repositorio es informativo
>
> Acá **no está el código fuente**. Publica la información del producto, la
> documentación técnica y los **binarios compilados** en
> [**Releases**](../../releases/latest).
>
> Laguna Media es software **propietario**: los binarios son de uso personal y
> el código no se distribuye. Ver [Licencia](#licencia-y-componentes-de-terceros).

---

## Tabla de contenido

- [Qué es Laguna Media](#qué-es-laguna-media)
- [Descargas](#descargas)
- [Instalación paso a paso](#instalación-paso-a-paso)
- [Cómo funciona](#cómo-funciona)
- [Qué versión elegir](#qué-versión-elegir)
- [Detalles técnicos y decisiones de diseño](#detalles-técnicos-y-decisiones-de-diseño)
- [Rendimiento](#rendimiento)
- [Historia: los errores que atravesamos](#historia-los-errores-que-atravésamos)
- [Control de calidad](#control-de-calidad)
- [Privacidad y transparencia](#privacidad-y-transparencia)
- [Preguntas frecuentes](#preguntas-frecuentes)
- [Licencia y componentes de terceros](#licencia-y-componentes-de-terceros)
- [Autor](#autor)
- [Aviso legal](#aviso-legal)

---

## Qué es Laguna Media

Laguna Media nació de una molestia concreta: hacer lo que quería hacer exigía
**cuatro o cinco programas distintos**. Uno para bajar el video de YouTube,
otro para bajar la música, un navegador con extensión para TikTok, un editor
para pasar los audios a Opus, un conversor aparte para cambiar de códec, y el
reproductor del sistema para escuchar el resultado.

Laguna Media unifica todo eso en una sola ventana:

| Función | Qué hace |
|---|---|
| **Descargar** | Videos y audio de **YouTube** y **TikTok**, hasta 4K/2160p, o sólo la pista de audio (MP3, Opus, M4A, FLAC, WAV) con bitrate seleccionable |
| **Convertir** | Audio y video entre formatos: MP3, Opus, M4A, FLAC, WAV, OGG, MP4 (H.264 / HEVC / AV1), MKV, WebM. Incluye *remux* sin recomprimir |
| **Reproducir** | Reproductor de música y de video con biblioteca local, miniaturas, carátulas y caché |
| **Personalizar** | Tema oscuro "neon" armado desde ocho colores y una tipografía, 27 atajos reasignables |
| **Auto-repararse** | Cuando YouTube cambia algo y las descargas dejan de funcionar, la app **actualiza su propio motor** sin reinstalar nada |

---

## Descargas

Todos los binarios de la última versión están en
**[Releases → última versión](../../releases/latest)**.

| Archivo | Plataforma | Requisitos |
|---|---|---|
| `LagunaMedia-x86_64.AppImage` | Linux x86_64 | glibc 2.35+ (Fedora 36+, Ubuntu 22.04+, Debian 12+) |
| `LagunaMedia-portable-win64.zip` | Windows 10 / 11 x64 | Ninguno (portable, sin instalador) |
| `LagunaMedia-Android-*.apk` | Android 7.0+ (API 24) | ~150 MB de espacio libre |
| `LagunaMediaLite-x86_64.AppImage` | Linux x86_64 | Variante mínima |
| `LagunaMediaLite-portable-win64.zip` | Windows 10 / 11 x64 | Variante mínima |
| `SHA256SUMS.txt` | — | Checksums para verificar integridad |

Verificar que un binario se descargó bien (Linux):

```bash
sha256sum -c SHA256SUMS.txt
```

---

## Instalación paso a paso

### Linux (AppImage)

El AppImage es un único archivo que ya trae todo adentro: no se instala, no
toca el sistema y se puede borrar cuando quieras.

```bash
chmod +x LagunaMedia-x86_64.AppImage
./LagunaMedia-x86_64.AppImage
```

Si tu distro no tiene FUSE instalado:

```bash
./LagunaMedia-x86_64.AppImage --appimage-extract-and-run
```

### Windows

1. Descomprimí el `.zip` en la carpeta que quieras (por ejemplo `C:\LagunaMedia`).
2. Ejecutá `lagunamedia.exe`.

No hay instalador ni entradas en el registro. Se desinstala borrando la carpeta.
Si Windows SmartScreen avisa que el programa no está firmado, es normal en
software independiente: *Más información → Ejecutar de todas formas*.

### Android

1. Pasá el `.apk` al teléfono (USB, nube o mensajería) y abrilo desde el
   explorador de archivos.
2. Cuando Android avise, permití **instalar apps de orígenes desconocidos**.
3. La primera vez la app prepara su motor (descomprime Python, yt-dlp y ffmpeg):
   puede tardar unos segundos. Después arranca al instante.

También se puede instalar por cable con `adb install -r LagunaMedia-Android-*.apk`.

---

## Cómo funciona

Laguna Media no reinventa la descarga de video: se apoya en dos herramientas
excelentes y les pone una interfaz cómoda, un flujo de conversión propio y
un sistema de actualización que mantiene todo funcionando con el tiempo.

```
        YouTube / TikTok
               │  (enlace)
               ▼
   ┌───────────────────────────┐
   │   Interfaz                │   Escritorio: Qt / PySide6 (QSS propio)
   │   (7 pestañas)            │   Android: Jetpack Compose + Material 3
   └─────────────┬─────────────┘
                 │  tarea (link + formato + destino)
                 ▼
   ┌───────────────────────────┐        ┌──────────────────────────┐
   │  Motor de descarga        │───────▶│  FFmpeg                  │
   │  yt-dlp  (actualizable)   │        │  mezcla, extrae, convierte│
   └─────────────┬─────────────┘        └────────────┬─────────────┘
                 │  archivo temporal                  │
                 └──────────────┬─────────────────────┘
                                ▼
                 Destino elegido: carpeta, galería del
                 sistema (MediaStore) o carpeta SAF
```

### El motor: yt-dlp, y por qué se actualiza solo

La descarga la hace **[yt-dlp](https://github.com/yt-dlp/yt-dlp)**, el
descargador de código abierto más mantenido que existe. El problema clásico de
cualquier app de este tipo es que YouTube cambia sus defensas cada tanto y la
copia que trae la aplicación queda obsoleta: las descargas empiezan a fallar
con errores raros y hay que esperar a que el desarrollador publique una versión
nueva.

Laguna Media rompe esa dependencia: la app **descarga la última versión oficial
de yt-dlp desde GitHub** y la usa en el momento, sin reinstalar ni actualizar
nada. Un botón. Si algo deja de descargar, se arregla así.

### La conversión: FFmpeg con progreso real

Las conversiones las hace **FFmpeg**. Lo importante no es que convierta (eso lo
hacen todos), sino que **muestre progreso real**: la app lee la salida de
`ffmpeg -progress pipe:1`, así que la barra avanza según el trabajo que falta y
no según una estimación inventada. Además:

- Convierte **varios archivos a la vez** (según los núcleos de la CPU).
- **No recomprime el audio** cuando no hace falta (`-c:a copy`).
- Usa `libsvtav1` para AV1 (mucho más rápido que `libaom-av1`) y cae a otro
  códec compatible si el sistema no lo tiene.

### Android: aplicación nativa, no un reempaquetado

La versión Android está escrita en **Kotlin con Jetpack Compose**. No es la app
de escritorio envuelta: es una aplicación nativa que usa
**[youtubedl-android](https://github.com/JunkFood02/youtubedl-android)** (el
mismo yt-dlp + ffmpeg + aria2c compilados como bibliotecas nativas) y un
**servicio en primer plano** con notificación de progreso, para que el sistema
no mate la descarga si cambiás de aplicación.

---

## Qué versión elegir

| | **Laguna Media** | **Laguna Media Lite** |
|---|---|---|
| Descargar YouTube / TikTok | ✅ | ✅ |
| Convertir audio y video | ✅ | ✅ |
| Reproductor de música y video | ✅ | ❌ |
| Conversión de códec (H.264/HEVC/AV1) | ✅ | ❌ |
| Biblioteca con miniaturas y caché | ✅ | ❌ |
| Temas y personalización | ✅ | ❌ (tema fijo) |
| Pestañas | 7 | **3** |
| Arranque | ~2.8 s | **< 1 s** |
| Tamaño | Completo | Más chico |

**Lite** existe para una sola cosa: bajar y convertir lo más rápido posible. No
carga reproductores, biblioteca, miniaturas ni temas. Si sólo querés eso, Lite
es la indicada.

---

## Detalles técnicos y decisiones de diseño

### Tecnologías

| Capa | Escritorio | Android |
|---|---|---|
| Lenguaje | Python 3.10+ | Kotlin 2.0 |
| Interfaz | PySide6 6.11 (Qt 6) + QSS generado | Jetpack Compose + Material 3 |
| Motor de descarga | yt-dlp (actualizable en caliente) | youtubedl-android (yt-dlp + ffmpeg + aria2c) |
| Multimedia | FFmpeg (GPL) | FFmpeg empaquetado en el APK |
| Estado / datos | QSettings + caché SQLite | DataStore + MediaStore/SAF |
| Inyección de dependencias | — | Koin |
| Empaquetado | PyInstaller (onedir) → AppImage / Flatpak / portable ZIP | Gradle + AGP → APK |
| CI/CD | GitHub Actions (5 binarios por Release) | ídem |

### Por qué está construido así

**Un solo código fuente para todos los formatos de escritorio.**
Al principio había una copia del código por cada formato de empaquetado
(AppImage, Flatpak) y se desincronizaban: un arreglo se aplicaba en una copia y
en la otra no. Ahora el AppImage lo referencia con un enlace simbólico y el
Flatpak con una ruta relativa en su manifiesto: una sola fuente de verdad.

**Cada pestaña en su propio módulo, cargada al primer uso.**
Las siete pestañas no se construyen al abrir la app. Eso bajó el arranque de
~10 s a ~2.8 s. Además, si el conversor de video falla, el reproductor ni se
entera: los errores no se propagan entre pestañas.

**La lógica de biblioteca y reproducción vive fuera de la interfaz.**
Está en un módulo sin dependencias de Qt, así que se puede probar y reutilizar
sin UI. Un bug de orden se arregla una vez, no dos.

**Cancelación cooperativa en vez de matar hilos.**
Ver la sección de historia: matar un hilo de la interfaz en caliente es lo que
producía los cierres con error. Ahora se cancela de forma ordenada (se mata el
proceso de FFmpeg que está corriendo, se espera a que todo termine y recién ahí
se cierra).

**Nada de telemetría.**
La app no envía estadísticas ni identificadores. Sale a internet sólo para
descargar lo que le pedís y para actualizar su motor.

---

## Rendimiento

| Aspecto | Antes | Ahora |
|---|---|---|
| Arranque (escritorio) | ~10 s | **~2.8 s** (pestañas diferidas + splash) |
| Biblioteca local | Escaneo completo en cada arranque | **Caché SQLite**: instantánea |
| Descarga | Un flujo por archivo | **8 fragmentos en paralelo**; con **aria2c** instalado, 16 conexiones (`-x 16 -s 16 -k 1M`) |
| Conversión por lotes | Archivo por archivo | **En paralelo** según núcleos de CPU |
| Progreso de conversión | Barra estimada | **Real** (`ffmpeg -progress pipe:1`) |
| AV1 | `libaom-av1` (lento) | **`libsvtav1`** con fallback automático |
| Android: primer arranque | — | Prepara el motor una vez y queda listo |

---

## Historia: los errores que atravesamos

Esta sección existe porque el proyecto se rompió varias veces de formas
interesantes, y cada rotura dejó una lección. Todo esto está arreglado.

| Síntoma | Causa real | Solución |
|---|---|---|
| **YouTube y TikTok dejaron de descargar** | Dos cosas a la vez: se usaba una opción que yt-dlp había descontinuado (`hls_prefer_native`) y la copia empaquetada del motor había quedado vieja frente a los cambios de YouTube | Se eliminó la opción obsoleta y se agregó el **actualizador del motor en la app** |
| **Conversiones "rotas", sin progreso** | El núcleo leía mal la salida de FFmpeg: el porcentaje quedaba en 0 % hasta terminar | Se reescribió el núcleo para leer `ffmpeg -progress pipe:1` y mostrar progreso real |
| **La app se cerraba con error al salir** (un *segmentation fault*) | Se mataban los hilos de trabajo con `QThread.terminate()`. Matar un hilo de Qt en caliente es *undefined behavior* y el proceso caía al cerrar | **Cancelación cooperativa**: se cancela la tarea, se mata el proceso de FFmpeg/yt-dlp que esté corriendo, se espera a que terminen los hilos y recién ahí se cierra |
| **Crash al abrir la pestaña Configuración** | Una pestaña recibía tres argumentos y su constructor aceptaba dos | Firma corregida |
| **Bucle infinito** al recorrer el layout | Se leía siempre el primer elemento (`itemAt(0)`) sin avanzar el índice | Iteración corregida |
| **"Abrir archivo" abría cualquier cosa** | Se usaba el widget real de una pestaña que todavía no se había cargado | Se resuelve la pestaña antes de usarla |
| **La biblioteca no detectaba archivos nuevos** | La caché comparaba contra una carpeta fija (`~/Música`) en lugar de la carpeta elegida por el usuario | Ahora se compara contra la carpeta configurada |
| **La versión Lite se quedó sin el arreglo del motor** | El código estaba duplicado en dos proyectos separados y una copia quedó vieja | Lite vive en el mismo repositorio y comparte el motor, para que no se vuelva a desincronizar |

La lección general: **casi todos estos bugs venían de duplicar código o de
forzar el cierre de procesos**, no de la lógica del negocio. Las decisiones de
diseño de la sección anterior salieron de acá.

---

## Control de calidad

- **Pipeline automático (CI)**: cada cambio en el código compila **los cinco
  binarios** (Linux ×2, Windows ×2, Android) en máquinas limpias. Si algo no
  compila, el cambio no se publica.
- **Verificación antes de publicar**: el APK se compila en un entorno con JDK 17
  y Android SDK 35, se firma y se verifica su contenido (paquete, versión,
  permisos) antes de subirlo.
- **Checksums**: cada Release incluye `SHA256SUMS.txt` para comprobar que el
  archivo descargado es exactamente el que se publicó.
- **Versionado semántico**: `MAJOR.MINOR.PATCH`, con historial de cambios
  documentado versión por versión.
- **El motor se mantiene actualizado desde la app**, así que los cambios de
  YouTube no dependen de que el autor publique una versión nueva.
- **Sin telemetría ni rastreo**: nada que auditar.

---

## Privacidad y transparencia

| Pregunta | Respuesta |
|---|---|
| ¿La app envía estadísticas de uso? | No. |
| ¿A qué servidores se conecta? | Sólo a YouTube/TikTok (para lo que le pedís) y a GitHub (para actualizar el motor, cuando vos lo pedís). |
| ¿Dónde guarda la configuración? | En tu equipo: `~/.config/LagunaMedia/` (Linux) o `%APPDATA%` (Windows). |
| ¿Dónde guarda archivos temporales? | En la caché del sistema, y los borra al terminar cada tarea. |
| ¿Necesita mis contraseñas? | No. Si un video pide sesión, opcionalmente podés indicarle el **navegador** del que tomar las cookies; la app las lee localmente, no las envía a ningún lado. |
| ¿Requiere cuenta? | No. |

---

## Preguntas frecuentes

**YouTube dejó de descargar, ¿se rompió?**
No. YouTube cambia seguido. Abrí la configuración y usá **Actualizar motor**
(Android: en la pantalla de descarga). La app baja la última versión oficial de
yt-dlp y sigue funcionando.

**YouTube me pide iniciar sesión / no me deja descargar.**
Elegí tu navegador en **Cookies del navegador**. La app usa la sesión con la que
ya estás logueado, sin que tengas que ingresar nada.

**TikTok no descarga.**
TikTok suele exigir sesión: cargá un archivo `cookies.txt` exportado desde el
navegador (Android) o elegí tu navegador (escritorio).

**Las descargas van lentas.**
Instalá `aria2c` (`sudo dnf install aria2` / `sudo apt install aria2`). La app
lo detecta y baja con 16 conexiones.

**El AppImage no abre.**
Puede ser FUSE o la versión de glibc. Probá
`./LagunaMedia-x86_64.AppImage --appimage-extract-and-run`. Si el problema es
glibc, tu distribución es más vieja que Ubuntu 22.04.

**¿Puedo usar la app para descargar contenido con derechos de autor?**
Ver el [Aviso legal](#aviso-legal). Es tu responsabilidad.

**¿Por qué el APK es tan grande (~150 MB)?**
Porque adentro lleva Python, yt-dlp, FFmpeg y aria2c compilados para cuatro
arquitecturas de teléfono. Todo eso es lo que hace que descargue y convierta
sin depender de apps externas.

**¿Va a estar en Google Play?**
Por ahora no. Se distribuye como APK directo desde Releases.

---

## Licencia y componentes de terceros

Laguna Media es **software propietario**. El código fuente no se distribuye y
los binarios se ofrecen para uso **personal y no comercial** conforme al
archivo [`LICENSE`](LICENSE). No se permite redistribuir, modificar, vender ni
hacer ingeniería inversa.

El software incorpora componentes de terceros que mantienen **sus propias
licencias**, y esta licencia no las restringe:

| Componente | Licencia | Uso |
|---|---|---|
| [FFmpeg](https://ffmpeg.org/legal.html) | GPL (build *gpl*) | Mezcla, extracción y conversión de audio/video |
| [PySide6 / Qt for Python](https://www.qt.io/licensing) | LGPL v3 | Interfaz de la versión de escritorio |
| [yt-dlp](https://github.com/yt-dlp/yt-dlp) | Unlicense (dominio público) | Motor de descarga |

---

## Autor

**Jonathan Hurtado**

- GitHub: [@JonathanDevHurtado](https://github.com/JonathanDevHurtado)
- Correo: [JonathanHurtadDev@proton.me](mailto:JonathanHurtadDev@proton.me)

En el perfil de GitHub están publicados los demás repositorios del autor.

---

## Aviso legal

Laguna Media es una herramienta de uso personal. El autor **no** aloja,
distribuye ni promueve contenido con derechos de autor: la aplicación
únicamente permite descargar material al que el usuario tenga derecho de
acceso, y el uso que se le dé es responsabilidad exclusiva del usuario.

Descargar o redistribuir contenido protegido puede infringir los términos de
servicio de las plataformas y las leyes de propiedad intelectual de tu país.
Verificá siempre qué estás autorizado a descargar.

Este software se ofrece **"tal cual"**, sin garantía de ningún tipo.
