---
tags:
  - tp5
---

# Actualización de vistas y restricciones base

## Leer una vista y actualizarla son problemas distintos

Para una escritura sobre una vista hay que decidir:

1. Si existe traducción automática hacia las tablas base.
2. Qué columnas se pueden escribir y cuáles son calculadas.
3. Si la operación respeta PK, FK, CHECK, NOT NULL y valores por defecto.
4. Si debe satisfacer un `WITH CHECK OPTION`.

Que una vista sea actualizable no garantiza que cualquier INSERT funcione. Por ejemplo, una columna obligatoria omitida en la vista puede impedir insertar si no tiene un valor por defecto.

## Distinguir los dos criterios de la consigna

La consigna pide justificar por separado con los criterios SQL-1999 de las filminas y las condiciones de PostgreSQL. Proyectar la PK y usar un join no reciben el mismo tratamiento en ambos análisis.

[[TP5 - Comparar actualizabilidad SQL-1999 y PostgreSQL|Comparar actualizabilidad SQL-1999 y PostgreSQL]]

## Validez de la escritura concreta

La actualizabilidad de la vista y la validez de una escritura son preguntas distintas. Una operación puede fallar por obligatoriedad, unicidad o referencia aunque la vista admita ese tipo de operación.

[[TP5 - Ejemplo de INSERT rechazado en una vista actualizable|Ejemplo de INSERT rechazado en una vista actualizable]]

## Fuente de estudio

Enunciado del TP5 en la carpeta `../../Practicos` y filminas de cátedra `BDI_05 (Vistas).pdf` (carpeta de cursada 2023).
