---
tags:
  - tp3
---

# Integridad referencial y matching

## Acciones referenciales

Se declaran en la tabla que contiene la FK, pero se activan al modificar o borrar la fila **referenciada**.

| Acción | Al borrar la fila referenciada | Al modificar su clave |
|---|---|---|
| `RESTRICT` | Rechaza si hay referencias | Rechaza si hay referencias afectadas |
| `CASCADE` | Borra las filas que la referencian | Propaga la nueva clave a las FK |
| `SET NULL` | Pone la FK afectada en nulo | Pone la FK afectada en nulo |
| `SET DEFAULT` | Asigna el valor por defecto | Asigna el valor por defecto |
| `NO ACTION` | Exige que la FK quede satisfecha al comprobarla | Lo mismo |

`NO ACTION` puede permitir comprobación diferida si la restricción se declaró diferible. `RESTRICT` no difiere esa acción. No asumir que son siempre intercambiables.

- `SET NULL` puede fallar si una columna es `NOT NULL` o parte de una PK.
- `SET DEFAULT` también debe dejar una referencia válida.
- Una cascada puede alcanzar otras tablas; si otra restricción impide el resultado, falla toda la sentencia.
- Cambiar directamente una FK en la tabla hija exige que el nuevo padre exista; `ON UPDATE CASCADE` no crea padres.
- Borrar una fila hija no borra automáticamente su padre.

Base técnica: [restricciones y acciones referenciales en PostgreSQL](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK).

## Agregar restricciones de integridad referencial con ALTER TABLE

Cuando las tablas ya existen, se pueden incorporar claves foráneas mediante `ALTER TABLE ... ADD CONSTRAINT`. Los siguientes ejemplos usan una biblioteca: `libro`, `ejemplar` y `prestamo`.

### Identificar la tabla que contiene la referencia

La **tabla hija** contiene la FK; la **tabla padre** contiene la clave referenciada. La sentencia se escribe sobre la hija.

| Relación del ejemplo | Tabla hija y FK | Tabla padre y clave referenciada |
|---|---|---|
| Cada ejemplar pertenece a un libro | `ejemplar (isbn)` | `libro (isbn)` |
| Cada préstamo corresponde a un ejemplar | `prestamo (isbn, nro_ejemplar)` | `ejemplar (isbn, nro_ejemplar)` |

Se supone que `libro` tiene PK `(isbn)` y `ejemplar` tiene PK `(isbn, nro_ejemplar)`. Una FK también puede referenciar una clave declarada `UNIQUE` apropiada.

### Plantilla de la sentencia

```sql
-- Plantilla: sustituir los marcadores por nombres y acciones concretos.
ALTER TABLE tabla_hija
ADD CONSTRAINT nombre_de_la_restriccion
FOREIGN KEY (columna_fk_1, columna_fk_2)
REFERENCES tabla_padre (columna_clave_1, columna_clave_2)
ON DELETE accion_de_baja
ON UPDATE accion_de_modificacion;
```

`ADD CONSTRAINT` agrega una regla con nombre. Las columnas deben existir y los datos ya almacenados deben cumplir la nueva restricción. Para una FK simple se indica una sola columna en cada lista.

### Ejemplo de FK simple

Regla elegida: impedir borrar un libro si tiene ejemplares y propagar los cambios de su ISBN a esos ejemplares.

```sql
ALTER TABLE ejemplar
ADD CONSTRAINT fk_ejemplar_libro
FOREIGN KEY (isbn)
REFERENCES libro (isbn)
ON DELETE RESTRICT
ON UPDATE CASCADE;
```

- `ALTER TABLE ejemplar`: modifica la tabla hija.
- `FOREIGN KEY (isbn)`: identifica la columna que contiene la referencia.
- `REFERENCES libro (isbn)`: indica dónde debe existir el ISBN.
- `ON DELETE RESTRICT`: impide borrar un libro mientras tenga ejemplares que lo referencien.
- `ON UPDATE CASCADE`: si cambia el ISBN del libro, actualiza ese valor en sus ejemplares.

Si existe el libro con ISBN `'978-001'`, un ejemplar puede referenciarlo. Una referencia a `'978-999'` se rechaza si ese libro no existe.

### Ejemplo de FK compuesta

Un ejemplar se identifica por la **combinación** de ISBN y número de ejemplar. El número por sí solo puede repetirse para libros diferentes.

Regla elegida: impedir borrar o cambiar la clave de un ejemplar mientras tenga préstamos que lo referencien.

```sql
ALTER TABLE prestamo
ADD CONSTRAINT fk_prestamo_ejemplar
FOREIGN KEY (isbn, nro_ejemplar)
REFERENCES ejemplar (isbn, nro_ejemplar)
ON DELETE RESTRICT
ON UPDATE RESTRICT;
```

Las columnas se emparejan por posición: `isbn` con `isbn`, y `nro_ejemplar` con `nro_ejemplar`. Los nombres podrían ser diferentes en cada tabla; deben corresponderse sus tipos y su significado.

Si los únicos ejemplares son `('978-001', 1)` y `('978-002', 2)`:

| Referencia del préstamo | Resultado de la FK |
|---|---|
| `('978-001', 1)` | Válida: existe esa combinación. |
| `('978-001', 2)` | Inválida: los valores aparecen por separado, pero no juntos en un ejemplar. |

Dos FK independientes no reemplazan una FK compuesta: no comprobarían que los valores pertenezcan a la misma fila padre, y necesitarían claves referenciables individuales.

### Elegir las acciones según la regla

Las acciones anteriores son decisiones de los ejemplos. En otro modelo podrían corresponder otras acciones:

```sql
-- Fragmentos alternativos para la declaración de una FK:
ON DELETE CASCADE
ON UPDATE RESTRICT
```

`CASCADE` al borrar elimina las filas hijas que referencian al padre. Con `SET NULL`, en cambio, se conservan esas filas y se ponen en NULL las columnas de la FK; esas columnas deben admitir nulos.

Cuando una consigna usa un par **[baja, modificación a derecha]**, el primer componente corresponde a `ON DELETE` y el segundo a `ON UPDATE`. La modificación a derecha se refiere al cambio de la clave referenciada en la tabla padre.

| Símbolo | Acción SQL |
|---|---|
| `R` | `RESTRICT` |
| `C` | `CASCADE` |
| `N` | `SET NULL` |

Las acciones se evalúan junto con las demás restricciones: una cascada puede quedar impedida por otra regla. Si se omite `MATCH`, PostgreSQL usa `MATCH SIMPLE`.

### Errores frecuentes

- Alterar la tabla padre en lugar de la hija.
- Invertir las acciones de baja y modificación.
- Usar solo una parte de una clave compuesta o cambiar el orden de correspondencia.
- Confundir la PK completa de la hija con las columnas que necesita una FK concreta.
- Creer que `ON UPDATE CASCADE` permite asignar directamente a una hija una FK cuyo padre no existe.

Ejemplos de estudio: no se ejecutó SQL. Sintaxis de referencia: [ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html) y [FOREIGN KEY y acciones referenciales](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK).

## MATCH SIMPLE, FULL y PARTIAL

Controlan los nulos de una **FK compuesta**.

| Modalidad   | Regla                                                                                  |
| ----------- | -------------------------------------------------------------------------------------- |
| **SIMPLE**  | Si hay algún NULL, no busca coincidencia. Sin nulos, exige la pareja exacta.           |
| **FULL**    | Todos NULL o pareja exacta sin nulos. Rechaza nulos parciales.                         |
| **PARTIAL** | Los valores informados deben coincidir en una misma fila padre. Todos NULL se admiten. |

### Cómo se declara

```sql
CREATE TABLE cliente (
    zona CHAR(1),
    nro_cliente INTEGER,
    PRIMARY KEY (zona, nro_cliente)
);

INSERT INTO cliente VALUES ('A', 1);

CREATE TABLE pedido (
    id_pedido INTEGER PRIMARY KEY,
    zona CHAR(1),
    nro_cliente INTEGER,
    FOREIGN KEY (zona, nro_cliente)
        REFERENCES cliente (zona, nro_cliente)
        MATCH SIMPLE
);
```

Para elegir otra modalidad, reemplazar `MATCH SIMPLE` por `MATCH FULL` o, en SQL estándar, `MATCH PARTIAL`. Las columnas de la FK admiten nulos en este ejemplo.

### Qué acepta cada uno

La única fila de `cliente` es **`('A', 1)`**:

| FK que se intenta insertar en pedido | SIMPLE | FULL | PARTIAL |
|---|:---:|:---:|:---:|
| `('A', 1)` | Sí | Sí | Sí |
| `('A', 9)` | No | No | No |
| `('A', NULL)` | Sí | No | Sí: coincide A |
| `(NULL, 1)` | Sí | No | Sí: coincide 1 |
| `('Z', NULL)` | Sí | No | No: no existe zona Z |
| `(NULL, NULL)` | Sí | Sí | Sí |

Ejemplo concreto:

```sql
INSERT INTO pedido VALUES (10, 'Z', NULL);
-- SIMPLE: acepta por tener un NULL.
-- FULL: rechaza porque solo una parte es NULL.
-- PARTIAL: rechazaría porque ninguna fila padre tiene zona Z.
```

**PostgreSQL:** SIMPLE es el valor por defecto; FULL está soportado; PARTIAL no está implementado. [Referencia](https://www.postgresql.org/docs/current/sql-createtable.html).
