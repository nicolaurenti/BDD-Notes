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

## Ejercicio 1.a — Agregar claves foráneas con ALTER TABLE

El enunciado indica que las tablas y sus PK **ya existen**. Se pide incorporar las restricciones de integridad referencial (RIR), por lo que corresponde usar `ALTER TABLE ... ADD CONSTRAINT`, sin volver a crear las tablas.

### Paso 1. Leer cada relación del diagrama

La **tabla hija** contiene la FK; la **tabla padre** contiene la clave referenciada. La sentencia se escribe sobre la hija.

| RIR | Tabla hija y columnas FK | Tabla padre y clave referenciada | [Baja, modificación a derecha] |
|---|---|---|---|
| R1 | `Trabaja_En (TipoE, NroE)` | `Empleado (TipoE, NroE)` | `[C, R]` |
| R2 | `Trabaja_En (IdProy)` | `Proyecto (IdProy)` | `[R, C]` |
| R3 | `Auspicio (IdProy)` | `Proyecto (IdProy)` | `[R, R]` |
| R4 | `Auspicio (TipoE, NroE)` | `Empleado (TipoE, NroE)` | `[N, R]` |

Se usa `Trabaja_En`, como figura en el diagrama; algunas operaciones del texto lo escriben `TrabajaEn`. Al aplicar las sentencias, usar el nombre real de la tabla.

### Paso 2. Traducir las acciones

El primer componente del par corresponde a `ON DELETE`; el segundo a `ON UPDATE`. La «modificación a derecha» es el cambio de la clave referenciada en la tabla padre.

| Símbolo | Cláusula SQL | Significado |
|---|---|---|
| `R` | `RESTRICT` | Impide borrar o cambiar la clave del padre si hay referencias afectadas. |
| `C` | `CASCADE` | Propaga el borrado o el cambio de clave a las filas hijas. |
| `N` | `SET NULL` | Pone en NULL las columnas de la FK afectada. |

Por ejemplo, `[C, R]` se traduce a `ON DELETE CASCADE ON UPDATE RESTRICT`.

### Paso 3. Completar la plantilla

```sql
-- Plantilla: reemplazar los marcadores por nombres del diagrama.
ALTER TABLE tabla_hija
ADD CONSTRAINT nombre_de_la_restriccion
FOREIGN KEY (columna_fk_1, columna_fk_2)
REFERENCES tabla_padre (columna_clave_1, columna_clave_2)
ON DELETE accion_de_baja
ON UPDATE accion_de_modificacion;
```

Para una FK simple, cada lista tiene una sola columna. `ADD CONSTRAINT` da un nombre a la regla; no agrega columnas ni cambia la PK existente.

### Ejemplo resuelto 1: R1, una FK compuesta

Cada asignación en `Trabaja_En` debe corresponder a un empleado existente, identificado por **la pareja** `(TipoE, NroE)`.

```sql
ALTER TABLE Trabaja_En
ADD CONSTRAINT fk_r1_trabaja_en_empleado
FOREIGN KEY (TipoE, NroE)
REFERENCES Empleado (TipoE, NroE)
ON DELETE CASCADE
ON UPDATE RESTRICT;
```

Lectura de cada parte:

- `ALTER TABLE Trabaja_En`: se modifica la tabla que contiene la referencia.
- `FOREIGN KEY (TipoE, NroE)`: ambos atributos forman **una sola FK**.
- `REFERENCES Empleado (TipoE, NroE)`: se exige esa misma combinación en Empleado, respetando el orden de las columnas.
- `ON DELETE CASCADE`: al borrar el empleado, R1 propaga el borrado a sus asignaciones.
- `ON UPDATE RESTRICT`: si tiene asignaciones, R1 impide cambiar su clave referenciada.

Por ejemplo, con empleados `('A', 1)` y `('B', 2)`, una asignación para `('A', 2)` no cumple esta FK. No alcanza con que exista el tipo A en una fila y el número 2 en otra. Dos FK separadas tampoco expresan esta regla; además, las columnas referenciadas individualmente necesitarían su propia garantía de unicidad.

Estos efectos describen R1: para decidir si una operación completa se acepta, también hay que comprobar las demás restricciones.

### Ejemplo resuelto 2: R2, una FK simple

Cada asignación debe referenciar un proyecto existente. Para `[R, C]`:

```sql
ALTER TABLE Trabaja_En
ADD CONSTRAINT fk_r2_trabaja_en_proyecto
FOREIGN KEY (IdProy)
REFERENCES Proyecto (IdProy)
ON DELETE RESTRICT
ON UPDATE CASCADE;
```

Si un proyecto está referenciado desde `Trabaja_En`, R2 impide borrarlo. Si cambia su `IdProy`, R2 propaga la nueva clave a esas asignaciones. Otras restricciones podrían impedir la operación completa.

### Para completar R3 y R4

Aplicar la misma plantilla con las filas de la tabla inicial:

- **R3:** alterar `Auspicio`, declarar la FK simple sobre `IdProy` y usar `RESTRICT` en ambas acciones.
- **R4:** alterar `Auspicio`, declarar una única FK compuesta `(TipoE, NroE)` hacia Empleado y traducir `[N, R]`. En el diagrama ambas columnas admiten NULL, por lo que pueden recibir el resultado de `SET NULL`.

El apartado 1.a no fija un matching especial. Si se omite `MATCH`, PostgreSQL usa `MATCH SIMPLE`. El apartado 1.c pide comparar las modalidades de R4; no hay que declarar tres FK simultáneas para resolver esa comparación.

### Errores frecuentes al escribir el 1.a

- Alterar la tabla padre en lugar de la hija.
- Invertir `ON DELETE` y `ON UPDATE` al leer el par.
- Usar solo `NroE` cuando la clave es `(TipoE, NroE)`.
- Incluir toda la PK de `Trabaja_En` en la FK a Empleado: `IdProy`, `Anio` y `Mes` no pertenecen a esa referencia.
- Creer que `ON UPDATE CASCADE` permite asignar directamente a una hija una FK cuyo padre no existe.

Ejemplos de estudio: no se ejecutó SQL. Sintaxis de referencia: [ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html) y [FOREIGN KEY y acciones referenciales](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK).

## MATCH SIMPLE, FULL y PARTIAL

Controlan los nulos de una **FK compuesta**.

| Modalidad | Regla |
|---|---|
| **SIMPLE** | Si hay algún NULL, no busca coincidencia. Sin nulos, exige la pareja exacta. |
| **FULL** | Todos NULL o pareja exacta sin nulos. Rechaza nulos parciales. |
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
