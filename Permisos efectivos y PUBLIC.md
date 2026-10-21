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

## Seguir una revocación a PUBLIC

Supongamos que todos reciben INSERT y UPDATE sobre TAREA por PUBLIC, y U1 además tiene INSERT directo. Al retirar INSERT de PUBLIC, un usuario U2 que dependía solo de PUBLIC lo pierde; U1 sigue insertando por su concesión directa; UPDATE no se ve afectado. El propietario conserva su capacidad sobre el objeto.

PUBLIC no es una lista congelada de los usuarios actuales: representa a todos, también a los futuros. Por eso no hace falta repetir la concesión por cada nueva cuenta.

## La pregunta sobre delegación

El ejercicio pide discutir la diferencia de incluir WGO: conceptualmente, además de ejercer el privilegio, el receptor podría transmitirlo y se generarían concesiones dependientes. Sin WGO, recibir por PUBLIC no da por sí solo autoridad para concederlo. Esa discusión debe separarse de si el motor admite la combinación escrita: PostgreSQL no permite conceder WGO a PUBLIC.

Para hallar el permiso efectivo de una persona, reunir concesiones directas, roles y PUBLIC. Para decidir si puede **conceder**, reunir solo las fuentes que también dan autoridad de delegación. Tener tres caminos de SELECT sin WGO no equivale a tener uno con WGO.

## Fuente de estudio

Enunciado del TP6 en la carpeta `../Practicos` y filminas de cátedra `BDI_06 (Seguridad).pdf` (carpeta de cursada 2023).
