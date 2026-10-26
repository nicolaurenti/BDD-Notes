---
tags:
  - tp6
---

# Privilegios y GRANT

## Privilegios sobre tablas

Un privilegio es un permiso para realizar una operación sobre un objeto. Los principales para este práctico son:

| Privilegio | Qué permite |
|---|---|
| SELECT | Consultar datos. |
| INSERT | Agregar filas. |
| UPDATE | Modificar datos de filas existentes. |
| DELETE | Eliminar filas. |

Los privilegios son independientes: poder consultar una tabla no implica poder modificarla. Se analizan por usuario, objeto y operación. Por ejemplo, un usuario puede consultar una tabla, modificar otra y no tener acceso a una tercera.

ALL PRIVILEGES permite conceder todos los privilegios aplicables al objeto que quien concede esté autorizado a otorgar. No convierte al destinatario en administrador ni le da automáticamente capacidad para delegar esos privilegios.

## Otorgar permisos con GRANT

La estructura básica es:

```sql
GRANT privilegio ON tabla TO usuario;
```

Por ejemplo:

```sql
GRANT SELECT ON voluntario TO ana;
GRANT INSERT ON voluntario TO ana;
```

Ana puede consultar y agregar filas. Las concesiones se acumulan: la segunda no reemplaza la primera. También pueden expresarse juntas:

```sql
GRANT SELECT, INSERT ON voluntario TO ana;
```

Para conceder todos los privilegios que el otorgante puede conceder sobre esa tabla:

```sql
GRANT ALL PRIVILEGES ON voluntario TO ana;
```

Puede otorgar un privilegio el propietario o quien tenga autorización para delegarlo. Tener permiso para operar no basta para concedérselo a otro usuario.

### Permisos sobre columnas específicas

En PostgreSQL, SELECT, INSERT y UPDATE pueden limitarse a determinadas columnas. Se escriben entre paréntesis después del privilegio:

```sql
GRANT SELECT(nombre) ON voluntario TO ana;
GRANT INSERT(nombre) ON voluntario TO ana;
GRANT UPDATE(nombre) ON voluntario TO ana;
```

Estas sentencias permiten consultar nombre, asignarle un valor al insertar una fila y modificarlo, respectivamente. No conceden esos privilegios sobre las demás columnas.

INSERT(nombre) sigue creando una fila completa: las columnas omitidas reciben su valor predeterminado o NULL. Si alguna columna obligatoria no tiene un valor predeterminado adecuado, la inserción falla. Tener el privilegio no evita las restricciones de integridad.

DELETE no admite una lista de columnas porque elimina filas completas:

```sql
GRANT DELETE ON voluntario TO ana;
```

Quitar el valor de una columna es una modificación con UPDATE, por ejemplo asignándole NULL si las restricciones lo permiten; no es un DELETE sobre esa columna.

Se pueden combinar privilegios con alcances diferentes:

```sql
GRANT SELECT(nombre, apellido), UPDATE(nombre)
ON voluntario TO ana;
```

Ana puede consultar nombre y apellido, pero modificar solamente nombre. Si un UPDATE utiliza otra columna en el WHERE, necesita SELECT sobre esa columna para leerla. Por ejemplo, para filtrar por nro_voluntario necesita SELECT sobre nro_voluntario, además de UPDATE sobre la columna que modifica.

Estos ejemplos suponen que Ana no tiene permisos más amplios por otras concesiones, roles o PUBLIC. Un GRANT limitado no restringe permisos que ya tenía.

## Material

Enunciado: Practicos/Practico 6 - Seguridad.docx, dentro de la carpeta Bases de Datos, fuera del baúl.

Referencia para los detalles indicados de PostgreSQL: [GRANT](https://www.postgresql.org/docs/current/sql-grant.html) y [privilegios](https://www.postgresql.org/docs/17/ddl-priv.html).
