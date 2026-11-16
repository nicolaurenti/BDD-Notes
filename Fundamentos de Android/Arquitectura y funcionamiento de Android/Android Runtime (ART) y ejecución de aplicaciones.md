# Android Runtime (ART) y ejecución de aplicaciones

Temario **1.1.02** · 15 de noviembre de 2026

## Por qué dos arranques en frío pueden tardar distinto

Imaginá que medís el arranque de tu app recién instalada. Después la usás varias veces, volvés a medir con el proceso cerrado y el tiempo baja. Las dos mediciones son arranques en frío en el sentido de que Android tiene que crear el proceso. Aun así, pueden partir de estados distintos. El sistema puede haber preparado código compilado con información de ejecuciones anteriores. También pueden haber cambiado los datos que la app ya tiene guardados, la caché del sistema y lo que hace el resto del dispositivo. La diferencia de tiempo plantea una hipótesis sobre ART, pero hace falta un experimento controlado para atribuírsela.

## Qué ejecuta ART

Al compilar una app Android, el código de Kotlin y Java termina convertido a **DEX**, un formato de bytecode. El bytecode contiene instrucciones para el runtime de Android. ART carga clases, ejecuta sus métodos y administra los objetos del código gestionado. Gradle y las herramientas de compilación producen el DEX. Una vez que la app está instalada, el runtime del dispositivo se ocupa de ejecutarlo.

ART puede ejecutar un método de tres formas. Con el **intérprete**, el runtime lee las instrucciones DEX y realiza sus operaciones sin generar previamente código de máquina para ese método. Con **JIT** (*just-in-time*), ART genera instrucciones de máquina durante la ejecución para métodos que vale la pena optimizar. Esa compilación consume trabajo de CPU y el resultado ocupa espacio en una caché de código en memoria. Si el método vuelve a ejecutarse con frecuencia, las siguientes llamadas pueden aprovechar el resultado.

Con **AOT** (*ahead-of-time*), Android prepara código de máquina con anticipación y puede reutilizarlo en ejecuciones posteriores. La política de compilación y los perfiles disponibles influyen en qué código se prepara y cuándo. Compilar más métodos tiene un costo de CPU y almacenamiento. Una selección útil concentra ese costo en rutas que importan al usuario y deja que ART ejecute el resto con los mecanismos disponibles. Una sola ejecución de la app puede combinar métodos interpretados, compilados por JIT y compilados por AOT.

## Cómo se decide qué conviene compilar

Mientras la app se usa, ART puede registrar datos sobre clases y métodos ejecutados. Un **perfil** es un conjunto de información de ese tipo que guía qué código vale la pena preparar. Si Android dispone de un perfil apropiado, puede usarlo para compilar por adelantado una parte del código. Así, el conocimiento de una ejecución puede beneficiar a una ejecución posterior, incluso cuando el proceso anterior terminó.

Un **Baseline Profile** permite que el equipo identifique de antemano los recorridos críticos, como el arranque y la primera navegación a una pantalla muy usada. El perfil se distribuye con la app y, cuando el mecanismo de instalación y compilación lo aplica, ART puede preparar esos caminos antes de que el usuario los recorra. El archivo describe clases y métodos. El código de máquina se produce en el dispositivo cuando corresponde. Su utilidad depende de elegir recorridos representativos y verificar que el perfil se aplicó durante la medición.

## Qué pasa con los objetos y las pausas

ART también administra el **heap gestionado**, la parte de la memoria donde viven objetos como colecciones, instancias de modelos y otros valores referenciados por el código administrado. El **garbage collector** identifica objetos que dejaron de ser alcanzables y puede reutilizar su memoria. Si un listener mantiene una referencia a una pantalla que ya terminó, esa pantalla sigue siendo alcanzable. El GC puede funcionar correctamente y la app seguir reteniendo memoria por cómo están conectados sus objetos.

Generar muchos objetos temporales puede aumentar el trabajo de recolección. Si un tirón de la interfaz coincide con actividad de GC en una traza, conviene revisar qué se estaba asignando y cuánto explica esa actividad del retraso. Una medición de arranque o de renderizado indica el síntoma. La traza ayuda a separar tiempo de ejecución, recolección y espera por otras operaciones, como lectura de disco.

## Cómo comprobar si un perfil ayuda de verdad

Supongamos que el primer arranque de una app tarda en mostrar una lista. Parte del tiempo se consume preparando la UI y parte esperando la consulta que trae los datos. Un experimento con **Macrobenchmark** puede comparar la misma variante de app en el mismo dispositivo y con el mismo estado de datos. En una serie de mediciones configurás el benchmark sin compilación AOT inducida por perfiles. En la otra, usás compilación parcial con un Baseline Profile y exigís que esté disponible. En ambas series medís arranques en frío. Así podés estimar qué cambia al preparar el código prioritario con anticipación.

Si la mejora es chica y la traza muestra que la espera dominante es la consulta, tiene sentido investigar el acceso a los datos o cómo se programa ese trabajo. Si la mejora aparece en el recorrido incluido en el perfil, la evidencia apoya su utilidad para ese caso. Para decidir si mantenerlo, medí la métrica que corresponda al problema, como el tiempo hasta el primer frame o el tiempo hasta que el contenido principal está listo para usarse. Conservá las condiciones del benchmark para volver a medir cuando cambie la ruta de arranque.

## Síntesis para entrevistas

Explicá las dos etapas de compilación por su propósito. Las herramientas del proyecto producen DEX. ART ejecuta ese DEX en el dispositivo y combina interpretación, JIT y AOT según el código y los perfiles que tiene disponibles. Un Baseline Profile ayuda a elegir qué merece compilarse por anticipado. Para defender esa optimización, compará ejecuciones equivalentes y usá una traza para entender qué parte del tiempo puede cambiar con el perfil.

## Fuentes oficiales

Consultadas el 15 de noviembre de 2026.

- [AOSP · Android Runtime y Dalvik](https://source.android.com/docs/core/runtime)
- [AOSP · Compilación JIT y perfiles en ART](https://source.android.com/docs/core/runtime/jit-compiler)
- [Android Developers · Baseline Profiles](https://developer.android.com/topic/performance/baselineprofiles/overview)
