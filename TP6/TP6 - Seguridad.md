---
tags:
  - tp6
---

# TP6 - Seguridad

La seguridad de una base de datos permite definir qué operaciones puede realizar cada usuario sobre sus objetos. Este apunte reúne las explicaciones y dudas trabajadas en la conversación, con ejemplos ilustrativos que no se ejecutaron sobre la base.

## Subtemas

- [[Privilegios y GRANT]] — Operaciones permitidas, concesiones y permisos sobre columnas.
- [[Delegación y grafos de permisos]] — WITH GRANT OPTION, caminos de autorización y acumulación de permisos.
- [[Revocación de permisos con REVOKE]] — CASCADE, RESTRICT y retiro de la opción de delegación.
- [[Permisos para todos con PUBLIC]] — Concesiones generales y diferencias entre el estándar SQL y PostgreSQL.
- [[Usuarios y roles]] — Creación, asignación, administración y eliminación de roles.
- [[Análisis de secuencias y permisos finales]] — Un ejemplo completo para seguir las operaciones y obtener el estado final.

## Material

Enunciado: Practicos/Practico 6 - Seguridad.docx, dentro de la carpeta Bases de Datos, fuera del baúl.

Referencia para los detalles indicados de PostgreSQL: [GRANT](https://www.postgresql.org/docs/current/sql-grant.html) y [privilegios](https://www.postgresql.org/docs/17/ddl-priv.html).
