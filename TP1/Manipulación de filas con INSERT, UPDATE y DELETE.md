---
tags:
  - tp1
---

# Manipulación de filas con INSERT, UPDATE y DELETE

Se utiliza `INSERT INTO`.

```sql
INSERT INTO articulo (
    id_articulo,
    titulo,
    autor,
    fecha_pub
)
VALUES (
    1,
    'Bases de datos',
    'Ana',
    DATE '2026-10-18'
);
```

También pueden insertarse varias filas:

```sql
INSERT INTO palabra (cod_p, idioma, descripcion)
VALUES
    (1, 'ES', 'Datos'),
    (2, 'ES', 'Tablas'),
    (1, 'EN', 'Data');
```

Las filas con código `1` son válidas porque la PK incluye también el idioma.

## Cómo se corresponden columnas y valores

La primera expresión de VALUES se asigna a la primera columna listada, la segunda a la segunda, etc. Escribir la lista evita depender del orden físico de las columnas. Una columna omitida toma su valor por defecto o NULL; si es obligatoria y no tiene un valor apropiado, la inserción falla.

Para cargar una asociación en CONTIENE, primero deben existir el artículo y la palabra. Una fila `(1, 1, 'ES')` no crea automáticamente ninguno de esos padres: solo los referencia. Si la palabra cargada es `(1, 'EN')`, la asociación en español será inválida.

En el ejercicio de «por lo menos cinco tuplas», no basta con contar cinco VALUES: deben ser cinco filas válidas bajo **todas** las restricciones. Conviene elegir datos que muestren distintos casos del modelo, por ejemplo un equipo usado por varias grabaciones o un mismo código de palabra en idiomas diferentes.

## Actualización de filas con UPDATE

Se utiliza `UPDATE`.

### Actualizar una fila

Se filtra por una clave que identifique la fila:

```sql
UPDATE articulo
SET autor = 'Juan'
WHERE id_articulo = 1;
```

#### Actualizar un subconjunto de filas

```sql
UPDATE articulo
SET autor = 'María'
WHERE autor = 'Ana';
```

#### Actualizar todas las filas

```sql
UPDATE articulo
SET autor = 'Autor pendiente';
```

> [!warning]
> Un `UPDATE` sin `WHERE` afecta a todas las filas de la tabla.

### Qué determina cuántas filas se modifican

WHERE se evalúa para cada fila. Si filtra por una clave completa, selecciona como máximo una; si filtra por un atributo no único, puede seleccionar varias. «Cambiar el autor Ana» no implica una sola fila aunque hoy exista un único artículo suyo.

En una tabla con PK `(cod_p, idioma)`, `WHERE cod_p = 1` puede afectar palabras de varios idiomas. Para una sola palabra usar también `AND idioma = 'ES'`.

SET admite expresiones basadas en el valor anterior: `SET duracion = duracion + 5` incrementa cinco unidades; `SET duracion = 5` reemplaza el valor por cinco. Los valores resultantes deben seguir cumpliendo los tipos y restricciones. Un UPDATE puede seleccionar correctamente una fila y aun así rechazarse porque produce una PK repetida o una FK inexistente.

Para justificar los tres apartados del TP1, señalar el criterio de selección: sin WHERE, condición no única o clave completa. Si no coincide ninguna fila, la sentencia no modifica datos; eso no significa necesariamente un error SQL.

## Borrado de filas con DELETE

Se utiliza `DELETE FROM tabla WHERE condición`.

- **Una fila:** filtrar por su clave.
- **Un subconjunto:** utilizar una condición que seleccione las filas.
- **Todas las filas:** omitir `WHERE`.

Borrar filas conserva la estructura de la tabla.

Las FK pueden impedir el borrado de una fila que está siendo referenciada, dependiendo de las acciones referenciales definidas.

### Ejemplos de los tres alcances

Sobre una tabla ilustrativa ARTICULO:

```sql
-- Una fila como máximo si id_articulo es la PK.
DELETE FROM articulo WHERE id_articulo = 10;

-- Todas las filas del autor indicado.
DELETE FROM articulo WHERE autor = 'Ana';

-- Todas las filas, conservando la tabla.
DELETE FROM articulo;
```

Son alternativas para estudiar el alcance de la sentencia, no una secuencia ejecutada. En el segundo caso puede haber cero, una o muchas coincidencias. DELETE no significa «borrar una tabla» ni «borrar una base».

Antes de determinar el resultado, revisar si otras tablas referencian las filas seleccionadas. Si CONTIENE referencia un artículo y la relación impide su baja, no puede eliminarse ese artículo dejando asociaciones sin padre. El resultado depende de las restricciones declaradas; no se deduce únicamente del WHERE.

En papel, registrar tanto las filas elegidas como si el motor admite eliminarlas. Elegir una fila mediante una clave correcta no garantiza que su borrado sea válido.

## Fuente de estudio

Enunciado del TP1 en la carpeta `../../Practicos` y filminas de cátedra `BDI_01 (Intro - creacion esquemas).pdf` (carpeta de cursada 2023).
