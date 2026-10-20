---
tags:
  - tp2
---

# Consultas con “todos”

### Patrón de doble NOT EXISTS

**No existe un elemento requerido para el cual no exista una relación.**

Voluntarios que trabajaron históricamente en todas las instituciones:

```sql
SELECT v.nro_voluntario, v.nombre, v.apellido
FROM voluntario v
WHERE NOT EXISTS (
    SELECT 1
    FROM institucion i
    WHERE NOT EXISTS (
        SELECT 1
        FROM historico h
        WHERE h.nro_voluntario = v.nro_voluntario
          AND h.id_institucion = i.id_institucion
    )
);
```

Lectura:

> Seleccionar voluntarios para los que no exista una institución en la que no hayan trabajado.

> [!note]
> Si el conjunto de instituciones está vacío, la condición se cumple para todos los voluntarios. Si el enunciado exige otra interpretación, hay que agregar una comprobación de existencia.
