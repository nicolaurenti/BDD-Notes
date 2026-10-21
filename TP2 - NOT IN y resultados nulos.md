---
tags:
  - tp2
---

# TP2 - NOT IN y resultados nulos

Casos con 5, 7 y NULL para interpretar cada resultado.

## Riesgo de NOT IN con NULL

```sql
WHERE nro_voluntario NOT IN (
    SELECT id_coordinador
    FROM voluntario
)
```

Si la subconsulta devuelve algún `NULL`, puede impedir que se seleccionen filas que se esperaban obtener.

Alternativas:

- Excluir los nulos de la subconsulta.
- Expresar la ausencia mediante `NOT EXISTS`.

## Ejemplo de NOT IN con nulos

Si `SELECT DISTINCT puntos FROM equipo` devuelve una fila con `NULL`, para una fila exterior con puntos nulos:

```sql
NULL IN (NULL)     -- UNKNOWN
NULL NOT IN (NULL) -- UNKNOWN
```

`NOT` no transforma lo desconocido en verdadero. Por eso el `WHERE` no conserva esa fila. Tampoco se trata de que dos valores nulos sean iguales.

Para un valor no nulo, `5 NOT IN (5, NULL)` es `FALSE`, mientras que `7 NOT IN (5, NULL)` es `UNKNOWN`: el nulo impide confirmar que no hay coincidencia.

Filtrar nulos de la subconsulta o usar `NOT EXISTS` requiere decidir también qué se pretende hacer con los nulos de la consulta exterior. No son reemplazos equivalentes automáticamente en presencia de nulos.

El bloque de NULL del TP2 desarrolla la lógica de tres valores.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
