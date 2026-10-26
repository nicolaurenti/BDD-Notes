---
tags:
  - tp6
---

# Delegación y grafos de permisos

## Delegar permisos con WITH GRANT OPTION

WITH GRANT OPTION permite que el destinatario otorgue a otros el privilegio recibido:

```sql
-- Ejecutado por el propietario:
GRANT SELECT ON voluntario TO ana WITH GRANT OPTION;

-- Ejecutado por Ana:
GRANT SELECT ON voluntario TO juan;
```

Ana puede consultar y conceder SELECT. Juan puede consultar, pero no delegar a partir de esta concesión, porque Ana no incluyó WITH GRANT OPTION. Ana también podría concederle esa capacidad:

```sql
GRANT SELECT ON voluntario TO juan WITH GRANT OPTION;
```

Solo se delega lo que se recibió con esa opción y dentro de su alcance. Si Ana tiene SELECT con opción de delegación y UPDATE sin ella, puede conceder SELECT, pero no UPDATE.

El alcance por columna también se conserva:

```sql
GRANT UPDATE(nombre) ON voluntario TO ana
WITH GRANT OPTION;
```

Ana puede conceder UPDATE(nombre), pero esta autorización no le permite conceder UPDATE(apellido) ni UPDATE sobre toda la tabla.

## Grafos de permisos

Un grafo representa quién otorgó cada privilegio a quién. En estos ejemplos, el asterisco indica WITH GRANT OPTION:

```text
A ── SELECT* ──→ B ── SELECT ──→ C
```

A es el propietario; B puede consultar y delegar; C puede consultar, pero no delegar a partir de esa concesión.

Cada privilegio y su alcance se analizan por separado:

```text
A ── SELECT* ──→ B ── SELECT ──→ C
A ── UPDATE  ──→ B
```

B puede modificar datos, pero no conceder UPDATE. Si intenta hacerlo, la sentencia falla y no se agrega una flecha al grafo.

### Varios caminos y acumulación de permisos

Un mismo usuario puede recibir permisos de distintos otorgantes. Por ejemplo, suponiendo que B está autorizado a delegar SELECT(nombre):

```sql
-- Ejecutado por B:
GRANT SELECT(nombre) ON voluntario TO C;

-- Ejecutado por A:
GRANT SELECT ON voluntario TO C;
```

C puede consultar toda la tabla, incluido SELECT * FROM voluntario. Los permisos se acumulan: la concesión limitada de B no restringe la concesión de A sobre la tabla completa. Decir que «prevalece el más abarcativo» describe el resultado en este caso, pero la regla general es que se reúnen los permisos de todos los caminos válidos.

Ambas concesiones siguen existiendo. Si A retira su concesión directa a C, C conserva el permiso sobre nombre concedido por B, siempre que ese camino siga siendo válido. Por eso perder un camino no implica necesariamente perder todo el privilegio.

## Material

Enunciado: Practicos/Practico 6 - Seguridad.docx, dentro de la carpeta Bases de Datos, fuera del baúl.

Referencia para los detalles indicados de PostgreSQL: [GRANT](https://www.postgresql.org/docs/current/sql-grant.html) y [privilegios](https://www.postgresql.org/docs/17/ddl-priv.html).
