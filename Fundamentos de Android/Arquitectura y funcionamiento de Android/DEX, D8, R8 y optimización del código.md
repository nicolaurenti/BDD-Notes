# DEX, D8, R8 y optimización del código

Temario 1.1.05 · 2026-11-18 · Fundamentos de Android

## Una release pasó de 31 a 27 MB

Un equipo activa la optimización en una app de compras. Al comparar los paquetes que recibiría el mismo teléfono, la descarga inicial estimada pasa de 31 a 27 MB. Es tentador reportar que «R8 eliminó 4 MB de código». Para sostener esa afirmación habría que abrir los artefactos y ver qué cambió. Parte del ahorro puede venir de nombres más cortos o de código reescrito. Si también se habilitó la reducción de recursos, otra parte puede venir de imágenes o layouts que quedaron sin uso. El porcentaje depende además del dispositivo y los módulos entregados.

En esa misma release, la app abre y deja navegar, pero falla cuando el servidor habilita una pantalla promocional que se busca por nombre de clase. Ambos resultados pueden estar relacionados. R8 puede quitar código al que no encuentra un camino y puede renombrar una clase que sí conserva. Para entender qué ahorró el optimizador y qué pudo romper, conviene empezar por el archivo que efectivamente llega al dispositivo.

El escenario, los números y los fragmentos de esta lectura son ilustrativos.

## Del bytecode de Kotlin a los archivos DEX

Cuando compilás Kotlin para Android, el compilador genera bytecode en archivos JVM `.class`. Estos representan clases, métodos y sus instrucciones. En ese punto el compilador ya hizo el trabajo de entender el lenguaje y comprobar, por ejemplo, que una llamada reciba un argumento del tipo adecuado. El código de las bibliotecas compiladas que incorpora la app también puede llegar al build como bytecode JVM. Android necesita representar ese código en su formato ejecutable, DEX.

Un archivo DEX contiene definiciones de clases, instrucciones y tablas compartidas con identificadores de tipos, métodos, campos y cadenas. Una instrucción que invoca un método puede apuntar a su identificador dentro del archivo. El método puede estar definido en ese DEX, en otro DEX o en el framework. Una llamada a `Log.d`, por ejemplo, puede verse en el código de la app, aunque la implementación de `Log.d` la proporciona Android. Cuando el código se carga y se ejecuta, ART usa esa representación. Por eso en APK Analyzer existen columnas distintas para métodos definidos y referenciados. El límite de 65.536 referencias de métodos se aplica por DEX. Si ves `classes2.dex`, tenés más de un archivo para inspeccionar y sumar al medir tamaño. [Formato DEX de AOSP](https://source.android.com/docs/core/runtime/dex-format) · [Multidex](https://developer.android.com/build/multidex) · [Vista de DEX en APK Analyzer](https://developer.android.com/studio/debug/apk-analyzer#view-dex-files).

En la ruta habitual sin optimización de la aplicación, el Android Gradle Plugin usa D8 para convertir bytecode JVM a DEX. D8 también puede combinar DEX intermedios, lo cual permite reutilizar parte del trabajo de builds incrementales. En esa conversión puede hacer *desugaring*: reescribe ciertas construcciones del bytecode a una forma compatible con versiones anteriores de Android. Una lambda compilada usando una característica que el nivel de API objetivo para compatibilidad no soporta puede representarse mediante una clase generada. Por separado, algunas APIs de biblioteca de Java requieren configurar *core library desugaring*. La compatibilidad del lenguaje y la disponibilidad de una API se resuelven en niveles distintos. [D8](https://developer.android.com/tools/d8) · [Desugaring de lenguaje y de bibliotecas](https://developer.android.com/studio/write/java8-support).

Con la optimización habilitada, R8 analiza y transforma el programa y genera DEX como parte de esa ruta del build. El modelo útil para depurar es comparar los DEX finales de ambas rutas. La diferencia entre ellos incluye las decisiones que tomó el optimizador sobre qué preservar, cómo reescribirlo y con qué nombres emitirlo. [Cómo funciona R8](https://developer.android.com/topic/performance/app-optimization/enable-app-optimization#r8-optimization-overview).

## Qué puede sacar R8 y cómo reconocerlo

R8 construye un grafo de referencias a partir de puntos de entrada de la app, entre ellos componentes declarados en el manifiesto y elementos protegidos por reglas de conservación. Si una actividad alcanza un repositorio, y el repositorio llama a un adaptador de una biblioteca, la cadena permite identificar código necesario. En cambio, una biblioteca puede traer adaptadores para varios proveedores que no tienen ningún camino desde esos puntos de entrada. R8 puede eliminarlos. Esta eliminación de código inalcanzable se llama *code shrinking*. Que una pantalla nunca aparezca en las sesiones que mediste es un dato de ejecución. Para eliminar su código, R8 necesita poder decidir durante el build que sobra. [Optimización con R8](https://developer.android.com/topic/performance/app-optimization/enable-app-optimization).

R8 también puede reescribir código alcanzable. Con *method inlining*, por ejemplo, puede poner el cuerpo de una función auxiliar directamente donde se la llama. Eso deja a R8 analizar juntos el cuerpo y los valores conocidos en ese lugar. Si la función ya quedó incorporada en sus llamadas y no hace falta conservar una definición separada, buscar su nombre en el APK puede dar cero resultados aunque el comportamiento siga disponible. Esta optimización tampoco implica por sí sola un ahorro de bytes: copiar un cuerpo a varios lugares tiene un costo que puede compensarse con otras simplificaciones. [Transformaciones de R8](https://developer.android.com/topic/performance/app-optimization/enable-app-optimization#r8-optimization-overview).

La ofuscación acorta nombres de clases, campos y métodos. Si R8 cambia el nombre de una clase, también puede actualizar las referencias que controla. Eso reduce el tamaño de sus representaciones y cambia los nombres que aparecen en los stack traces. El artefacto `mapping.txt` permite relacionar esos nombres con los originales. Usá el mapping del build exacto que produjo el APK o AAB: incluso un build cercano puede tener un mapeo distinto. [APK Analyzer y mapeos](https://developer.android.com/studio/debug/apk-analyzer#load_proguard_mappings) · [R8 retrace](https://developer.android.com/tools/retrace).

## Cómo medir el ahorro

Primero definí qué querés atribuir. Para medir el efecto de activar la optimización de código, compará **A**, una build sin optimización ni reducción de recursos, con **B**, la misma build con R8 activo y la reducción de recursos deshabilitada. Si además querés medir el efecto de la configuración que publicarías, generá **C** con optimización de código y reducción de recursos. A→B estima el impacto de la optimización de código con esa configuración. B→C estima el efecto adicional de habilitar la reducción de recursos sobre esa base. Con la reducción optimizada, código y recursos se analizan juntos, así que B→C puede incluir diferencias en los DEX. Si tu DSL activa ambas optimizaciones juntas, podés reportar A→C como el efecto combinado y usar el desglose para explicar dónde cambió el paquete. [Configuración de R8 y reducción de recursos según AGP](https://developer.android.com/topic/performance/app-optimization/enable-app-optimization).

Generá los artefactos con el mismo commit y la misma variante de release. Conservá el flavor, las dependencias, los perfiles, la firma y las versiones de herramientas. La única variación deliberada debe estar en la configuración de optimización que querés medir. Guardá cada artefacto con un nombre distinto antes de recompilar: Gradle puede reutilizar la misma ruta de salida. Revisá la versión de AGP para ver cómo se expresa el cambio. En el DSL anterior los controles son `isMinifyEnabled` y `isShrinkResources`. La guía de AGP 9.3 o superior documenta `optimization { enable = true }`, que activa ambas optimizaciones, y también documenta que el DSL anterior sigue soportado. Comparar un debug con un release sirve para entender cómo difieren tus builds habituales. Para atribuir el delta a R8 necesitás controlar las otras diferencias. [Activación según la versión del plugin](https://developer.android.com/topic/performance/app-optimization/enable-app-optimization#enable-optimization).

### Cuánto cambió el código empaquetado

En Android Studio, abrí el artefacto con **Build > Analyze APK** y elegí **Compare with previous APK** para cargar la segunda versión. En cada APK, sumá la contribución de todos los `classes*.dex` usando la misma columna, por ejemplo **Raw File Size**, que muestra cuánto aporta la entrada al tamaño del archivo. Si los DEX cambian de distribución entre builds, `classes.dex` podría crecer mientras la suma cae. Si analizás el AAB para investigar composición, compará los mismos módulos. Para calcular qué recibe un teléfono, usá la medición de entrega del siguiente apartado. [Comparación de artefactos con APK Analyzer](https://developer.android.com/studio/debug/apk-analyzer#compare-files).

En CI podés inspeccionar dos APK del mismo tipo con `apkanalyzer`, que forma parte de las herramientas de línea de comandos del SDK. La comparación muestra tamaño anterior, nuevo y diferencia por entrada. Los comandos de DEX ayudan a revisar qué archivos intervienen y cuántas referencias hay en cada uno. Usá bytes para calcular diferencias y convertí a MB o MiB al presentar el resultado, manteniendo la unidad explícita. [Comandos de apkanalyzer](https://developer.android.com/tools/apkanalyzer#commands).

```bash
apkanalyzer apk compare --different-only --files-only A.apk B.apk
apkanalyzer dex list A.apk
apkanalyzer dex list B.apk
```

Tras medir el cambio, inspeccioná `lib/`, `assets/`, `res/` y la tabla de recursos. Si dos tercios del espacio se concentran en bibliotecas nativas o assets, una gran reducción porcentual de DEX puede mover poco el total. Para explicar una diferencia de código, cargá `mapping.txt` y, si está disponible, `usage.txt` del mismo artefacto optimizado. El primero ayuda a localizar elementos renombrados. El segundo permite ver qué elementos se quitaron durante el shrinking. Sirven para investigar la causa, mientras que la comparación de bytes cuantifica el resultado. [Tamaños y mapeos en APK Analyzer](https://developer.android.com/studio/debug/apk-analyzer).

### Cuánto menos descarga el usuario

Si distribuís por Google Play como AAB, para estimar la descarga inicial localmente generá un conjunto de APK por cada AAB con `bundletool`. Medí ambos con la misma especificación de dispositivo, que incluye SDK, densidad, arquitecturas y configuraciones de idioma. Usá la misma versión de `bundletool` y la misma configuración de firma para ambos conjuntos. El comando mide los módulos de instalación inicial por defecto. Si cambiás ese alcance o evaluás otra configuración de dispositivo, registralo junto al resultado. Si querés hablar de toda la base instalada, compará varias configuraciones representativas. [Estimaciones de descarga con bundletool](https://developer.android.com/tools/bundletool#measure_size).

```bash
bundletool get-device-spec --output=device.json
bundletool build-apks --bundle=A.aab --output=A.apks
bundletool build-apks --bundle=C.aab --output=C.apks
bundletool get-size total --apks=A.apks --device-spec=device.json
bundletool get-size total --apks=C.apks --device-spec=device.json
```

El resultado de `get-size total` es una estimación de descarga comprimida. Con una especificación parcial puede devolver un rango, así que compará la misma configuración o los mismos extremos del rango en ambos builds. El tamaño del AAB local incluye contenido para configuraciones que un teléfono concreto puede no recibir. El espacio ocupado tras instalar y el tráfico de una actualización desde una versión ya instalada son otras mediciones.

### Cómo leer los números juntos

Supongamos que al comparar la contribución de los DEX al APK obtenés 12 MB en A y 8 MB en B. El ahorro de DEX es de 4 MB, o **33,3 %**. Para ese mismo teléfono, la descarga estimada de A es 31 MB, la de B es 28,5 MB y la de C es 27 MB. La optimización de código explica un ahorro estimado de **2,5 MB de descarga (8,1 % respecto de A)**. La configuración completa explica **4 MB (12,9 % respecto de A)**. La reducción adicional al habilitar el análisis de recursos es de 1,5 MB de descarga en esta comparación.

Que aparezca «4 MB» en la inspección de DEX y en la descarga completa es casualidad. Uno describe archivos de código dentro del APK y el otro describe la entrega comprimida de un conjunto de APK. Para calcular el porcentaje siempre usá como denominador el tamaño anterior de la misma métrica. Conviene guardar los artefactos, el mapping, la especificación de dispositivo y los valores en bytes junto a la versión de la app. Cuando un release posterior crezca, podés reproducir la comparación y ubicar el cambio en la parte exacta del paquete.

## Cómo comprobar que se conservó lo necesario

Volvamos a la pantalla promocional. Imaginá que el servidor envía `com.example.PromoScreenFactory` en una configuración y el cliente la abre así:

```kotlin
val factoryClass = Class.forName(config.factoryClassName)
val factory = factoryClass.getDeclaredConstructor().newInstance() as ScreenFactory
factory.open()
```

Al compilar, la expresión tiene tipo y la llamada a `Class.forName` es válida. La clase concreta llega como un valor de configuración durante la ejecución. Si solo se usa por esa vía, R8 puede no detectar que debe retenerla con su nombre original. Si la clase se encuentra pero se quitó el constructor sin argumentos, el fallo aparece en `getDeclaredConstructor()`. Los dos fallos se investigan en lugares distintos de la misma secuencia. [Cargas por nombre y constructores por reflexión](https://developer.android.com/topic/performance/app-optimization/keep-rule-examples#reflection-with-classes-loaded-by-name).

Reproducí con el artefacto optimizado y la misma configuración de servidor. Si la traza está ofuscada, usá `retrace mapping.txt crash.txt` con el mapping que corresponde a ese artefacto. Después buscá la clase en los DEX que están instalados. Una entrada que figura como eliminada en el reporte de shrinking respalda una hipótesis distinta de una clase presente con el nombre acortado. Si el constructor falta, comprobá primero qué firma pide el código. También vale comprobar que el servidor haya enviado un nombre válido y que el módulo que define la clase esté instalado: un `ClassNotFoundException` puede aparecer por cualquiera de esos motivos. [Diagnóstico de optimización](https://developer.android.com/topic/performance/app-optimization/troubleshoot-the-optimization) · [Retrace](https://developer.android.com/tools/retrace).

Si el contrato actual exige el nombre completo de la clase y el constructor sin argumentos, una regla acotada hace visibles esos dos requisitos:

```proguard
-keep,allowoptimization class com.example.PromoScreenFactory {
    <init>();
}
```

La regla preserva la clase y el constructor que se busca. `allowoptimization` permite optimizar los elementos protegidos dentro de esas restricciones. La llamada a `open()` se expresa a través de una interfaz en el bytecode, de modo que este ejemplo no requiere agregarla a la regla como un miembro que se descubre mediante reflexión. Si la carga dinámica forma parte del contrato de una biblioteca reutilizable, la biblioteca puede distribuir las reglas necesarias para sus consumidores. Una regla que abarca un paquete entero puede conservar muchos elementos adicionales y alterar justamente el ahorro que querés medir. [Opciones de keep rules](https://developer.android.com/topic/performance/app-optimization/add-keep-rules) · [Ejemplos de reflexión](https://developer.android.com/topic/performance/app-optimization/keep-rule-examples).

Si el equipo controla ambos lados del contrato, el servidor puede enviar un ID de producto como `promo` y el cliente resolverlo con una referencia directa a su fábrica. Eso le permite a R8 seguir la relación y desacopla los nombres del código del protocolo de red. Si ya hay versiones desplegadas que consumen los nombres anteriores, la migración requiere mantener el formato que esperan o versionar la configuración. Con cualquiera de los dos diseños, probá el mismo flujo con una build optimizada. Repetí después la medición de la configuración corregida, que es la que realmente vas a distribuir. Si también querés afirmar que el arranque mejoró, ejecutá un benchmark del arranque por separado. [Validación funcional y rendimiento tras optimizar](https://developer.android.com/topic/performance/app-optimization/test-the-optimization).

## Síntesis

DEX es la representación del código que usa ART. D8 genera DEX en la ruta habitual sin optimización de la app. R8 genera DEX aplicando, según la configuración, reducción de código, transformaciones y ofuscación. En una entrevista podés explicar cuánto cambió el código con los bytes de todos los DEX, cuánto cambia la descarga con el conjunto de APK del mismo dispositivo y qué verificaste en la release que mediste. Un contrato dinámico como la carga por nombre se hace explícito con una regla precisa o con una referencia que el optimizador pueda seguir.

Documentación oficial consultada el 2026-11-18. Las fuentes están al final de los párrafos correspondientes.
