---
tags:
  - tp3
---

# TP3 - Declaración de FK con ALTER TABLE

Ejemplos de biblioteca para identificar hija y padre, completar la plantilla y elegir acciones referenciales.

## Agregar restricciones de integridad referencial con ALTER TABLE

Cuando las tablas ya existen, se pueden incorporar claves foráneas mediante `ALTER TABLE ... ADD CONSTRAINT`. Los siguientes ejemplos usan una biblioteca: `libro`, `ejemplar` y `prestamo`.

### Identificar la tabla que contiene la referencia

La **tabla hija** contiene la FK; la **tabla padre** contiene la clave referenciada. La sentencia se escribe sobre la hija.

| Relación del ejemplo | Tabla hija y FK | Tabla padre y clave referenciada |
|---|---|---|
| Cada ejemplar pertenece a un libro | `ejemplar (isbn)` | `libro (isbn)` |
| Cada préstamo corresponde a un ejemplar | `prestamo (isbn, nro_ejemplar)` | `ejemplar (isbn, nro_ejemplar)` |

Se supone que `libro` tiene PK `(isbn)` y `ejemplar` tiene PK `(isbn, nro_ejemplar)`. Una FK también puede referenciar una clave declarada `UNIQUE` apropiada.

### Plantilla de la sentencia

```sql
-- Plantilla: sustituir los marcadores por nombres y acciones concretos.
ALTER TABLE tabla_hija
ADD CONSTRAINT nombre_de_la_restriccion
FOREIGN KEY (columna_fk_1, columna_fk_2)
REFERENCES tabla_padre (columna_clave_1, columna_clave_2)
ON DELETE accion_de_baja
ON UPDATE accion_de_modificacion;
```

`ADD CONSTRAINT` agrega una regla con nombre. Las columnas deben existir y los datos ya almacenados deben cumplir la nueva restricción. Para una FK simple se indica una sola columna en cada lista.

### Ejemplo de FK simple

Ejemplar contiene el ISBN que referencia al libro. El ejemplo impide la baja de un libro referenciado y propaga cambios de ISBN.

[[TP3 - Ejemplo de FK simple entre Ejemplar y Libro|Ejemplo de FK simple entre Ejemplar y Libro]]

### Ejemplo de FK compuesta

Préstamo referencia la pareja ISBN y número de ejemplar. Las dos columnas deben coincidir en una misma fila padre.

[[TP3 - Ejemplo de FK compuesta entre Préstamo y Ejemplar|Ejemplo de FK compuesta entre Préstamo y Ejemplar]]

### Elegir las acciones según la regla

Las acciones se toman de la regla del modelo y se escriben en el orden baja/modificación indicado por la consigna. Se comprueban además nulidad y claves completas.

[[TP3 - Elección de acciones y errores al declarar una FK|Elección de acciones y errores al declarar una FK]]

## Fuente de estudio

Enunciado del TP3 en la carpeta `../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
