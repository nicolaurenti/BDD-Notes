---
tags:
  - tp3
---

# Alcance y clasificación de restricciones


Una restricción expresa qué estados de la base son válidos. No alcanza con controlar la aplicación: el motor debe rechazar las operaciones que dejan datos inconsistentes.

| Alcance usado por la cátedra | Qué compara | Ejemplo | Recurso |
|---|---|---|---|
| Atributo o dominio | Un valor | Nacionalidad dentro de una lista | `CHECK`, `DOMAIN`, `NOT NULL` |
| Tupla | Columnas de la misma fila | Descripción y presentación no ambas nulas | `CHECK` de fila |
| Tabla | Varias filas de una tabla | Hasta 15 palabras por artículo en `contiene` | Chequeo de tabla en el modelo estándar; trigger si requiere agregación en PostgreSQL |
| General | Datos de varias tablas | Horas del voluntario dentro de los límites de su tarea | `ASSERTION` en el estándar; alternativa procedural en PostgreSQL |

La clasificación depende del modelo: identificar primero dónde están los atributos y qué filas hay que consultar. Un `CHECK` escrito al final del `CREATE TABLE` no se vuelve, por esa ubicación, un control de múltiples filas.

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

## 3. Restricción de tabla

Para decidir si una fila es válida, se consideran **otras filas de la misma tabla**.

### Unicidad con UNIQUE — PostgreSQL

```sql
CREATE TABLE usuario_ejemplo (
    id_usuario INTEGER PRIMARY KEY,
    email VARCHAR(120) NOT NULL,
    CONSTRAINT uq_usuario_email UNIQUE (email)
);
```

- Primera fila con `ana@ejemplo.com`: aceptada.
- Otra fila con distinto id pero el mismo email: rechazada.

La regla no compara dos atributos de una fila: necesita garantizar que no haya **otra fila** con ese email. UNIQUE es el recurso declarativo específico para esta regla.

### Límite de filas por grupo — ejemplo del práctico

«Cada artículo puede tener como máximo 15 palabras clave». Suponiendo que cada fila de `contiene` representa una asociación única entre artículo y palabra:

```sql
-- SQL estándar: PostgreSQL NO implementa CREATE ASSERTION.
CREATE ASSERTION max_15_palabras_por_articulo
CHECK (
    NOT EXISTS (
        SELECT id_articulo
        FROM contiene
        GROUP BY id_articulo
        HAVING COUNT(*) > 15
    )
);
```

Lectura: **no debe existir un artículo con más de 15 filas en contiene**.

- Con 14 asociaciones, agregar una deja 15: permitido por esta regla.
- Con 15, agregar otra deja 16: rechazado.
- Mover una asociación a otro artículo también puede superar el límite: no controlar únicamente INSERT.

El alcance es de tabla porque el control consulta solo `contiene`, aunque use una aserción como recurso declarativo del estándar. No hace falta consultar `articulo` para verificar este máximo.

En PostgreSQL, este límite agregado no puede escribirse como CHECK con subconsulta: requiere una implementación procedural que contemple los eventos relevantes y el acceso concurrente. En cambio, la unicidad del ejemplo anterior sí se implementa directamente con UNIQUE.

## 4. Restricción general

Relaciona datos de **varias tablas**.

«Las horas aportadas por cada voluntario deben estar entre el mínimo y el máximo de su tarea».

```sql
-- SQL estándar: PostgreSQL NO implementa CREATE ASSERTION.
CREATE ASSERTION horas_dentro_de_la_tarea
CHECK (
    NOT EXISTS (
        SELECT 1
        FROM voluntario v
        JOIN tarea t ON t.id_tarea = v.id_tarea
        WHERE v.horas_aportadas < t.min_horas
           OR v.horas_aportadas > t.max_horas
    )
);
```

La subconsulta busca **infractores**. La restricción exige que no encuentre ninguno.

Si la tarea permite entre 100 y 500 horas:

| Horas del voluntario | Resultado |
|---:|---|
| 100 | Aceptado |
| 350 | Aceptado |
| 500 | Aceptado |
| 600 | Rechazado |

Este ejemplo supone que horas y límites no admiten NULL y que una FK garantiza la tarea referenciada. Si esos supuestos no se cumplen, definir cómo tratar valores ausentes: ni el JOIN ni las comparaciones detectan automáticamente todos esos casos como infracción.

**También hay que controlar cambios en tarea:** si un voluntario aporta 350 horas y se baja el máximo de su tarea a 300, la base quedaría inválida aunque no se modifique la fila del voluntario.

En PostgreSQL se implementa este control general mediante triggers sobre los eventos relevantes de ambas tablas; un CHECK local sobre voluntario no alcanza.

## Cómo elegir el alcance

1. ¿La regla se decide mirando un solo atributo? **Atributo o dominio.**
2. ¿Necesito varios atributos, pero solo de la misma fila? **Tupla.**
3. ¿Necesito otras filas de la misma tabla? **Tabla.**
4. ¿Necesito consultar varias tablas? **General.**

Primero determinar el alcance; después elegir el recurso que lo expresa. El nombre de la sentencia no determina por sí solo la clasificación: una ASSERTION puede expresar una regla que consulta una sola tabla, y una restricción escrita al final de CREATE TABLE puede seguir siendo de tupla.

Los fragmentos son ejemplos de estudio: no se ejecutaron ni se crearon restricciones en las bases existentes.

