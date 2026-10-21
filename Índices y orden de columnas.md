---
tags:
  - tp7
---

# Índices y orden de columnas


Ejemplos de propuestas, no instrucciones ejecutadas:

```sql
CREATE INDEX idx_historico_voluntario
ON historico (nro_voluntario);

CREATE INDEX idx_entrega_distribuidor_fecha
ON entrega (id_distribuidor, fecha_entrega);
```

Un índice puede acelerar búsqueda, join u ordenamiento, a cambio de espacio y costo de INSERT/UPDATE/DELETE. PK y UNIQUE normalmente crean índices propios en PostgreSQL; una FK no crea automáticamente un índice en la tabla que referencia.

### Índices multicolumna

En un B-tree `(id_empleado, cod_proyecto)`, las condiciones sobre la columna inicial suelen permitir acotar mejor la búsqueda. Para consultar por proyecto suele ser más conveniente `(cod_proyecto, id_empleado)`.

Ambas combinaciones pueden expresar la misma unicidad, pero el orden físico sirve a accesos distintos. Para el análisis pedido por la cátedra, justificar con el prefijo del índice y las condiciones de búsqueda, sin basar la respuesta en optimizaciones de versiones posteriores.

## Mismo conjunto de columnas, distinto orden de búsqueda

En `(id_empleado, cod_proyecto)`, las asociaciones de un empleado quedan próximas entre sí; sirve para buscar los proyectos del empleado 3. En `(cod_proyecto, id_empleado)`, quedan próximas las asociaciones del proyecto 2; sirve para buscar sus empleados. Ambas PK impiden repetir la misma pareja, pero sus índices ordenan primero por componentes diferentes.

Para `id_distribuidor = 20 AND fecha_entrega >= ...`, el índice `(id_distribuidor, fecha_entrega)` permite localizar primero el distribuidor y luego el tramo de fechas. Si la consulta solo filtra fecha, esa disposición ya no ofrece el mismo prefijo útil. Hay que relacionar el orden con las consultas frecuentes, no con el orden alfabético de los nombres.

## Selectividad y costo de mantenimiento

Si una condición deja 10 filas de 100000, elimina gran parte de la tabla y puede favorecer un acceso por índice. Si deja 90000, quizá convenga una lectura secuencial. Un atributo con dos valores no es selectivo automáticamente: depende de cuántas filas satisfacen el valor buscado.

Cada inserción, baja o cambio de clave indexada también debe mantener el índice. Para proponer uno en el ejercicio 5, indicar qué consulta lo aprovecha, por qué reduce trabajo y si el mismo prefijo sirve a otras. No crear un índice por cada columna sin analizar el conjunto.

## Fuente de estudio

Enunciado del TP7 en la carpeta `../Practicos` y filminas de cátedra `BDI_07 (Optimizacion).pdf` (carpeta de cursada 2023).
