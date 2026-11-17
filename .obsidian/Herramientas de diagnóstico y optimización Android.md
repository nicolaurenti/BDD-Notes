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

## 4. Heap dump y referencias vivas

Un **heap dump** es una captura de los objetos en el heap gestionado y de las referencias entre ellos. Sirve para averiguar por qué un objeto que debería liberarse sigue siendo alcanzable. Abrí **Profiler**, ejecutá la app como *debuggable* y elegí **Analyze Memory Usage (Heap Dump)**. Hacé una captura de referencia y otra tras entrar y salir varias veces de la pantalla afectada.

Buscá la clase, elegí una instancia y abrí **References**. Una cadena como `singleton → listener → Activity` indica que un objeto de vida larga conserva un listener que referencia la pantalla. Revisá quién registró el listener y en qué punto del ciclo de vida corresponde liberarlo. Repetí el recorrido después del cambio y compará cuántas instancias siguen retenidas. Una captura aislada puede incluir objetos cuya liberación todavía está pendiente. [Android Developers · Capturar un heap dump](https://developer.android.com/studio/profile/capture-heap-dump)

## 5. Android Vitals

**Android Vitals** reúne métricas de calidad de dispositivos de usuarios de Google Play. Muestra patrones de crashes, ANR (la app deja de responder) y otros problemas. Sirve para priorizar fallas que aparecen en producción. En **Play Console → Monitor and improve → Android vitals → Overview**, seleccioná la app, revisá el período y filtrá por versión o modelo. Si la tasa aumenta después de un release, abrí el grupo de errores y usá la traza para reproducirlo. Evaluá la tendencia de la versión corregida cuando haya datos suficientes. [Ayuda de Play Console · Android Vitals](https://support.google.com/googleplay/android-developer/answer/9844486?hl=en)

## 6. Latencia de llamados a servicios

La **latencia percibida** es el tiempo entre la acción del usuario y el momento en que puede usar el resultado. Incluye la petición HTTP y el trabajo del cliente antes y después de la respuesta. Para revisar la parte HTTP en desarrollo, abrí **View → Tool Windows → App Inspection → Network Inspector**. Admite OkHttp, también cuando Retrofit lo usa por debajo. Marcá además en tu app cuándo empieza la acción, termina la descarga, termina el procesamiento y se muestra el resultado. [Android Developers · Network Inspector](https://developer.android.com/studio/debug/network-profiler)

Si el resultado aparece a los 900 ms y la descarga termina a los 250 ms desde el toque, investigá qué pasa durante los 650 ms siguientes. Puede haber parseo, escritura local, otra llamada o trabajo en la UI. Si casi toda la espera ocurre antes de recibir los primeros bytes, compará con los tiempos del backend para distinguir procesamiento del servidor de red o conexión. Si domina la descarga, medí bytes y evaluá paginación o una respuesta con menos campos. Para dos peticiones independientes y necesarias, evaluá iniciarlas en paralelo y volvé a medir el flujo completo.

## Síntesis

Para un build lento, identificá qué trabajo se repite. Para un paquete pesado, inspeccioná qué archivo creció. Para un problema de memoria, seguí la cadena de referencias. Para un servicio lento, separá HTTP del tiempo hasta mostrar el resultado. En cada caso, repetí la misma medición después del cambio para comprobar si corregiste el problema.
