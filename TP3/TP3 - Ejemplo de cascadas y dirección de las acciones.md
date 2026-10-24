---
tags:
  - tp3
---

# TP3 - Ejemplo de cascadas y dirección de las acciones

Libro, ejemplares y préstamos permiten seguir una cadena de acciones y una posible restricción que la bloquee.

## Seguir las acciones en la dirección correcta

Supongamos un libro L1 con ejemplares `(L1, 1)` y `(L1, 2)`. Si EJEMPLAR referencia LIBRO con ON DELETE CASCADE, borrar L1 solicita eliminar ambos ejemplares. Borrar solamente el ejemplar `(L1, 1)` no elimina el libro: la acción declarada se propaga desde la clave padre hacia sus referencias hijas.

Con ON UPDATE CASCADE, cambiar la clave del libro L1 a L9 transforma las referencias de sus ejemplares en `(L9, 1)` y `(L9, 2)`. En cambio, `UPDATE ejemplar SET isbn = 'L9' ...` es una modificación directa de la hija: necesita que exista L9 en LIBRO. No invoca una cascada para crear ese libro.

Si PRESTAMO referencia EJEMPLAR con RESTRICT, un préstamo puede impedir que se complete la eliminación en cascada que comenzó en LIBRO. Se revisa toda la cadena, no solo la primera FK. Si se rechaza la sentencia, no queda borrado el libro con sus ejemplares a medias.

## Fuente de estudio

Enunciado del TP3 en la carpeta `../../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
