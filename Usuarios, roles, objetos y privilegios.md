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
