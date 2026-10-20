---
tags:
  - tp1
---

# Inserción de filas con INSERT


Se utiliza `INSERT INTO`.

```sql
INSERT INTO articulo (
    id_articulo,
    titulo,
    autor,
    fecha_pub
)
VALUES (
    1,
    'Bases de datos',
    'Ana',
    DATE '2026-10-18'
);
```

También pueden insertarse varias filas:

```sql
INSERT INTO palabra (cod_p, idioma, descripcion)
VALUES
    (1, 'ES', 'Datos'),
    (2, 'ES', 'Tablas'),
    (1, 'EN', 'Data');
```

Las filas con código `1` son válidas porque la PK incluye también el idioma.
