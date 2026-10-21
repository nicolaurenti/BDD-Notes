---
tags:
  - tp2
---

# TP2 - Comparaciones, negación y COALESCE con NULL

Casos de negación, sustitución y comparación de valores ausentes.

### Negar una condición no incluye automáticamente los nulos

```sql
WHERE NOT (porcentaje BETWEEN 0.15 AND 0.30)
```

Esta condición tampoco selecciona filas cuyo porcentaje sea `NULL`.

Para incluirlas:

```sql
WHERE porcentaje NOT BETWEEN 0.15 AND 0.30
   OR porcentaje IS NULL
```

### Reemplazar un NULL con COALESCE

Devuelve el primer argumento no nulo:

```sql
SELECT COALESCE(telefono, 'Sin teléfono')
FROM voluntario;
```

### Comparaciones y negación

`NULL` no equivale a `0` ni a una cadena vacía (`''`). Incluso comparar dos nulos devuelve desconocido:

| Expresión | Resultado |
|---|---|
| `NULL = NULL` | `UNKNOWN` |
| `NULL <> NULL` | `UNKNOWN` |
| `NOT UNKNOWN` | `UNKNOWN` |
| `NULL IS NULL` | `TRUE` |
| `NULL IS NOT NULL` | `FALSE` |

Una condición desconocida no es falsa, pero tampoco permite que la fila pase un `WHERE`. En un `INNER JOIN`, `ON` también exige `TRUE` para formar una pareja.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
