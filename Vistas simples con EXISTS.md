---
tags:
  - tp5
---

# Vistas simples con EXISTS


Cuando solo se necesita filtrar entidades por relaciones, una subconsulta evita multiplicarlas y puede conservar un FROM simple:

```sql
CREATE VIEW distribuidores_con_entregas AS
SELECT d.*
FROM distribuidor d
WHERE EXISTS (
    SELECT 1
    FROM entrega e
    WHERE e.id_distribuidor = d.id_distribuidor
);
```

Esto es distinto de proyectar un join y eliminar duplicados con DISTINCT. Para «departamentos con más de 2 empleados», agrupar en la subconsulta por la clave completa de departamento.
