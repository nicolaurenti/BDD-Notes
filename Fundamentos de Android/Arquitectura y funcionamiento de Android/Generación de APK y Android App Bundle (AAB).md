# Generación de APK y Android App Bundle (AAB)

Temario **1.1.04** · 16 de noviembre de 2026

## Qué significa probar lo mismo que se va a distribuir

Imaginá que un equipo valida su APK de release en varios teléfonos. Tras publicar el AAB, la app falla en uno de esos modelos cuando usa una biblioteca nativa. El commit y la versión son los mismos. Sin embargo, el conjunto que recibió el dispositivo puede diferir del APK que se probó. Para investigar, hay que identificar qué se publicó, qué APK se generaron y cuáles quedaron instalados.

## El AAB da instrucciones para la distribución

Un **APK** es un paquete que el instalador de Android puede consumir cuando cumple los requisitos de firma y compatibilidad. Puede contener una app completa o ser una parte de una instalación compuesta por varios APK. Un **Android App Bundle** (AAB) contiene el código compilado y los recursos organizados en módulos. Es el formato que le entregás a Google Play para que construya los APK que corresponden a cada dispositivo.

Desde Gradle, en un proyecto sencillo, `:app:assembleRelease` produce un APK de release y `:app:bundleRelease` produce un AAB de release. Si existen flavors, usá las tareas de la variante real, por ejemplo `assembleProdRelease` y `bundleProdRelease`. Comparar un APK debug con el AAB de producción mezcla variables como código, recursos y firma. Aun tomando la misma variante, el APK que Gradle ensambló localmente y el conjunto generado a partir del AAB pueden tener distinta composición.

## APK base, APK de configuración y firma

Una instalación generada a partir del bundle puede incluir un **APK base**, con el punto de partida de la app, y **APK de configuración**, con partes destinadas a características del dispositivo. Algunas dimensiones son la arquitectura del procesador (**ABI**), la densidad de pantalla y el idioma. Si la app usa módulos de funcionalidad separados, también puede haber APK de esos módulos. El instalador trata el conjunto compatible como una instalación de la misma app. Copiar solo el APK base de una instalación dividida puede dejar fuera archivos necesarios.

Esta distribución reduce lo que se descarga en cada equipo. A cambio, una prueba con un APK único puede pasar por alto una diferencia de selección de archivos en una configuración particular. Un APK universal generado para diagnóstico puede ayudar a acotar una hipótesis, siempre que tengas presente que agrupa contenido de forma distinta. Para reproducir el caso distribuido, interesa generar y revisar el conjunto específico del dispositivo.

La firma también tiene dos usos. En **Play App Signing**, la clave de carga autentica el artefacto que subís. Google Play firma los APK que distribuye con la clave de firma de la app. Una compilación local firmada con otra clave puede fallar si intentás instalarla encima de la versión de Play. Si la sospecha es la firma, compará los certificados del APK instalado y del que querés instalar. Hacé las pruebas en un dispositivo de QA en el que puedas preparar el estado de instalación necesario. Una prueba con firma local tampoco sustituye la prueba a través de un canal interno de Play cuando el comportamiento depende de la identidad con la que se publica.

## Reproducir qué instalaría un dispositivo

Con `bundletool` y un dispositivo de pruebas conectado, podés generar un conjunto a partir del mismo AAB de release. Sin opciones de firma, bundletool intenta usar una clave de debug. Trabajá con un dispositivo dedicado y una instalación compatible con esa firma.

```sh
bundletool get-device-spec --output=device.json
bundletool build-apks --bundle=app-release.aab --output=qa-device.apks --device-spec=device.json
bundletool install-apks --apks=qa-device.apks
adb shell pm path com.example.app
```

`get-device-spec` describe la configuración del dispositivo. `build-apks` genera un archivo `.apks`, que contiene el conjunto. `install-apks` instala los APK pertinentes. El último comando enumera las rutas de los APK de la app instalada. Sustituí `com.example.app` por tu `applicationId`. Si además hay que verificar el contenido, abrí los APK relevantes del conjunto con **Analyze APK**.

Considerá un fallo con `UnsatisfiedLinkError` al cargar `libscanner.so`. Primero guardá el mensaje completo e identificá la ABI relevante. Inspeccioná si `lib/<abi>/libscanner.so` existe en los APK destinados a ese dispositivo. Si falta, revisá la dependencia nativa y las reglas de empaquetado de la variante. Si está, la falla todavía podría venir de una biblioteca de la que depende o de un símbolo que no se puede resolver. El texto del error ayuda a distinguir esos casos. Ver el archivo en el paquete demuestra que se incluyó. La carga exitosa se verifica repitiendo la operación en el dispositivo afectado.

## Síntesis para entrevistas

Explicá cuándo usarías cada artefacto. El APK permite instalar y probar un paquete concreto. El AAB permite publicar los insumos para que se generen APK adecuados a cada dispositivo. Al comparar QA y producción, identificá la variante, la firma y el conjunto instalado. Si el problema aparece solo en un modelo, inspeccionar ese conjunto te permite separar una ausencia en el paquete de un problema que ocurre al cargarlo.

Fuentes oficiales consultadas el 16 de noviembre de 2026. Los comandos ilustran el procedimiento y no se ejecutaron sobre un proyecto o dispositivo.

- [Android Developers · Android App Bundle](https://developer.android.com/guide/app-bundle)
- [Android Developers · bundletool](https://developer.android.com/tools/bundletool)
- [Android Developers · Firma de aplicaciones](https://developer.android.com/studio/publish/app-signing)
