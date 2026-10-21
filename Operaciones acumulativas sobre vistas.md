---
tags:
  - tp5
---

# Operaciones acumulativas sobre vistas


Para cada sentencia, registrar: aceptada/rechazada, tabla base afectada, filas visibles en cada vista y efecto sobre la sentencia siguiente. Un INSERT puede fallar por PK duplicada aunque pase los filtros. En ejercicios con edades, indicar la fecha de evaluación: `AGE(fecha_nacimiento)` depende de ella.

## Registrar tres estados diferentes

Después de cada sentencia conviene llevar una tabla con: aceptación o motivo de rechazo, filas de la tabla base y filas visibles en cada vista. Así se evita confundir «no aparece en la vista» con «no fue insertada».

Supongamos DISTRIBUIDOR con claves 1049 y 1050, y una vista `id_distribuidor > 1000`, sin WCO. Considerando válidas las demás columnas:

| Sentencia | Base después de la sentencia | Vista después de la sentencia |
|---|---|---|
| Insertar 1048 | 1048, 1049, 1050 | 1048, 1049, 1050 |
| Insertar 990 | 990, 1048, 1049, 1050 | 1048, 1049, 1050 |
| Insertar 1050 | Sin cambios: PK repetida | Sin cambios |

La segunda inserción es válida aunque quede oculta. La tercera falla por unicidad, no por el filtro ni por falta de actualizabilidad. Si hubiera WCO, la segunda se rechazaría y 990 tampoco aparecería en la base.

Para las vistas anidadas con horas y edad, agregar a la tabla la visibilidad en cada nivel. Una inserción rechazada no reserva una PK: una sentencia posterior con la misma clave puede ser válida si el identificador todavía no existe. Si la edad se calcula con AGE, declarar la fecha tomada para evaluar el ejercicio, porque el resultado depende de cuándo se calcula.

## Fuente de estudio

Enunciado del TP5 en la carpeta `../Practicos` y filminas de cátedra `BDI_05 (Vistas).pdf` (carpeta de cursada 2023).
