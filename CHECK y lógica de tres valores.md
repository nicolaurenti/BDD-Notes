---
tags:
  - tp3
---

# CHECK y lógica de tres valores


```sql
ALTER TABLE articulo
ADD CONSTRAINT ck_nacionalidad
CHECK (nacionalidad IN ('Argentina', 'Español', 'Inglesa', 'Alemana', 'Chilena'));

ALTER TABLE articulo
ADD CONSTRAINT ck_fecha_publicacion
CHECK (fecha_pub >= DATE '2010-01-01');

ALTER TABLE producto
ADD CONSTRAINT ck_descripcion_presentacion
CHECK (descripcion IS NOT NULL OR presentacion IS NOT NULL);
```

Un `CHECK` se incumple cuando da **FALSE**. Si da `UNKNOWN` por un nulo, no lo rechaza: agregar `NOT NULL` cuando la información sea obligatoria. Diferencia importante con `WHERE`, que solo conserva `TRUE`.

Una implicación «si A, entonces B» se expresa como `NOT A OR B`:

```sql
-- Después de 2020: desde 2021 inclusive.
CHECK (fecha_pub < DATE '2021-01-01' OR nacionalidad = 'Argentina')
```

Si la nacionalidad debe estar informada siempre, agregar `NOT NULL`. Si solo es obligatoria para artículos posteriores a 2020, la segunda rama debe incluir `nacionalidad IS NOT NULL`.

Para un prefijo literal `S_`:

```sql
CHECK (cod_suc LIKE 'S!_%' ESCAPE '!')
```

El `_` sin escape significa «cualquier carácter». Ver Manejo de strings.
