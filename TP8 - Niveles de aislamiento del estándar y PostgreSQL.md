---
tags:
  - tp8
---

# TP8 - Niveles de aislamiento del estándar y PostgreSQL

Tabla de anomalías por nivel, diferencias del motor y sintaxis para elegir SERIALIZABLE.

## Niveles del estándar usados en las filminas

| Nivel | Lectura sucia | Lectura no repetible | Fantasmas |
|---|---|---|---|
| READ UNCOMMITTED | Permitida por el nivel mínimo | Permitida | Permitidos |
| READ COMMITTED | Evitada | Permitida | Permitidos |
| REPEATABLE READ | Evitada | Evitada | Permitidos por la garantía mínima del estándar |
| SERIALIZABLE | Evitada | Evitada | Evitados; ejecución equivalente a un orden serial |

La tabla describe garantías mínimas, no una obligación de que ocurra la anomalía. Un motor puede impedir más de lo que exige un nivel.

## Respuesta específica sobre PostgreSQL

En el comportamiento de PostgreSQL descrito por el material, READ COMMITTED es el valor por defecto y cada sentencia obtiene una instantánea apropiada: otra sentencia puede ver cambios que se confirmaron entre ambas. READ UNCOMMITTED no ofrece un comportamiento distinto de READ COMMITTED; no permite leer cambios sucios.

REPEATABLE READ mantiene una instantánea estable para la transacción y evita también fantasmas en sus lecturas, aunque pueden existir anomalías de serialización. SERIALIZABLE añade controles para detectar ejecuciones incompatibles con un orden serial y puede abortar una transacción, que debe reintentarse completa.

```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- Operaciones de la transacción.
COMMIT;
```

Una respuesta de examen distingue la tabla del estándar del comportamiento del motor. No afirmar que «serializable» solo significa no leer datos sucios: esa propiedad ya la proporciona READ COMMITTED.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
