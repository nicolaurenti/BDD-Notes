---
tags:
  - tp3
---

# TP3 - Ejemplo de FK compuesta entre Préstamo y Ejemplar

Supuesto: EJEMPLAR se identifica por (isbn, nro_ejemplar); el número de ejemplar puede repetirse para libros diferentes.

## Ejemplo de FK compuesta

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

## Fuente de estudio

Enunciado del TP3 en la carpeta `../../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
