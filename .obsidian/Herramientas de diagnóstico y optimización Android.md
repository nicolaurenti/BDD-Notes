# Herramientas de diagnóstico y optimización Android

17 de noviembre de 2026

## 1. Build Analyzer y recompilación de módulos

**Build Analyzer** es el informe de Android Studio sobre el último build. Muestra las tareas y plugins que más influyeron en su duración. Sirve para investigar una espera concreta, por ejemplo por qué editar una pantalla volvió lento el siguiente build. Compilá desde Android Studio y abrí **Build → Build Analyzer**. Si una tarea aparece como costosa, revisá si sigue siéndolo al repetir el mismo escenario. El informe se concentra en las tareas que más afectan la duración. [Build Analyzer](https://developer.android.com/build/build-analyzer)

Para ver **qué compiló Gradle**, ejecutá el mismo comando dos veces desde la raíz del proyecto. Entre ambas ejecuciones, cambiá solo el cuerpo de una función en el módulo que querés investigar. Usá la variante habitual de desarrollo y conservá las salidas previas, como en un build cotidiano.

```sh
./gradlew :app:assembleDebug --console=plain
```

Un resultado ilustrativo sería:

```text
> Task :checkout:compileDebugKotlin
> Task :payments:compileDebugKotlin UP-TO-DATE
> Task :app:compileDebugKotlin FROM-CACHE
```

El prefijo identifica el módulo y la tarea indica el tipo de trabajo. En el ejemplo se ejecutó la tarea de compilación de Kotlin de `:checkout`. En `:payments`, Gradle aprovechó la salida local vigente. En `:app`, recuperó la salida de un caché. Una tarea ejecutada puede compilar solo los archivos afectados si la compilación es incremental. Si también se ejecutan tareas de módulos que no esperabas, repetí con `--info` y buscá qué entradas cambiaron o por qué la tarea dejó de estar al día. [Gradle · Estados de las tareas](https://docs.gradle.org/current/userguide/more_about_tasks.html)

## 2. Configuration Cache y Build Cache

**Configuration Cache** conserva el resultado de configurar Gradle para una invocación, incluido el grafo de tareas. Si se puede reutilizar, acorta la fase previa a ejecutar tareas. Tiene sentido cuando la configuración de muchos módulos o plugins consume una parte importante del tiempo. Probalo con `--configuration-cache` y repetí la misma tarea. La salida indica si se reutilizó la configuración o si hay incompatibilidades que resolver. [Gradle · Configuration Cache](https://docs.gradle.org/current/userguide/configuration_cache.html)

**Build Cache** conserva las salidas de tareas cacheables, como ciertos resultados de compilación. Si las entradas relevantes coinciden, Gradle puede recuperar la salida y marcar la tarea como `FROM-CACHE`. Probalo con `--build-cache`. Si todo figura `UP-TO-DATE`, se reutilizaron las salidas vigentes del directorio de trabajo. Para comprobar una recuperación desde el caché podés usar un segundo checkout equivalente. En equipos grandes, un caché remoto permite compartir resultados entre máquinas y CI. [Gradle · Build Cache](https://docs.gradle.org/current/userguide/build_cache.html)

## 3. APK Analyzer

**APK Analyzer** abre el APK o AAB ya generado y muestra qué se empaquetó, cuánto aporta cada parte al tamaño y cómo quedó el manifest. Abrí **Build → Analyze APK** y seleccioná el artefacto. Si el tamaño subió tras agregar un SDK, compará dos builds de la misma variante. Mirá si creció `classes.dex` (código), `lib/` (bibliotecas nativas) o `assets/` (archivos). Esa ubicación orienta qué revisar. Para estimar descargas específicas por dispositivo cuando distribuís un AAB, usá un conjunto de APK generado con `bundletool`. [APK Analyzer](https://developer.android.com/studio/debug/apk-analyzer) · [bundletool](https://developer.android.com/tools/bundletool)

## 4. Profiler de Android Studio

El **Profiler** observa la app mientras se ejecuta en un teléfono o emulador. Según la tarea, vas a obtener una tendencia en vivo, una traza de ejecución, una captura de objetos o un registro de asignaciones.

### Abrir el Profiler y capturar un heap dump

En Android Studio, al lado de **Run**, abrí **More Actions → Profile 'app' with complete data**. Esto abre el Profiler con una variante *debuggable*, normalmente debug. En **Home**, seleccioná el proceso de tu app a la izquierda. Si la tarjeta de heap dump no aparece, comprobá que el proceso esté marcado como **Debuggable** y que elegiste el modo **with complete data**. [Abrir el Profiler](https://developer.android.com/studio/profile)

En la captura, el heap dump es la tarjeta **Analyze Memory Usage · Heap Dump**, arriba a la derecha. Seleccionala y dejá la app en el estado que querés observar. En **Start profiler task from**, cambiá **Not specified** por la opción de adjuntar al proceso en ejecución. Después presioná **Start profiler task**. El aviso **No task starting point selected** indica que todavía falta elegir ese punto de inicio. Para investigar qué ocurre al abrir la app, elegí el inicio desde el arranque cuando la tarea lo permita. [Iniciar una tarea](https://developer.android.com/studio/profile#start-profiling) · [Capturar un heap dump](https://developer.android.com/studio/profile/capture-heap-dump)

### CPU, hilos y fluidez

- **Capture System Activities · System Trace.** Registra una línea de tiempo de la app y el sistema. Muestra cuándo se ejecutan o esperan los hilos y permite correlacionar su actividad con eventos de renderizado. Usalo si una transición da tirones. Grabá el gesto y buscá en la traza qué hacía el hilo principal en el momento del tirón. Si esperaba, investigá qué lo bloqueaba antes de optimizar consumo de CPU. [System Trace](https://developer.android.com/studio/profile/cpu-profiler)
- **View Live Telemetry.** Muestra en vivo el uso de CPU y memoria y el estado de los hilos. Sirve para ubicar un pico mientras repetís una acción. Si la memoria crece al abrir y cerrar una pantalla, podés pasar a un heap dump o a LeakCanary para investigar qué objetos quedaron retenidos. [Telemetría en vivo](https://developer.android.com/studio/profile/inspect-app-live)
- **Find CPU Hotspots · Java/Kotlin Method Recording.** Registra entradas y salidas de métodos Java/Kotlin y sus tiempos. Usalo si necesitás ver el orden de llamadas o comprobar si un método específico se ejecuta repetidamente. Grabá solo el tramo de interés. La instrumentación altera los tiempos y la guía recomienda grabaciones de hasta cinco segundos. [Grabación de métodos](https://developer.android.com/studio/profile/record-java-kotlin-methods)
- **Find CPU Hotspots · Callstack Sample.** Toma muestras periódicas de la pila de llamadas, es decir, de qué métodos están activos en el hilo al momento de la muestra. Usalo como primera pasada para ubicar trabajo frecuente o costoso sin registrar cada invocación. Una llamada breve puede quedar entre dos muestras. [Muestreo de pilas](https://developer.android.com/studio/profile/sample-callstack)

### Objetos, fugas y asignaciones de memoria

- **Analyze Memory Usage · Heap Dump.** Toma una fotografía del heap gestionado, donde ART administra objetos como listas, modelos y actividades. Muestra qué objetos están presentes, cuánto ocupan y las referencias entre ellos. Sirve para saber qué impide liberar un objeto. Compará una captura inicial con otra después de abrir y cerrar varias veces una pantalla. Elegí una instancia en el resultado y seguí **References**. Si encontrás la cadena `singleton → listener → Activity`, revisá quién registró el listener y cuándo corresponde retirarlo. Repetí el recorrido después del cambio para ver si dejan de acumularse instancias. Una única captura aporta una pista y puede incluir objetos pendientes de recolección. [Heap dumps y referencias](https://developer.android.com/studio/profile/capture-heap-dump)
- **Find Memory Leaks with LeakCanary.** Detecta objetos que el recolector debería poder liberar, como una Activity destruida, y analiza la ruta de referencias que los retiene. Iniciá la tarea, reproducí la navegación y revisá el informe para ubicar la referencia que prolonga la vida del objeto. Sirve para detectar posibles fugas sin buscar cada instancia a mano. [Integración en Android Studio](https://developer.android.com/studio/releases/past-releases/as-quail-2-release-notes?authuser=5#leakcanary_in_android_studio_profiler) · [Cómo funciona LeakCanary](https://square.github.io/leakcanary/fundamentals-how-leakcanary-works/)
- **Track Memory Consumption · Java/Kotlin Allocations.** Registra creaciones y liberaciones de objetos durante un intervalo y permite revisar desde dónde se asignaron. Usalo si durante un scroll se generan muchas listas o modelos temporales y querés ubicar su origen. Grabá el recorrido, revisá las clases y pilas de asignación y repetilo tras el cambio. Si el registro completo ralentiza la app, probá **Sampled**. [Asignaciones Java/Kotlin](https://developer.android.com/studio/profile/record-java-kotlin-allocations)
- **Track Memory Consumption · Native Allocations.** Registra asignaciones y liberaciones en código nativo, por ejemplo con `malloc` y `free` en C. Usalo si el consumo aumenta al pasar por un SDK nativo. Grabá varias repeticiones y revisá cuántos bytes asignados en ese intervalo siguen sin liberarse y en qué pila se asignaron. Contrastá esa retención con la vida útil esperada de los datos. [Asignaciones nativas](https://developer.android.com/studio/profile/record-native-allocations)

Si querés saber **qué quedó en memoria** al cerrar una pantalla, usá un heap dump. Si querés saber **qué se creó mientras hacías scroll**, grabá asignaciones. Si querés encontrar **por qué se trabó el scroll**, empezá por System Trace y mirá qué estaba haciendo el hilo principal.

## 5. Android Vitals

**Android Vitals** reúne métricas de calidad de dispositivos de usuarios de Google Play. Muestra patrones de crashes, ANR (la app deja de responder) y otros problemas. Sirve para priorizar fallas que aparecen en producción. En **Play Console → Monitor and improve → Android vitals → Overview**, seleccioná la app, revisá el período y filtrá por versión o modelo. Si la tasa aumenta después de un release, abrí el grupo de errores y usá la traza para reproducirlo. Evaluá la tendencia de la versión corregida cuando haya datos suficientes. [Ayuda de Play Console · Android Vitals](https://support.google.com/googleplay/android-developer/answer/9844486?hl=en)

## 6. Latencia de llamados a servicios

La **latencia percibida** es el tiempo entre la acción del usuario y el momento en que puede usar el resultado. Incluye la petición HTTP y el trabajo del cliente antes y después de la respuesta. Para revisar la parte HTTP en desarrollo, abrí **View → Tool Windows → App Inspection → Network Inspector**. Admite OkHttp, también cuando Retrofit lo usa por debajo. Marcá además en tu app cuándo empieza la acción, termina la descarga, termina el procesamiento y se muestra el resultado. [Android Developers · Network Inspector](https://developer.android.com/studio/debug/network-profiler)

Si el resultado aparece a los 900 ms y la descarga termina a los 250 ms desde el toque, investigá qué pasa durante los 650 ms siguientes. Puede haber parseo, escritura local, otra llamada o trabajo en la UI. Si casi toda la espera ocurre antes de recibir los primeros bytes, compará con los tiempos del backend para distinguir procesamiento del servidor de red o conexión. Si domina la descarga, medí bytes y evaluá paginación o una respuesta con menos campos. Para dos peticiones independientes y necesarias, evaluá iniciarlas en paralelo y volvé a medir el flujo completo.

## Síntesis

Para un build lento, identificá qué trabajo se repite. Para un paquete pesado, inspeccioná qué archivo creció. Para un problema de memoria, seguí la cadena de referencias. Para un servicio lento, separá HTTP del tiempo hasta mostrar el resultado. En cada caso, repetí la misma medición después del cambio para comprobar si corregiste el problema.
