---
tags:
  - tp3
---

# TP3 - Ejemplos de restricciones de atributo y tupla

Ejemplos de nacionalidad, dominio reutilizable, rango de horas y datos no ambos nulos.

## 1. Restricción de atributo o dominio

Controla un único valor, sin consultar otras filas.

### CHECK sobre una columna

```sql
CREATE TABLE articulo_ejemplo (
    id_articulo INTEGER PRIMARY KEY,
    nacionalidad VARCHAR(20) NOT NULL
        CHECK (nacionalidad IN ('Argentina', 'Español', 'Inglesa'))
);
```

- `'Argentina'`: aceptado.
- `'Francesa'`: rechazado por el CHECK.
- `NULL`: rechazado por NOT NULL.

El CHECK limita qué valores se admiten; NOT NULL obliga a informar uno. Sin NOT NULL, el nulo no sería rechazado por este CHECK.

### DOMAIN para reutilizar la regla

Un dominio es un tipo de dato con restricciones que se puede usar en distintas columnas.

```sql
CREATE DOMAIN nacionalidad_permitida AS VARCHAR(20)
CHECK (VALUE IN ('Argentina', 'Español', 'Inglesa'));

CREATE TABLE autor_ejemplo (
    id_autor INTEGER PRIMARY KEY,
    nacionalidad nacionalidad_permitida NOT NULL
);
```

`VALUE` representa el valor que se está comprobando. Cualquier columna de este dominio reutiliza su lista de nacionalidades. Ambos ejemplos son compatibles con PostgreSQL.

## 2. Restricción de tupla

Compara atributos de **una misma fila**. No necesita leer otras filas de la tabla.

```sql
CREATE TABLE tarea_ejemplo (
    id_tarea INTEGER PRIMARY KEY,
    min_horas INTEGER NOT NULL,
    max_horas INTEGER NOT NULL,
    CONSTRAINT ck_rango_horas
        CHECK (min_horas <= max_horas)
);
```

| min_horas | max_horas | Resultado |
|---:|---:|---|
| 100 | 500 | Aceptado |
| 500 | 500 | Aceptado: se permite igualdad |
| 700 | 500 | Rechazado |

Otro caso del práctico: descripción y presentación no pueden ser ambas nulas.

```sql
ALTER TABLE producto
ADD CONSTRAINT ck_datos_producto
CHECK (descripcion IS NOT NULL OR presentacion IS NOT NULL);
```

Una puede ser nula, pero al menos una debe estar informada. Tener ambas informadas también es válido. Una cadena vacía (`''`) no es NULL: si tampoco se admite texto vacío, hace falta otro control.

Aunque el CHECK se escriba después de todas las columnas, sigue siendo **de tupla**, porque solo consulta valores de la fila evaluada.

## Fuente de estudio

Enunciado del TP3 en la carpeta `../../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
