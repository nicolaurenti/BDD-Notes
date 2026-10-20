---
tags:
  - tp5
---

# Definición y uso de vistas


Una vista común guarda una consulta con nombre: al consultarla, se obtiene información de sus tablas base. No guarda por separado una copia de sus filas. Una vista materializada sí almacena un resultado y requiere actualizarlo con el mecanismo correspondiente.

Sirve para simplificar consultas, presentar un subconjunto de datos y ofrecer una interfaz de acceso. Para restringir acceso, combinarla con permisos sobre la vista y sobre las tablas base; ver TP6 - Seguridad.

```sql
CREATE VIEW envios500 AS
SELECT id_proveedor, id_articulo, cantidad
FROM envio
WHERE cantidad >= 500;

CREATE VIEW envios500_m AS
SELECT *
FROM envios500
WHERE cantidad < 1000;
```

La segunda vista devuelve cantidades entre 500 y 999 si son enteras. Se usa `_` en el nombre para evitar un identificador con guion que requiera comillas.
