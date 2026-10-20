---
tags:
  - tp6
---

# GRANT y WITH GRANT OPTION


```sql
GRANT SELECT ON TABLE instalacion TO adm WITH GRANT OPTION;
GRANT UPDATE ON TABLE instalacion TO adm WITH GRANT OPTION;
GRANT DELETE ON TABLE instalacion TO adm;
```

`adm` puede consultar, actualizar y borrar. Puede delegar SELECT y UPDATE porque recibió grant option; no puede delegar DELETE basándose únicamente en esas concesiones.

```sql
-- Ejecutadas por adm con los permisos anteriores:
GRANT SELECT ON TABLE instalacion TO doc;
GRANT UPDATE (cant_horas, tarea)
ON TABLE instalacion TO doc WITH GRANT OPTION;
```

Se usan nombres ilustrativos `cant_horas` y `tarea`; adaptar a las columnas reales. El permiso UPDATE puede restringirse a columnas; no restringe filas. La concesión de SELECT no implica permiso INSERT, ni grant option para otro privilegio. [GRANT en PostgreSQL](https://www.postgresql.org/docs/current/sql-grant.html).
