---
tags:
  - tp6
---

# Análisis de secuencias y permisos finales

## Analizar secuencias y permisos finales

Los ejercicios se resuelven en el orden indicado: cada sentencia parte del estado dejado por las anteriores. Antes de actualizar el grafo hay que comprobar quién ejecuta la operación, qué privilegio y alcance necesita y si está autorizado para delegar. Una concesión que no está autorizada no agrega el permiso al grafo. Si una sentencia completa falla, no se aplican sus cambios; en PostgreSQL, algunas concesiones no autorizadas pueden producir advertencias en lugar de un error, por lo que no hay que confundir «no concede el permiso» con «siempre arroja un error».

Ejemplo: A es propietario de voluntario. B, C y D son usuarios ordinarios y no tienen permisos previos, ni directamente ni mediante roles o PUBLIC. La tabla tiene nombre y otras columnas. El asterisco en los grafos representa WITH GRANT OPTION.

### Secuencia

```sql
-- 1. Ejecutado por A:
GRANT SELECT ON voluntario TO B WITH GRANT OPTION;

-- 2. Ejecutado por B:
GRANT SELECT ON voluntario TO C;

-- 3. Intentado por C:
GRANT SELECT ON voluntario TO D;

-- 4. Ejecutado por A:
GRANT SELECT(nombre) ON voluntario TO C;

-- 5. Ejecutado por A:
REVOKE GRANT OPTION FOR SELECT ON voluntario FROM B CASCADE;
```

| Paso | Razonamiento y resultado |
|---|---|
| 1 | A puede conceder el permiso. B obtiene SELECT sobre toda la tabla y puede delegarlo. |
| 2 | B está autorizado. C obtiene SELECT sobre toda la tabla, sin capacidad de delegación. |
| 3 | C puede consultar, pero no delegar. No se concede SELECT a D. |
| 4 | C obtiene además una concesión directa de A sobre nombre. Su permiso efectivo sigue abarcando toda la tabla por la concesión de B. |
| 5 | B pierde únicamente la opción de delegar y conserva SELECT. La concesión de B a C se retira en cascada; la concesión directa de A a C sobre nombre se conserva. |

Antes del paso 5:

```text
A ── SELECT* ──→ B ── SELECT ──→ C
A ── SELECT(nombre) ──────────→ C
```

Después del paso 5:

```text
A ── SELECT ──→ B
A ── SELECT(nombre) ──→ C
```

### Permisos finales y operaciones posibles

- B puede consultar toda la tabla, pero no delegar SELECT.
- C puede consultar nombre, pero no las demás columnas ni delegar SELECT.
- D no recibió permisos.

Por ejemplo, C puede ejecutar SELECT nombre FROM voluntario, pero no SELECT * FROM voluntario porque la tabla contiene otras columnas. Tampoco puede filtrar por una columna no autorizada: consultar nombre con un WHERE sobre nro_voluntario requeriría además SELECT sobre nro_voluntario.

El método de resolución consiste en actualizar el estado después de cada sentencia válida y, al terminar, reunir los permisos que conserva cada usuario por todos sus caminos independientes. No basta con mirar la última sentencia ni con preguntar si el usuario tiene «algún permiso»: hay que distinguir operación, columnas, capacidad de delegar y origen de cada concesión.

## Material

Enunciado: Practicos/Practico 6 - Seguridad.docx, dentro de la carpeta Bases de Datos, fuera del baúl.

Referencia para los detalles indicados de PostgreSQL: [GRANT](https://www.postgresql.org/docs/current/sql-grant.html) y [privilegios](https://www.postgresql.org/docs/17/ddl-priv.html).
