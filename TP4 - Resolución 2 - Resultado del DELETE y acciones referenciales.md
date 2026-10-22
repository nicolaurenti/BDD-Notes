---
tags:
  - tp4
---

# TP4 - Resolución 2 - Resultado del DELETE y acciones referenciales

La operación pedida es:

```sql
DELETE FROM entrega WHERE id_video = 3582;
```

Primero se analiza la FK de RENGLON_ENTREGA hacia ENTREGA. La auditoría no elimina ni reemplaza esa restricción.

## Con el SQL local proporcionado

En `catedra/peliculas.sql`, `fk_re_entrega` no declara ON DELETE CASCADE: queda NO ACTION. Si alguna entrega seleccionada tiene renglones que la referencian, el DELETE se rechaza. ENTREGA y RENGLON_ENTREGA quedan sin cambios; tampoco quedan nuevas filas de auditoría de la sentencia fallida.

Si se seleccionan entregas sin renglones, sí pueden borrarse: ROW registra una fila por entrega y STATEMENT una entrada con su cantidad. Si no se selecciona ninguna, ROW registra cero entradas y STATEMENT una con cantidad cero.

## Si la instancia del ejercicio declara CASCADE

Si la FK permite propagar la baja, se borran las entregas seleccionadas y sus renglones. Con M entregas y R renglones, ROW registra M + R entradas, cada una con cantidad 1.

En la prueba controlada con dos entregas y tres renglones asociados, la alternativa STATEMENT de esta resolución produjo dos entradas: ENTREGA/DELETE/cantidad 2 y RENGLON_ENTREGA/DELETE/cantidad 3. Se verificó el resultado ejecutándolo; no se dedujo que bastara cambiar FOR EACH ROW por STATEMENT.

[[TP4 - Resolución 2 - Ejemplo de tablas antes y después|Ejemplo reproducible: datos iniciales, tablas finales y auditoría]].

No se cambió la FK de tus tablas. El caso con cascada se probó en otro esquema temporal cuya definición ya la incluía.
