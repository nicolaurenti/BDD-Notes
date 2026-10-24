---
tags:
  - tp1
---

# Actualización de filas con UPDATE


Se utiliza `UPDATE`.

### Actualizar una fila

Se filtra por una clave que identifique la fila:

```sql
UPDATE articulo
SET autor = 'Juan'
WHERE id_articulo = 1;
```

### Actualizar un subconjunto de filas

```sql
UPDATE articulo
SET autor = 'María'
WHERE autor = 'Ana';
```

### Actualizar todas las filas

```sql
UPDATE articulo
SET autor = 'Autor pendiente';
```

> [!warning]
> Un `UPDATE` sin `WHERE` afecta a todas las filas de la tabla.

## Qué determina cuántas filas se modifican

WHERE se evalúa para cada fila. Si filtra por una clave completa, selecciona como máximo una; si filtra por un atributo no único, puede seleccionar varias. «Cambiar el autor Ana» no implica una sola fila aunque hoy exista un único artículo suyo.

En una tabla con PK `(cod_p, idioma)`, `WHERE cod_p = 1` puede afectar palabras de varios idiomas. Para una sola palabra usar también `AND idioma = 'ES'`.

SET admite expresiones basadas en el valor anterior: `SET duracion = duracion + 5` incrementa cinco unidades; `SET duracion = 5` reemplaza el valor por cinco. Los valores resultantes deben seguir cumpliendo los tipos y restricciones. Un UPDATE puede seleccionar correctamente una fila y aun así rechazarse porque produce una PK repetida o una FK inexistente.

Para justificar los tres apartados del TP1, señalar el criterio de selección: sin WHERE, condición no única o clave completa. Si no coincide ninguna fila, la sentencia no modifica datos; eso no significa necesariamente un error SQL.

## Fuente de estudio

Enunciado del TP1 en la carpeta `../../Practicos` y filminas de cátedra `BDI_01 (Intro - creacion esquemas).pdf` (carpeta de cursada 2023).
