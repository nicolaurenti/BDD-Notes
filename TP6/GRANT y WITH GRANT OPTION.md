---
tags:
  - tp6
---

# GRANT y WITH GRANT OPTION


```sql
GRANT SELECT ON TABLE instalacion TO adm WITH GRANT OPTION;
GRANT UPDATE ON TABLE instalacion TO adm WITH GRANT OPTION;
GRANT DELETE ON TABLE instalacion TO adm;
```

`adm` puede consultar, actualizar y borrar. Puede delegar SELECT y UPDATE porque recibió grant option; no puede delegar DELETE basándose únicamente en esas concesiones.

```sql
-- Ejecutadas por adm con los permisos anteriores:
GRANT SELECT ON TABLE instalacion TO doc;
GRANT UPDATE (cantHoras, tarea)
ON TABLE instalacion TO doc WITH GRANT OPTION;
```

Se conserva `cantHoras`, como en el enunciado del ejercicio 1. El permiso UPDATE puede restringirse a columnas; no restringe filas. La concesión de SELECT no implica permiso INSERT, ni grant option para otro privilegio.

## El permiso de uso no basta para conceder

Después de las tres concesiones iniciales, adm tiene esta situación:

| Privilegio sobre INSTALACION | Puede ejercerlo | Puede transmitirlo por esta concesión |
|---|---|---|
| SELECT | Sí | Sí |
| UPDATE | Sí, toda la tabla | Sí |
| DELETE | Sí | No |

Por eso su intento `GRANT DELETE ON instalacion TO doc` no queda autorizado. El receptor no obtiene DELETE solo porque el otorgante sí pueda borrar. En cambio, adm puede transmitir UPDATE limitado a cantHoras y tarea, ya que ese subconjunto está contenido en el permiso amplio que recibió con delegación.

Si doc recibe UPDATE de esas dos columnas con WGO, puede transmitir ese subconjunto; no puede agregar otra columna que nunca recibió con delegación. Tampoco puede transmitir SELECT si lo recibió sin WGO, aunque sí pueda consultar.

Para autorizar a U2 a conceder INSERT a U3, hay que darle a U2 **INSERT con WGO**. Haberle dado SELECT con WGO no alcanza: la opción acompaña a un privilegio concreto, no habilita la delegación general de cualquier operación.

## Fuente de estudio

Enunciado del TP6 en la carpeta `../../Practicos` y filminas de cátedra `BDI_06 (Seguridad).pdf` (carpeta de cursada 2023).
