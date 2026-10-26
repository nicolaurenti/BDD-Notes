---
tags:
  - tp6
---

# Usuarios y roles

## Usuarios y roles

Un usuario identifica a quien accede al sistema. Un rol permite reunir privilegios y asignarlos como conjunto: por ejemplo, un rol de carga de datos puede tener INSERT sobre varias tablas. Así se evita repetir concesiones para cada usuario y se administran los permisos de forma centralizada.

### Crear un rol, darle permisos y asignarlo

```sql
CREATE ROLE carga_datos;
GRANT INSERT ON pais TO carga_datos;
GRANT INSERT ON provincia TO carga_datos;
```

Crear el rol no le da automáticamente permisos sobre las tablas. Las sentencias GRANT son las que le conceden INSERT.

Para asignarlo a usuarios ya existentes:

```sql
GRANT carga_datos TO ana, juan;
```

Aquí no aparece ON: se concede pertenencia al rol. En cambio, GRANT INSERT ON pais TO carga_datos concede un privilegio sobre un objeto.

En el ejemplo suponemos que los usuarios heredan los privilegios del rol. En PostgreSQL esto depende de la configuración de herencia de la pertenencia; también puede utilizarse SET ROLE si está autorizado. No hay que confundir ser miembro con poder delegar permisos.

Si después se agrega un permiso al rol:

```sql
GRANT UPDATE(nombre) ON voluntario TO carga_datos;
```

Sus miembros pueden utilizar también ese permiso bajo el supuesto de herencia anterior. No hace falta repetirlo para cada usuario.

### Crear usuarios: detalle de PostgreSQL

```sql
CREATE USER ana;
CREATE USER juan;
```

En PostgreSQL, un usuario es un rol con capacidad de iniciar sesión (LOGIN). CREATE USER supone LOGIN por defecto; CREATE ROLE supone NOLOGIN. Estas sentencias por sí solas no completan necesariamente la configuración de autenticación y conexión. CREATE USER es una forma de PostgreSQL: el estándar SQL deja la creación de usuarios a cada implementación.

### Retirar un permiso del rol o retirar una pertenencia

```sql
REVOKE INSERT ON pais FROM carga_datos CASCADE;
```

Retira al rol el permiso de inserción sobre pais; sus miembros dejan de obtenerlo por ese camino. En cambio:

```sql
REVOKE carga_datos FROM ana;
```

Retira a Ana la pertenencia al rol. Juan sigue siendo miembro. Ana conserva los privilegios recibidos por concesiones directas, otros roles o PUBLIC. En PostgreSQL, según las dependencias de pertenencia, puede ser necesario especificar CASCADE o RESTRICT.

### Eliminar un rol

La sentencia que expresa la eliminación es:

```sql
DROP ROLE carga_datos;
```

Si la eliminación se completa, Ana y Juan siguen existiendo: pierden la pertenencia al rol y los privilegios que obtenían exclusivamente por él. Sus otros permisos se conservan.

En PostgreSQL, DROP ROLE falla si el rol todavía tiene privilegios concedidos sobre objetos o es propietario de objetos. Antes deben retirarse esas concesiones y resolverse las dependencias de propiedad preservando los objetos. Las pertenencias, en cambio, se retiran automáticamente al eliminar el rol. Por eso no debe suponerse que el DROP ROLE del ejercicio funciona inmediatamente después de concederle permisos si se implementa en PostgreSQL.

### Delegación de pertenencia y delegación de privilegios

WITH ADMIN OPTION permite delegar la pertenencia a un rol:

```sql
GRANT carga_datos TO ana WITH ADMIN OPTION;
```

Ana puede asignar carga_datos a otros usuarios. WITH GRANT OPTION, por su parte, autoriza a delegar un privilegio sobre un objeto, como SELECT sobre una tabla. Son autorizaciones diferentes.

Referencias de PostgreSQL: [CREATE ROLE](https://www.postgresql.org/docs/current/sql-createrole.html), [CREATE USER](https://www.postgresql.org/docs/current/sql-createuser.html), [DROP ROLE](https://www.postgresql.org/docs/current/sql-droprole.html) y GRANT, enlazado en Material.

## Material

Enunciado: Practicos/Practico 6 - Seguridad.docx, dentro de la carpeta Bases de Datos, fuera del baúl.

Referencia para los detalles indicados de PostgreSQL: [GRANT](https://www.postgresql.org/docs/current/sql-grant.html) y [privilegios](https://www.postgresql.org/docs/17/ddl-priv.html).
