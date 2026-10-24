---
tags:
  - tp3
---

# TP3 - Ejemplo de FK simple entre Ejemplar y Libro

Supuesto: LIBRO tiene PK(isbn) y cada EJEMPLAR almacena el ISBN del libro al que pertenece.

## Ejemplo de FK simple

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

## Fuente de estudio

Enunciado del TP3 en la carpeta `../../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
