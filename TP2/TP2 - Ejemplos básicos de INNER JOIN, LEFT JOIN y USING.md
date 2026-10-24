---
tags:
  - tp2
---

# TP2 - Ejemplos básicos de INNER JOIN, LEFT JOIN y USING

Consultas básicas de instituciones y voluntarios para contrastar coincidencias y filas preservadas.

#### INNER JOIN

```sql
SELECT i.nombre_institucion, v.nombre, v.apellido
FROM institucion i
JOIN voluntario v
    ON v.id_institucion = i.id_institucion;
```

Solo aparecen instituciones con voluntarios relacionados.

#### LEFT JOIN

```sql
SELECT i.nombre_institucion, v.nombre, v.apellido
FROM institucion i
LEFT JOIN voluntario v
    ON v.id_institucion = i.id_institucion;
```

También aparecen las instituciones sin voluntarios.

#### JOIN USING

Cuando las columnas de unión tienen el mismo nombre:

```sql
FROM voluntario v
JOIN institucion i USING (id_institucion)
```

> [!warning]
> `NATURAL JOIN` utiliza todas las columnas con nombres coincidentes. Si cambia la estructura de las tablas, puede cambiar también el significado del join.

## Fuente de estudio

Enunciado del TP2 en la carpeta `../../Practicos` y filminas de cátedra `BDI_02.P1 (Consultas–1).pdf; BDI_02.P2 (Consultas–2).pdf` (carpeta de cursada 2023).
