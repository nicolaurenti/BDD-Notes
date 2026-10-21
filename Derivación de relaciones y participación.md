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

## Ejemplo razonado de cada cardinalidad

En una relación **1:N** entre equipo y grabación comercial, un equipo puede aparecer en muchas grabaciones, pero cada grabación referencia un equipo. Por eso `nro_equipo` queda en `GRAB_COMERCIAL`. Poner una única FK de grabación en `EQUIPO` permitiría registrar solo una de sus grabaciones.

En una relación **N:M** entre artículo y palabra, ninguna de las dos tablas puede contener una sola FK que represente todas las asociaciones. Se crea `CONTIENE(id_articulo, cod_p, idioma)`: sus dos referencias indican qué artículo contiene qué palabra. Su PK combina las claves de ambos extremos e impide repetir la misma asociación. Si la relación tuviera un atributo, como `relevancia`, iría en CONTIENE, porque depende de la pareja artículo–palabra.

En una relación **1:1**, una FK sin unicidad no alcanza. Si `persona.id_pasaporte` referencia PASAPORTE, dos personas podrían señalar el mismo pasaporte. Agregar `UNIQUE(id_pasaporte)` limita esa referencia a una persona. `NOT NULL` obliga a que cada persona tenga pasaporte, pero no obliga a que todos los pasaportes estén asignados.

## Cardinalidad máxima y participación mínima

Son preguntas diferentes: «¿cuántos como máximo?» y «¿puede no tener ninguno?». Una FK obligatoria resuelve la segunda pregunta para la fila que la contiene. No impone la participación mínima del extremo referenciado. Al justificar la derivación, explicar qué asegura cada restricción y qué condición del modelo queda sin representar con PK, FK, UNIQUE y NOT NULL.

## Fuente de estudio

Enunciado del TP1 en la carpeta `../Practicos` y filminas de cátedra `BDI_01 (Intro - creacion esquemas).pdf` (carpeta de cursada 2023).
