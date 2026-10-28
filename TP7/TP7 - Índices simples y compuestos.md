---
tags:
  - tp7
---

# 6. Índices simples y compuestos

## Idea central

Un índice es una estructura adicional para localizar filas sin recorrer necesariamente toda la tabla. Ocupa espacio y cuesta mantenerlo al insertar, borrar o modificar datos. Su utilidad depende de la carga de consultas y de la fracción de filas que evita examinar.

## Cómo razonarlo

Un índice simple usa una columna; uno compuesto usa varias. En un árbol B+ compuesto, las entradas se ordenan por la primera columna y, dentro de ella, por las siguientes: (a,b) no organiza los datos igual que (b,a).

Las condiciones de igualdad sobre el comienzo de la clave y el primer rango posterior suelen delimitar mejor la búsqueda. Condiciones posteriores aún pueden filtrarse en el índice, aunque no reduzcan del mismo modo el tramo recorrido. Este es el criterio básico del curso; no afirmar que una columna no inicial sea absolutamente inutilizable. Las capacidades concretas dependen del motor y la versión.

En PostgreSQL, PK y UNIQUE crean índices automáticamente; las FK del lado que referencia no reciben automáticamente uno. Cambiar el orden de una PK compuesta conserva las combinaciones únicas permitidas, pero cambia su índice de soporte.

## Ejemplo

```sql
CREATE INDEX idx_trabaja_proyecto_empleado
ON trabaja_en (cod_proyecto, id_empleado);
```

Para localizar empleados de un proyecto, el primer componente agrupa las participaciones de ese proyecto. Un índice (id_empleado,cod_proyecto) favorece la dirección inversa: proyectos de un empleado. Si esa dirección ya la cubre la PK, evaluar el índice complementario en lugar de duplicarla.

Para nombre por prefijo y correo por igualdad, `(e_mail,nombre)` permite ubicar primero el correo y después el rango de nombres. Si el correo ya es único, el índice simple existente podría localizar como máximo una fila y volver innecesario el compuesto para estas consultas.

## Errores frecuentes

No crear un índice por cada columna sin revisar los ya existentes. Muchos valores distintos no bastan para garantizar beneficio: `id > 50` puede devolver casi toda la tabla. Tampoco se debe elegir siempre «la columna más selectiva primero» ignorando igualdad, rangos y las otras consultas que compartirán el índice.

## Qué recordar

Justificar columnas, orden, consultas beneficiadas y mantenimiento. Sin estadísticas no hay una elección universalmente óptima. [Índices compuestos](https://www.postgresql.org/docs/current/indexes-multicolumn.html).
