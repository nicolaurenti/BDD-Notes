---
tags:
  - tp5
---

# WITH CHECK OPTION y migración de filas


```sql
CREATE VIEW envios500_controlados AS
SELECT id_proveedor, id_articulo, cantidad
FROM envio
WHERE cantidad >= 500
WITH CHECK OPTION;
```

Sin chequeo, insertar una cantidad 300 puede modificar la tabla base y, aun así, no aparecer en la vista. Actualizar 700 a 100 puede hacer que una fila desaparezca de la vista.

Con chequeo, INSERT y UPDATE sobre esa vista deben dejar las filas visibles según las condiciones exigidas. No vuelve actualizable una vista que no lo era; tampoco sustituye las restricciones de las tablas.

| Operación sobre ENVIOS500 | Sin WCO | Con WCO |
|---|---|---|
| Insertar 500 | Pasa el filtro | Pasa el filtro |
| Insertar 300 | Puede insertarse y quedar oculta | Se rechaza |
| Cambiar una fila visible a 100 | Puede migrar fuera | Se rechaza |
| Cambiar una fila visible a 1000 | Pasa el filtro | Pasa el filtro |
| Referenciar artículo inexistente | La FK puede rechazar | La FK puede rechazar igualmente |

Solo se actualizan filas actualmente visibles. Si un INSERT de 300 quedó oculto, un UPDATE posterior sobre la vista no lo encuentra, aunque filtre por su proveedor. WCO no impide escrituras directas sobre la tabla base que dejen filas fuera de la vista.
