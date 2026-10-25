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

### Ejemplo de FK simple entre Ejemplar y Libro

Supuesto: LIBRO tiene PK(isbn) y cada EJEMPLAR almacena el ISBN del libro al que pertenece.

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

### Ejemplo de FK compuesta

Préstamo referencia la pareja ISBN y número de ejemplar. Las dos columnas deben coincidir en una misma fila padre.

### Ejemplo de FK compuesta entre Préstamo y Ejemplar

Supuesto: EJEMPLAR se identifica por (isbn, nro_ejemplar); el número de ejemplar puede repetirse para libros diferentes.

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

### Elegir las acciones según la regla

Las acciones se toman de la regla del modelo y se escriben en el orden baja/modificación indicado por la consigna. Se comprueban además nulidad y claves completas.

### Elección de acciones y errores al declarar una FK

Equivalencia entre la notación R/C/N del práctico y las cláusulas SQL, con errores frecuentes.

Las acciones anteriores son decisiones de los ejemplos. En otro modelo podrían corresponder otras acciones:

```sql
-- Fragmentos alternativos para la declaración de una FK:
ON DELETE CASCADE
ON UPDATE RESTRICT
```

`CASCADE` al borrar elimina las filas hijas que referencian al padre. Con `SET NULL`, en cambio, se conservan esas filas y se ponen en NULL las columnas de la FK; esas columnas deben admitir nulos.

Cuando una consigna usa un par **[baja, modificación a derecha]**, el primer componente corresponde a `ON DELETE` y el segundo a `ON UPDATE`. La modificación a derecha se refiere al cambio de la clave referenciada en la tabla padre.

| Símbolo | Acción SQL |
|---|---|
| `R` | `RESTRICT` |
| `C` | `CASCADE` |
| `N` | `SET NULL` |

Las acciones se evalúan junto con las demás restricciones: una cascada puede quedar impedida por otra regla. Si se omite `MATCH`, PostgreSQL usa `MATCH SIMPLE`.

#### Errores frecuentes

- Alterar la tabla padre en lugar de la hija.
- Invertir las acciones de baja y modificación.
- Usar solo una parte de una clave compuesta o cambiar el orden de correspondencia.
- Confundir la PK completa de la hija con las columnas que necesita una FK concreta.
- Creer que `ON UPDATE CASCADE` permite asignar directamente a una hija una FK cuyo padre no existe.

Los ejemplos usan la forma ALTER TABLE del práctico; no se ejecutaron sobre las bases existentes.

## Fuente de estudio

Enunciado del TP3 en la carpeta `../../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
