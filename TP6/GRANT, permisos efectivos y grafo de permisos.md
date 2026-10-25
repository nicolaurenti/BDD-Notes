---
tags:
  - tp6
---

# GRANT, permisos efectivos y grafo de permisos

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

## Permisos efectivos y PUBLIC

### Permisos efectivos: se suman

Un usuario puede acceder por concesiones directas, pertenencia a roles, PUBLIC y capacidades de propietario/administrador.

Revocar un camino no equivale a prohibir una acción si existe otro que la autoriza. `REVOKE` no es un DENY global.

Ejemplo: si U2 tiene SELECT directo y también por un rol, revocar el SELECT directo no elimina el del rol.

### PUBLIC

`PUBLIC` representa a todos los roles, incluidos los que se creen después.

```sql
GRANT INSERT, UPDATE ON TABLE tarea TO PUBLIC;
REVOKE INSERT ON TABLE tarea FROM PUBLIC;
```

Tras la revocación, pueden seguir insertando el propietario y quienes conserven permiso directo o mediante otros roles, además de administradores con capacidad suficiente. No responder «nadie» automáticamente.

En PostgreSQL no se concede WITH GRANT OPTION a PUBLIC. Distinguir esta limitación de la discusión teórica sobre permisos y delegación del estándar.

### Seguir una revocación a PUBLIC

Supongamos que todos reciben INSERT y UPDATE sobre TAREA por PUBLIC, y U1 además tiene INSERT directo. Al retirar INSERT de PUBLIC, un usuario U2 que dependía solo de PUBLIC lo pierde; U1 sigue insertando por su concesión directa; UPDATE no se ve afectado. El propietario conserva su capacidad sobre el objeto.

PUBLIC no es una lista congelada de los usuarios actuales: representa a todos, también a los futuros. Por eso no hace falta repetir la concesión por cada nueva cuenta.

### La pregunta sobre delegación

El ejercicio pide discutir la diferencia de incluir WGO: conceptualmente, además de ejercer el privilegio, el receptor podría transmitirlo y se generarían concesiones dependientes. Sin WGO, recibir por PUBLIC no da por sí solo autoridad para concederlo. Esa discusión debe separarse de si el motor admite la combinación escrita: PostgreSQL no permite conceder WGO a PUBLIC.

Para hallar el permiso efectivo de una persona, reunir concesiones directas, roles y PUBLIC. Para decidir si puede **conceder**, reunir solo las fuentes que también dan autoridad de delegación. Tener tres caminos de SELECT sin WGO no equivale a tener uno con WGO.

## Grafo de permisos

Representar una arista por concesión con: otorgante, receptor, objeto, privilegio, columnas y si puede delegarse.

```text
Propietario -- SELECT + delegación --> adm -- SELECT --> doc
Propietario -- UPDATE + delegación --> adm -- UPDATE de columnas + delegación --> doc
Propietario -- DELETE -------------> adm
```

Si `adm` intenta `GRANT DELETE ... TO doc`, esa arista no queda autorizada por el DELETE recibido sin grant option. Antes de analizar una revocación, eliminar del grafo las concesiones que nunca fueron válidas.

Para cada sentencia preguntar: ¿el otorgante tiene autoridad para conceder ese privilegio y esas columnas? ¿La tiene por algún otro camino?

### Construirlo sentencia por sentencia

Dibujar un grafo separado por privilegio o rotular claramente cada arista. En cada paso, comprobar primero la autoridad del otorgante. Si la sentencia no está autorizada, no se agrega una arista y las sentencias siguientes no pueden apoyarse en ella.

Ejemplo con SELECT sobre T:

```text
A -- SELECT con WGO --> B -- SELECT --> D
A -- SELECT ---------> D
```

D tiene dos concesiones independientes: una de B y otra de A. Si A retira a B SELECT con CASCADE, desaparecen A→B y la concesión dependiente B→D, pero queda A→D. D sigue consultando.

La tabla auxiliar ayuda a no perder información:

| Otorgante | Receptor | Objeto | Privilegio | Columnas | WGO |
|---|---|---|---|---|---|
| A | B | T | SELECT | Todas | Sí |
| B | D | T | SELECT | Todas | No |
| A | D | T | SELECT | Todas | No |

No fusionar las dos entradas de D en una sola que diga «D tiene SELECT»: para calcular una revocación hace falta saber **de dónde lo obtuvo**. El grafo representa concesiones, no solo el resultado final de acceso.

## Fuente de estudio

Enunciado del TP6 en la carpeta `../../Practicos` y filminas de cátedra `BDI_06 (Seguridad).pdf` (carpeta de cursada 2023).
