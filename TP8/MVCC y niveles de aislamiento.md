---
tags:
  - tp8
---

# MVCC y niveles de aislamiento

El ejercicio 4 pide investigar cómo controla la concurrencia PostgreSQL y qué niveles ofrece. Las filminas presentan multiversión, además de bloqueos y ordenamiento por marcas de tiempo. Estos mecanismos no se simulan todos como el protocolo elemental S/X del ejercicio anterior.

## Control por multiversión

MVCC mantiene versiones de los datos y determina cuáles son visibles para una lectura. Si T1 prepara un cambio de saldo todavía no confirmado, una lectura común de T2 puede ver una versión anterior confirmada, en lugar de leer el valor sucio. La visibilidad depende de la instantánea y del nivel de aislamiento.

Esto reduce interferencias entre lectores y escritores, pero no significa que desaparezcan los bloqueos ni que dos escrituras incompatibles puedan hacerse siempre a la vez. Tampoco se puede trasladar literalmente al ejercicio 3 la conducta del motor: allí se pide simular solicitudes y liberaciones S/X explícitas.

## Garantías de aislamiento

Los niveles fijan garantías sobre lo que puede ver una transacción. Se comparan las garantías mínimas del estándar y el comportamiento PostgreSQL que pide investigar el ejercicio.

[[TP8 - Niveles de aislamiento del estándar y PostgreSQL|Niveles de aislamiento del estándar y PostgreSQL]]

## Otro mecanismo del material

Las filminas también presentan el ordenamiento por marcas de tiempo como mecanismo distinto de MVCC y del bloqueo elemental.

[[TP8 - Control de concurrencia por marcas de tiempo|Control de concurrencia por marcas de tiempo]]

## Fuente de estudio

Enunciado del TP8 en la carpeta `../../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
