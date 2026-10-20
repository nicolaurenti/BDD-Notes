---
tags:
  - tp6
---

# Creación y administración de roles


Ejemplo PostgreSQL sobre tablas ilustrativas `pais` y `provincia` del enunciado:

```sql
CREATE ROLE ins_prov NOLOGIN;
GRANT INSERT ON TABLE pais, provincia TO ins_prov;

CREATE ROLE u3 LOGIN;
CREATE ROLE u4 LOGIN;
GRANT ins_prov TO u3, u4;

GRANT UPDATE (nombre) ON TABLE voluntario TO ins_prov;
```

La disponibilidad de esos permisos al iniciar sesión depende de la configuración de herencia de la membresía; `SET ROLE` permite actuar como un rol autorizado. Antes de usar nombres reales, verificar que no existan y que las tablas correspondan al esquema del ejercicio.

**WITH ADMIN OPTION** en membresías permite administrar la pertenencia a un rol; **WITH GRANT OPTION** permite delegar privilegios sobre objetos. No son lo mismo.

Eliminar un rol no elimina sus usuarios miembros. En PostgreSQL puede impedirlo la existencia de objetos propios o dependencias de privilegios que deben resolverse previamente. Como efecto conceptual de perder la membresía, los usuarios dejan de obtener los permisos que dependían exclusivamente de ese rol.
