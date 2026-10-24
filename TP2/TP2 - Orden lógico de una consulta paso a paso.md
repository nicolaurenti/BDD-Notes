---
tags:
  - tp2
---

# TP2 - Orden lógico de una consulta paso a paso

Seguí las etapas de la consulta y después comprobá sus efectos con cuatro filas.

## Orden lógico de ejecución

El orden lógico para interpretar una consulta es:

1. `FROM` y `JOIN`: construir las filas de origen.
2. `WHERE`: filtrar filas.
3. `GROUP BY`: formar grupos.
4. Calcular las agregaciones.
5. `HAVING`: filtrar grupos.
6. `SELECT`: producir las columnas del resultado.
7. `DISTINCT`: eliminar filas repetidas.
8. `ORDER BY`: ordenar.
9. `OFFSET` y `LIMIT`: seleccionar la porción del resultado.

> [!important]
> Este es el orden lógico. El motor puede utilizar otro orden físico para ejecutar la consulta de manera más eficiente.

#### Consecuencias

- `WHERE` no puede filtrar por una agregación calculada en ese mismo nivel.
- Los alias definidos en `SELECT` pueden usarse en `ORDER BY`, pero no en `WHERE` ni en `HAVING` en PostgreSQL.

```sql
SELECT id_institucion, COUNT(*) AS cantidad
FROM voluntario
GROUP BY id_institucion
HAVING COUNT(*) > 10
ORDER BY cantidad DESC;
```

## Seguir una consulta con datos pequeños

Supongamos estos voluntarios, mostrados como `(nro, institución, horas)`: `(1, 20, 500)`, `(2, 20, 2000)`, `(3, 20, 3000)`, `(4, 50, 4000)`.

```sql
SELECT id_institucion, COUNT(*) AS cantidad
FROM voluntario
WHERE horas_aportadas > 1000
GROUP BY id_institucion
HAVING COUNT(*) >= 2
ORDER BY id_institucion;
```

WHERE elimina al voluntario 1. Quedan dos filas de la institución 20 y una de la 50. GROUP BY forma esos dos grupos; COUNT produce 2 y 1; HAVING conserva solo el grupo de 20. El resultado es una fila `(20, 2)`.

Si se quitara WHERE, la cantidad para 20 sería 3. Si se cambiara HAVING por `WHERE COUNT(*) >= 2`, la consulta sería incorrecta: WHERE decide sobre filas antes de formar los grupos. Escribir primero qué representa una fila de entrada y qué debe representar una fila de salida ayuda a elegir las cláusulas.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
