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

## 7. Usuarios y roles

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

## 8. Analizar secuencias y permisos finales

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
