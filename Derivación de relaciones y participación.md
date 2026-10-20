---
tags:
  - tp1
---

# Derivación de relaciones y participación


| Relación | Representación |
|---|---|
| **1:N** | La clave de la tabla del lado 1 se incorpora como FK en la tabla del lado N. |
| **N:M** | Se crea una tabla intermedia con las claves de ambas entidades como FK. Habitualmente, su combinación es la PK. |
| **1:1** | Puede representarse con una FK que además tenga `UNIQUE`. Su ubicación depende de la participación y del diseño elegido. |

Los **atributos propios de una relación** se guardan donde se representa esa relación.

### Participación obligatoria y opcional

- Si una fila debe tener una referencia, la FK lleva `NOT NULL`.
- Si la referencia es opcional, puede admitir `NULL`.

> [!important]
> Una FK no garantiza que cada fila de la tabla referenciada tenga filas relacionadas. Por ejemplo, una FK de empleado a departamento no obliga a que todos los departamentos tengan empleados.

### Ejemplos del práctico

- `ARTICULO` y `PALABRA`: relación N:M representada por `CONTIENE`.
- `CURSO` y `ALUMNO`: relación N:M representada por `INSCRIPTO`.
- `EQUIPO` y `GRAB_COMERCIAL`: relación 1:N; la FK `nro_equipo` queda en `GRAB_COMERCIAL`.
