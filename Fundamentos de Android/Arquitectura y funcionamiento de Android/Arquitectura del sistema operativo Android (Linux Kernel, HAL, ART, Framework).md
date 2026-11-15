# Arquitectura del sistema operativo Android (Linux Kernel, HAL, ART, Framework)

Temario **1.1.01** · 15 de noviembre de 2026

## Una sola API de cámara para muchos dispositivos

Una app abre la cámara con `CameraManager.openCamera()`. El mismo código puede ejecutarse en teléfonos con sensores, procesadores de imagen y drivers distintos. Además, el sistema tiene que decidir si esa app puede usar la cámara y qué ocurre si el recurso ya está ocupado. Para entender cómo Android resuelve esas dos necesidades, conviene distinguir el código que ejecuta la app, el servicio que coordina el acceso y la implementación que controla el hardware.

## Linux Kernel y HAL

El **kernel de Linux** es la parte del sistema operativo que administra recursos como CPU y memoria. Decide qué hilos obtienen tiempo de CPU, mantiene espacios de memoria virtual separados entre procesos y ofrece acceso al hardware a través de drivers. Un **driver** traduce operaciones del sistema a instrucciones que un dispositivo concreto puede ejecutar. En el caso de la cámara, el driver forma parte del trabajo necesario para que el sensor y los demás componentes del dispositivo produzcan imágenes.

El driver y la **HAL** (*Hardware Abstraction Layer*, capa de abstracción de hardware) resuelven problemas diferentes. La HAL define un contrato que el software de Android puede usar aunque cambie el dispositivo. En la cámara, ese contrato abarca operaciones como configurar flujos de captura y enviar solicitudes. El fabricante implementa el contrato con el código y los componentes que necesita su equipo. Así, el servicio de cámara puede trabajar en términos de sesiones, solicitudes y resultados sin incorporar la lógica específica de cada sensor.

Un contrato común también puede expresar diferencias. Dos teléfonos pueden implementar Camera2 y ofrecer distintas resoluciones o capacidades. La app debe consultar qué admite la cámara concreta, por ejemplo mediante `CameraCharacteristics`, antes de configurar una función que dependa de ello. La decisión de diseño es mantener un contrato compartido y permitir que cada dispositivo declare las capacidades específicas que puede ofrecer.

## ART y Android Framework

**ART** (*Android Runtime*) ejecuta el código administrado que llega al dispositivo en formato **DEX**, el bytecode que usan las aplicaciones Android. El runtime también gestiona el heap de objetos bajo su control. Su recolector de basura puede recuperar memoria de objetos que dejaron de ser alcanzables. Ese trabajo es distinto de la gestión de memoria virtual que hace el kernel, que opera sobre recursos del proceso y páginas de memoria. ART también puede compilar bytecode a instrucciones de máquina, con mecanismos como JIT durante la ejecución y AOT de forma anticipada. Las decisiones de compilación quedan para la siguiente lectura.

Cuando la app invoca `openCamera()`, ART ejecuta el código administrado de esa llamada dentro del proceso de la app. La API pertenece al **Android Framework**, que expone las clases de plataforma y conecta llamadas con servicios del sistema cuando necesitan actuar fuera del proceso de la app. Es importante distinguir los dos usos de «framework» en este recorrido. Una parte es el código de la API que corre dentro de la app. Otra es la implementación del servicio al que esa API accede. En cámara, `CameraService` coordina el acceso y se comunica con la HAL.

La comunicación entre procesos se hace mediante **Binder**, el mecanismo de Android para enviar una solicitud a otro proceso y recibir resultados o callbacks. El código de la app llama a una API de Kotlin o Java y, debajo, el cliente del framework realiza transacciones Binder con el servicio. Eso permite que el servicio aplique las reglas de acceso y arbitre el uso del recurso desde un proceso distinto. ART, por su parte, sigue ejecutando el código administrado de la app y sus callbacks. Por eso conviene ubicar ART donde se está ejecutando el código, en vez de tratarlo como un servicio intermedio al que se le manda cada operación de hardware.

## Qué pasa al abrir la cámara

Supongamos que una pantalla abre la cámara para escanear un documento. Primero, el código de la app pide abrir una cámara concreta a través de Camera2. La API comunica la solicitud al servicio del sistema mediante Binder. El servicio aplica las reglas de acceso y consulta la disponibilidad. Si la apertura se concreta, la app recibe `onOpened()`. Después puede configurar una sesión de captura con las superficies que recibirán las imágenes, como una superficie para la vista previa.

Cuando se solicitan capturas, el servicio se comunica con la implementación de la HAL que ofrece ese dispositivo. Debajo intervienen componentes del fabricante y los drivers que controlan el hardware. Esta distinción entre apertura y captura ayuda a interpretar síntomas. Si el código llega a `onOpened()` pero la vista previa queda negra, la apertura se completó y todavía hay que verificar el resultado de configurar la sesión y sus superficies. Si llega `ERROR_CAMERA_IN_USE`, el sistema informó que la cámara está ocupada. La investigación empieza en la disponibilidad del recurso y en cómo la app abre y cierra sus sesiones.

En un proyecto profesional, esta distinción ayuda a decidir qué registrar. Para una falla de cámara conviene guardar la etapa que falló, el código de error disponible y el modelo y versión del sistema, sin registrar imágenes. Si el fallo aparece en un solo modelo, esa concentración permite priorizar una reproducción en el dispositivo, pero todavía quedan hipótesis que comprobar. Por ejemplo, la app podría estar solicitando una configuración que ese equipo no admite. Primero se contrastan sus capacidades publicadas y el resultado de configurar la sesión. Para comprobar una corrección, se repite la misma operación en el equipo afectado y en otro que ya funcionaba.

## Síntesis para entrevistas

Un recorrido de cámara permite explicar las capas por su responsabilidad. ART ejecuta el código administrado de la app. La API del framework llega por Binder a un servicio que coordina el acceso. La HAL ofrece un contrato para que ese servicio trabaje con distintas implementaciones del fabricante. Linux administra los recursos del sistema y proporciona los drivers que intervienen en el control del hardware. Al diagnosticar un fallo, la etapa en que aparece y los datos que devuelve la API ayudan a decidir dónde buscar evidencia primero.

## Fuentes oficiales

Consultadas el 15 de noviembre de 2026.

- [Android Developers — Arquitectura de la plataforma](https://developer.android.com/guide/platform)
- [AOSP — Arquitectura del subsistema de cámara](https://source.android.com/docs/core/camera#architecture)
- [Android Developers — CameraDevice.StateCallback](https://developer.android.com/reference/android/hardware/camera2/CameraDevice.StateCallback)
