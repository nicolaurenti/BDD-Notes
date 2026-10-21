---
tags:
  - tp6
---

# Usuarios, roles, objetos y privilegios


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

## Fuente de estudio

Enunciado del TP6 en la carpeta `../Practicos` y filminas de cátedra `BDI_06 (Seguridad).pdf` (carpeta de cursada 2023).
