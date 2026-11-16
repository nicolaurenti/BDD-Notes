# Proceso de compilación de una aplicación Android

Temario **1.1.03** · 16 de noviembre de 2026

## El bug que ya está presente en el APK

Un equipo añade un recurso `R.string.api_host` y lo usa al construir el cliente HTTP. Desde Android Studio la app se conecta al servidor de pruebas. En producción también se conecta a pruebas, aunque el build de release pasó y el dispositivo instaló el APK. Para saber dónde corregirlo conviene seguir tres transformaciones: qué insumos seleccionó el build, qué generaron los compiladores y qué terminó dentro del paquete que se ejecutó.

## La variante determina qué entra al build

Gradle ejecuta tareas con dependencias entre sí. El **Android Gradle Plugin** (AGP) aporta las que conocen la estructura de una app Android. Al pedir `:app:assembleRelease`, el resultado esperado para el módulo `app` es el APK de la variante de release. Si el proyecto tiene *product flavors*, como `demo` y `prod`, la variante suele combinar un flavor con un tipo de build. En ese caso, `:app:assembleProdRelease` identifica la combinación concreta.

Cada variante selecciona **source sets**, que son grupos de archivos como `src/main`, `src/prod`, `src/release` y `src/prodRelease`. Con una sola dimensión de flavors, para un recurso de la variante `prodRelease` tiene prioridad `prodRelease`, seguido por `release`, `prod` y `main`. Esto permite que dos grupos declaren el mismo recurso y que la variante determine el valor efectivo. El código Kotlin y Java de los source sets activos se compila junto. Declarar la misma clase con el mismo nombre completo en dos source sets activos puede fallar por duplicación aunque una colisión equivalente entre valores de recursos se resuelva por prioridad.

El manifest también tiene su propio mecanismo. AGP combina declaraciones del proyecto y las librerías, y resuelve conflictos según las reglas de fusión. Para diagnosticar una diferencia entre variantes, seleccioná la variante correcta en Android Studio y abrí **Merged Manifest**. Esa vista permite rastrear de dónde salió una declaración, por ejemplo el permiso que añadió una dependencia. El manifest contenido en el APK permite confirmar que la declaración llegó al paquete final.

## Las dos transformaciones que confluyen en el APK

**Recursos.** AAPT2 procesa los recursos en dos pasos. En *compile*, convierte archivos de recursos individuales a representaciones intermedias. En *link*, enlaza esos resultados, resuelve referencias y construye la tabla de recursos que relaciona identificadores, configuraciones y valores. El código puede referirse a un identificador mediante `R.string.api_host`, y al ejecutar la app Android puede elegir el valor correspondiente a la configuración del dispositivo, por ejemplo el idioma. La selección de archivos de la variante ocurre en el build. La selección entre configuraciones que quedaron empaquetadas ocurre al usar el recurso. Ambas importan cuando el valor que ves en el teléfono difiere del que esperabas.

**Código.** Los compiladores producen clases en bytecode JVM a partir de Kotlin y Java. La cadena de herramientas Android transforma esas clases y las dependencias que correspondan en archivos DEX. En un build con optimización habilitada, R8 participa en ese recorrido. El paquete final reúne DEX, recursos enlazados, manifest, assets y bibliotecas nativas si las hay. Luego, para que el APK sea instalable, debe estar firmado. Al ejecutar la app, ART carga el DEX que llegó al dispositivo. El build y la ejecución están conectados por ese artefacto.

## Cómo localizar la primera evidencia del fallo

Volvamos al servidor equivocado. Primero confirmá qué variante produjo **el archivo exacto** que probó el equipo. Consultá la salida de Gradle, y si es necesario buscá el ID del paquete y el código de versión con **Analyze APK** en Android Studio. Después inspeccioná la tabla de recursos del APK y localizá `string/api_host`. Si el APK correcto contiene el valor de pruebas para la configuración relevante, ya podés reproducir el error sin abrir la app. Revisá el source set que aporta el valor y reconstruí la misma variante. Volvé a inspeccionar el nuevo APK antes de instalarlo.

Si el APK contiene el valor esperado, seguí hacia la ejecución. Por ejemplo, poné un punto de interrupción en la creación del cliente o inspeccioná en un entorno de pruebas el host de la primera petición. Si el cliente usa un valor distinto al del recurso, investigá si lo toma de una preferencia local, configuración remota u otra fuente. Evitá registrar tokens o URLs que incluyan credenciales. Confirmar un recurso en el paquete prueba que está disponible. Para demostrar qué usa el cliente, hace falta observar el código que lee la configuración efectiva.

Otras fallas dejan señales diferentes. Si el build muestra un error de tipos de Kotlin, empezá en el código y las dependencias de la variante. Si AAPT2 informa que no puede resolver un recurso, revisá las entradas de la variante y sus referencias. Si Gradle falla al construir o firmar el paquete, revisá la tarea fallida y su causa antes de pasar al dispositivo. Si ya existe un APK y Android rechaza la instalación, conservá el diagnóstico del instalador. Y si el proceso llega a arrancar y falla, usá el stack trace o una traza de esa misma versión. Haber encontrado un APK en la carpeta de salida tampoco acredita que pertenezca al build que acaba de fallar.

## Síntesis para entrevistas

Explicá el recorrido como una cadena de entradas y salidas. La variante decide qué código, manifest y recursos aportan los módulos. Las herramientas convierten esas entradas en DEX, recursos enlazados y un manifest efectivo. El APK permite comprobar qué llegó al paquete. La ejecución permite comprobar cómo se usa. Cuando algo falla, ubicar en cuál de esos puntos aparece por primera vez una señal comprobable reduce lo que tenés que investigar.

Fuentes oficiales consultadas el 16 de noviembre de 2026. El caso es ilustrativo.

- [Android Developers · Configurar variantes de compilación](https://developer.android.com/build/build-variants)
- [Android Developers · AAPT2](https://developer.android.com/tools/aapt2)
- [Android Developers · apkanalyzer](https://developer.android.com/tools/apkanalyzer)
