---
tags:
  - tp6
---

# Usuarios, roles y administración de privilegios

La autenticación identifica quién se conecta. La autorización decide qué puede hacer sobre qué objeto.

En PostgreSQL, usuarios y grupos se representan mediante roles: un rol con LOGIN puede iniciar sesión; otro puede servir para agrupar permisos.

| Privilegio | Operación |
|---|---|
| SELECT | Consultar datos |
| INSERT | Insertar filas |
| UPDATE | Modificar datos |
| DELETE | Borrar filas |
| REFERENCES | Declarar referencias según el objeto |
| USAGE sobre esquema | Acceder a objetos del esquema, si además se tienen sus permisos |
| EXECUTE sobre función/procedimiento | Ejecutar el objeto |

El propietario controla su objeto. `ALL PRIVILEGES` se refiere a los privilegios aplicables a ese tipo de objeto: no convierte al receptor en propietario ni en administrador de todo el sistema.

## Leer un permiso como una relación concreta

En `GRANT SELECT ON instalacion TO adm`, el objeto es INSTALACION, la operación autorizada es SELECT y el receptor es adm. Saber que adm «tiene permisos» es insuficiente: podría consultar esa tabla y no poder insertar en ella ni consultar otra.

La cátedra distingue capacidades de cuenta, como administrar objetos, y privilegios sobre relaciones, como SELECT, INSERT, UPDATE y DELETE. También distingue control discrecional, donde se conceden privilegios sobre objetos, y control por roles, donde se agrupan privilegios y se asigna el grupo.

## Alcance de columna y de fila

`GRANT UPDATE(cantHoras, tarea) ON instalacion TO doc` permite actualizar esas columnas, no todas. No limita por sí mismo qué filas puede modificar doc. Para presentar solo ciertas filas o columnas se puede usar una vista, ya estudiada en TP5, y conceder permisos sobre ella.

ALL PRIVILEGES sobre INSTITUCION da los privilegios aplicables a esa tabla. No autoriza automáticamente a crear usuarios ni concede permisos sobre VOLUNTARIO. Tampoco implica WITH GRANT OPTION: usar los datos y transmitir el permiso son capacidades distintas.

Al resolver, tomar como punto inicial solo las concesiones y capacidades declaradas por el ejercicio. No suponer que un usuario ordinario es administrador porque su nombre sea adm o porque haya podido hacer SELECT.

## Creación y administración de roles

Ejemplo PostgreSQL sobre tablas ilustrativas `pais` y `provincia` del enunciado:

```sql
CREATE ROLE ins_prov;
GRANT INSERT ON TABLE pais, provincia TO ins_prov;

CREATE USER u3;
CREATE USER u4;
GRANT ins_prov TO u3, u4;

GRANT UPDATE (nombre) ON TABLE voluntario TO ins_prov;
```

La disponibilidad de esos permisos al iniciar sesión depende de la configuración de herencia de la membresía; `SET ROLE` permite actuar como un rol autorizado. Antes de usar nombres reales, verificar que no existan y que las tablas correspondan al esquema del ejercicio.

**WITH ADMIN OPTION** en membresías permite administrar la pertenencia a un rol; **WITH GRANT OPTION** permite delegar privilegios sobre objetos. No son lo mismo.

Eliminar un rol no elimina sus usuarios miembros. En PostgreSQL puede impedirlo la existencia de objetos propios o dependencias de privilegios que deben resolverse previamente. Como efecto conceptual de perder la membresía, los usuarios dejan de obtener los permisos que dependían exclusivamente de ese rol.

### Qué cambia cuando se modifica el rol

ins_prov agrupa INSERT sobre PAIS y PROVINCIA. Asignarlo a U3 y U4 evita repetir las concesiones sobre cada usuario. Si luego se concede UPDATE(nombre) de VOLUNTARIO al rol, ambos obtienen ese acceso por el mismo agrupamiento; no hace falta volver a asignar el rol.

```sql
GRANT ins_prov TO u3 WITH ADMIN OPTION;
```

Esta opción permite a U3 conceder el rol a otros. No es WGO de INSERT sobre PAIS: la primera administra membresías; la segunda transmite un privilegio de objeto. En el grafo, distinguir una arista de pertenencia al rol de una concesión sobre una tabla.

Quitar ins_prov no borra las cuentas U3 y U4. Dejan de recibir lo que dependía del rol, pero conservan permisos directos o procedentes de otros roles. Si se estudia la eliminación física en PostgreSQL, pueden existir dependencias que impidan retirar el rol hasta resolverlas; eso es una condición de ejecución, no una eliminación de sus usuarios.

En una respuesta SQL, indicar quién ejecuta las sentencias de creación y asignación. El ejercicio parte de U0 con autoridad administrativa; no atribuir a U2 esa autoridad por haber recibido SELECT sobre una tabla.

## Fuente de estudio

Enunciado del TP6 en la carpeta `../../Practicos` y filminas de cátedra `BDI_06 (Seguridad).pdf` (carpeta de cursada 2023).
