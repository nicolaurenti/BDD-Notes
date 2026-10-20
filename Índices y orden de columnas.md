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

Ambas combinaciones pueden expresar la misma unicidad, pero el orden físico sirve a accesos distintos. No afirmar que una columna posterior nunca puede aprovecharse: el optimizador puede elegir otras estrategias, incluido skip scan en versiones que lo soportan. [Índices multicolumna](https://www.postgresql.org/docs/current/indexes-multicolumn.html).
