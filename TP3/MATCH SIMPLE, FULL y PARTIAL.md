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

## Declarar una referencia

La FK se incorpora en la hija y referencia una clave completa del padre. Las acciones ON DELETE y ON UPDATE se eligen según la regla del modelo.

[[TP3 - Declaración de FK con ALTER TABLE|Declaración de FK con ALTER TABLE]]

## MATCH SIMPLE, FULL y PARTIAL

Controlan los nulos de una **FK compuesta**.

| Modalidad   | Regla                                                                                  |
| ----------- | -------------------------------------------------------------------------------------- |
| **SIMPLE**  | Si hay algún NULL, no busca coincidencia. Sin nulos, exige la pareja exacta.           |
| **FULL**    | Todos NULL o pareja exacta sin nulos. Rechaza nulos parciales.                         |
| **PARTIAL** | Los valores informados deben coincidir en una misma fila padre. Todos NULL se admiten. |

### Declaración y casos de matching

La declaración y la tabla de casos muestran qué se acepta cuando falta uno o ambos componentes de una FK compuesta.

### Casos de MATCH SIMPLE, FULL y PARTIAL

Se parte de una única fila padre, cliente (A, 1), para distinguir las tres modalidades.

#### Cómo se declara

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

#### Qué acepta cada uno

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

**PostgreSQL:** SIMPLE es el valor por defecto; FULL está soportado; PARTIAL no está implementado.

#### Justificar el matching como búsqueda de una fila

En PARTIAL, para `('A', NULL)` se busca alguna fila padre cuyo primer componente sea A; el segundo queda sin exigir. Para `('A', 3)` se busca exactamente esa pareja. No se acepta porque A aparezca en una fila y 3 en otra. FULL añade la exigencia de que no haya un nulo parcial, y SIMPLE deja de exigir coincidencia cuando alguno de los componentes es NULL.

El matching decide sobre esa FK. Para aceptar toda una inserción también se revisan las restantes FK, PK, UNIQUE y NOT NULL. Superar el matching de una referencia no garantiza que la sentencia completa sea admisible.

## Seguir una cadena de acciones

Las acciones se propagan desde una modificación del padre hacia las referencias hijas. Una modificación directa de la hija no crea padres ni dispara una cascada hacia arriba.

### Ejemplo de cascadas y dirección de las acciones

Libro, ejemplares y préstamos permiten seguir una cadena de acciones y una posible restricción que la bloquee.

#### Seguir las acciones en la dirección correcta

Supongamos un libro L1 con ejemplares `(L1, 1)` y `(L1, 2)`. Si EJEMPLAR referencia LIBRO con ON DELETE CASCADE, borrar L1 solicita eliminar ambos ejemplares. Borrar solamente el ejemplar `(L1, 1)` no elimina el libro: la acción declarada se propaga desde la clave padre hacia sus referencias hijas.

Con ON UPDATE CASCADE, cambiar la clave del libro L1 a L9 transforma las referencias de sus ejemplares en `(L9, 1)` y `(L9, 2)`. En cambio, `UPDATE ejemplar SET isbn = 'L9' ...` es una modificación directa de la hija: necesita que exista L9 en LIBRO. No invoca una cascada para crear ese libro.

Si PRESTAMO referencia EJEMPLAR con RESTRICT, un préstamo puede impedir que se complete la eliminación en cascada que comenzó en LIBRO. Se revisa toda la cadena, no solo la primera FK. Si se rechaza la sentencia, no queda borrado el libro con sus ejemplares a medias.

Para justificar un caso se busca una misma fila padre que cumpla los componentes exigidos por la modalidad. Después se comprueban las demás restricciones de la sentencia.

## Fuente de estudio

Enunciado del TP3 en la carpeta `../../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
