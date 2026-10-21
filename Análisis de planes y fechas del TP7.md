---
tags:
  - tp7
---

# Análisis de planes y fechas del TP7


En 1.a, revisar el acceso a película por género y las claves de entrega/renglón. En 1.b, comparar versiones IN, EXISTS o JOIN con la misma semántica histórica, enlazando el voluntario con el historial por su identificador.

El archivo del TP7 tiene una discrepancia: el texto de 1.b menciona 01/01/2023, pero su SQL usa 24/07/1998. Elegir y declarar la fecha antes de comparar planes: cambiar el filtro puede cambiar mucho la selectividad.

## Qué explicar de cada plan

En 1.a, identificar primero dónde se aplica `genero = 'TERROR'`. Después seguir cómo las películas se vinculan a RENGLON_ENTREGA por código y cómo esos renglones alcanzan ENTREGA por nro_entrega. Si el filtro reduce mucho las películas, explicar cómo eso cambia el trabajo de las uniones siguientes. No afirmar que debe aparecer un tipo de join sin haber leído el plan real.

En 1.b, cada voluntario debe tener **algún registro histórico** cuya fecha cumpla el límite y cuya tarea tenga diferencia máxima–mínima de hasta 5000. Cambiar la fecha o consultar la tarea actual del voluntario cambia la pregunta. Una versión JOIN puede repetir a quien tiene varias coincidencias históricas; IN o EXISTS expresan existencia y evitan esa repetición exterior.

La fecha es una discrepancia de la fuente, no un detalle de formato. Para comparar dos versiones, usar en ambas 01/01/2023 si se sigue el texto, o 24/07/1998 si se analiza literalmente el SQL impreso, y dejarlo indicado. Compararlas con fechas distintas mezclaría reescritura con selectividad.

## Fuente de estudio

Enunciado del TP7 en la carpeta `../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).
