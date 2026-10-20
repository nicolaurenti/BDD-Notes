---
tags:
  - tp1
---

# Actualización de filas con UPDATE


Se utiliza `UPDATE`.

### Actualizar una fila

Se filtra por una clave que identifique la fila:

```sql
UPDATE articulo
SET autor = 'Juan'
WHERE id_articulo = 1;
```

### Actualizar un subconjunto de filas

```sql
UPDATE articulo
SET autor = 'María'
WHERE autor = 'Ana';
```

### Actualizar todas las filas

```sql
UPDATE articulo
SET autor = 'Autor pendiente';
```

> [!warning]
> Un `UPDATE` sin `WHERE` afecta a todas las filas de la tabla.
