---
tags:
  - tp6
---

# Permisos para todos con PUBLIC

## Permisos para todos mediante PUBLIC

PUBLIC representa al conjunto de todos los usuarios, incluidos los que se creen más adelante. No hace falta asignar cada usuario a PUBLIC.

```sql
GRANT INSERT, UPDATE ON tarea TO PUBLIC;
```

Esta sentencia permite a todos insertar filas y modificar datos de tarea. No concede SELECT ni DELETE: PUBLIC identifica a los destinatarios, no significa «todos los privilegios». Las operaciones siguen sujetas a las restricciones de integridad y a los demás permisos necesarios; por ejemplo, leer columnas en un WHERE requiere SELECT.

Para retirar la concesión de inserción hecha a PUBLIC:

```sql
REVOKE INSERT ON tarea FROM PUBLIC CASCADE;
```

Se retira INSERT por ese camino, pero UPDATE permanece. Además, quien tenga INSERT por una concesión directa o por un rol conserva ese permiso independiente. Revocar a PUBLIC no equivale a prohibir la operación a todos: retira una fuente de autorización. El propietario conserva sus privilegios propios salvo que también los haya revocado expresamente; en PostgreSQL, un superusuario puede operar independientemente de estos permisos.

Por ejemplo:

```sql
GRANT INSERT ON tarea TO PUBLIC;
GRANT INSERT ON tarea TO ana;
REVOKE INSERT ON tarea FROM PUBLIC CASCADE;
```

Ana conserva INSERT por la concesión directa. Los usuarios cuyo único permiso de inserción provenía de PUBLIC lo pierden.

### PUBLIC y WITH GRANT OPTION: estándar y PostgreSQL

En el criterio del estándar SQL, conceder a PUBLIC con WITH GRANT OPTION permite a todos usar el privilegio y delegarlo; sin esa cláusula, la concesión a PUBLIC solo permite usarlo. Esto responde a la comparación solicitada por el ejercicio 2 del TP6.

PostgreSQL no admite conceder WITH GRANT OPTION a PUBLIC: para autorizar la delegación hay que otorgarla a usuarios o roles concretos. Esta diferencia debe explicitarse al pasar de la respuesta del práctico a una implementación en PostgreSQL.

## Material

Enunciado: Practicos/Practico 6 - Seguridad.docx, dentro de la carpeta Bases de Datos, fuera del baúl.

Referencia para los detalles indicados de PostgreSQL: [GRANT](https://www.postgresql.org/docs/current/sql-grant.html) y [privilegios](https://www.postgresql.org/docs/17/ddl-priv.html).
