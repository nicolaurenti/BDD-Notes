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

### Niveles de aislamiento del estándar y PostgreSQL

Tabla de anomalías por nivel, diferencias del motor y sintaxis para elegir SERIALIZABLE.

#### Niveles del estándar usados en las filminas

| Nivel | Lectura sucia | Lectura no repetible | Fantasmas |
|---|---|---|---|
| READ UNCOMMITTED | Permitida por el nivel mínimo | Permitida | Permitidos |
| READ COMMITTED | Evitada | Permitida | Permitidos |
| REPEATABLE READ | Evitada | Evitada | Permitidos por la garantía mínima del estándar |
| SERIALIZABLE | Evitada | Evitada | Evitados; ejecución equivalente a un orden serial |

La tabla describe garantías mínimas, no una obligación de que ocurra la anomalía. Un motor puede impedir más de lo que exige un nivel.

#### Respuesta específica sobre PostgreSQL

En el comportamiento de PostgreSQL descrito por el material, READ COMMITTED es el valor por defecto y cada sentencia obtiene una instantánea apropiada: otra sentencia puede ver cambios que se confirmaron entre ambas. READ UNCOMMITTED no ofrece un comportamiento distinto de READ COMMITTED; no permite leer cambios sucios.

REPEATABLE READ mantiene una instantánea estable para la transacción y evita también fantasmas en sus lecturas, aunque pueden existir anomalías de serialización. SERIALIZABLE añade controles para detectar ejecuciones incompatibles con un orden serial y puede abortar una transacción, que debe reintentarse completa.

```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- Operaciones de la transacción.
COMMIT;
```

Una respuesta de examen distingue la tabla del estándar del comportamiento del motor. No afirmar que «serializable» solo significa no leer datos sucios: esa propiedad ya la proporciona READ COMMITTED.

## Otro mecanismo del material

Las filminas también presentan el ordenamiento por marcas de tiempo como mecanismo distinto de MVCC y del bloqueo elemental.

### Control de concurrencia por marcas de tiempo

Desarrollo del criterio de ordenamiento, las marcas por dato y los reinicios ante operaciones incompatibles.

#### Marcas de tiempo en las filminas

El control por timestamps ordena lógicamente las transacciones por su marca temporal. Para cada dato conserva marcas de lectura y escritura y rechaza una operación que contradiga el orden permitido. Por ejemplo, si una transacción más nueva ya escribió X, una lectura tardía de X por otra más antigua no puede aceptarse como si respetara el orden temporal. Ese enfoque evita la espera circular propia de los bloqueos, pero puede provocar reinicios y falta de progreso si se repiten conflictos. No confundir timestamp de ordenamiento con fecha del dato guardado.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
