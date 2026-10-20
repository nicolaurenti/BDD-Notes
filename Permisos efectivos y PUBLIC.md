---
tags:
  - tp6
---

# Permisos efectivos y PUBLIC

## Permisos efectivos: se suman

Un usuario puede acceder por concesiones directas, pertenencia a roles, PUBLIC y capacidades de propietario/administrador.

Revocar un camino no equivale a prohibir una acción si existe otro que la autoriza. `REVOKE` no es un DENY global.

Ejemplo: si U2 tiene SELECT directo y también por un rol, revocar el SELECT directo no elimina el del rol.

## PUBLIC

`PUBLIC` representa a todos los roles, incluidos los que se creen después.

```sql
GRANT INSERT, UPDATE ON TABLE tarea TO PUBLIC;
REVOKE INSERT ON TABLE tarea FROM PUBLIC;
```

Tras la revocación, pueden seguir insertando el propietario y quienes conserven permiso directo o mediante otros roles, además de administradores con capacidad suficiente. No responder «nadie» automáticamente.

En PostgreSQL no se concede WITH GRANT OPTION a PUBLIC. Distinguir esta limitación de la discusión teórica sobre permisos y delegación del estándar.
