---
tags:
  - tp6
---

# TP6 - Seguridad

La seguridad de una base de datos permite definir qué operaciones puede realizar cada usuario sobre sus objetos. Este apunte reúne las explicaciones y dudas trabajadas en la conversación, con ejemplos ilustrativos que no se ejecutaron sobre la base.

## 1. Privilegios sobre tablas

Un privilegio es un permiso para realizar una operación sobre un objeto. Los principales para este práctico son:

| Privilegio | Qué permite |
|---|---|
| SELECT | Consultar datos. |
| INSERT | Agregar filas. |
| UPDATE | Modificar datos de filas existentes. |
| DELETE | Eliminar filas. |

Los privilegios son independientes: poder consultar una tabla no implica poder modificarla. Se analizan por usuario, objeto y operación. Por ejemplo, un usuario puede consultar una tabla, modificar otra y no tener acceso a una tercera.

ALL PRIVILEGES permite conceder todos los privilegios aplicables al objeto que quien concede esté autorizado a otorgar. No convierte al destinatario en administrador ni le da automáticamente capacidad para delegar esos privilegios.

## 2. Otorgar permisos con GRANT

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

## 3. Delegar permisos con WITH GRANT OPTION

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

## 4. Grafos de permisos

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

## 5. Revocar permisos con REVOKE

REVOKE retira una concesión de un privilegio. La estructura es REVOKE privilegio ON tabla FROM usuario, seguida del comportamiento ante permisos dependientes: CASCADE o RESTRICT.

Supongamos esta secuencia:

```sql
-- Ejecutado por A, propietario:
GRANT SELECT ON voluntario TO B WITH GRANT OPTION;

-- Ejecutado por B:
GRANT SELECT ON voluntario TO C;
```

El grafo es:

```text
A ── SELECT* ──→ B ── SELECT ──→ C
```

Si A ejecuta:

```sql
REVOKE SELECT ON voluntario FROM B CASCADE;
```

Se retira el permiso concedido a B y también la concesión dependiente de B a C. Suponiendo que no tienen otros permisos, ninguno puede seguir consultando la tabla. CASCADE propaga la revocación por las concesiones que quedan sin sustento; no elimina usuarios, tablas ni datos.

### Otro camino válido

Si A también había concedido SELECT directamente a C, esa concesión no depende de B:

```text
A ── SELECT* ──→ B ── SELECT ──→ C
A ── SELECT ──────────────────→ C
```

Al revocar a B con CASCADE, C conserva SELECT por su concesión directa de A. Para resolver un ejercicio hay que examinar todos los caminos válidos, no solo la cadena afectada.

### RESTRICT

Con el grafo inicial, A podría intentar:

```sql
REVOKE SELECT ON voluntario FROM B RESTRICT;
```

Como existe una concesión dependiente a C que tendría que retirarse, la sentencia falla y no realiza la revocación. RESTRICT exige que no haya una revocación dependiente que efectuar; CASCADE permite propagarla. Es conveniente escribir explícitamente la modalidad pedida por el ejercicio.

### Retirar solamente la capacidad de delegar

También es posible retirar la opción de delegación conservando el privilegio de uso:

```sql
REVOKE GRANT OPTION FOR SELECT ON voluntario FROM B CASCADE;
```

B conserva SELECT, pero pierde la capacidad de otorgarlo por esa concesión. Los permisos que había delegado y que dependían exclusivamente de esa autorización se revocan en cascada. Como siempre, otros caminos independientes pueden sostener permisos y opciones de delegación.

## 6. Permisos para todos mediante PUBLIC

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

## Temas pendientes de desarrollar en la conversación

- Creación, asignación y eliminación de usuarios y roles.
- Análisis de secuencias de operaciones y permisos finales.

## Material

Enunciado: Practicos/Practico 6 - Seguridad.docx, dentro de la carpeta Bases de Datos, fuera del baúl.

Referencia para los detalles indicados de PostgreSQL: [GRANT](https://www.postgresql.org/docs/current/sql-grant.html) y [privilegios](https://www.postgresql.org/docs/17/ddl-priv.html).
