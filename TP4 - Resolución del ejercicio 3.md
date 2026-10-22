---
tags:
  - tp4
---

# TP4 - Resolución del ejercicio 3

El nombre autoincremento no describe el cuerpo: el UPDATE **resta el 5 % del sueldo mínimo** a las filas que ya están en EMPLEADO_1.

Se toma el sueldo como NUMERIC(10,2), un empleado inicial de id 100 con sueldo 500 y las inserciones de id 2/700, id 3/300 e id 4/700, en ese orden.

| id_empleado | BEFORE ROW | BEFORE STATEMENT |
|---:|---:|---:|
| 100 | 436.25 | 475.00 |
| 2 | 661.25 | 700.00 |
| 3 | 285.00 | 300.00 |
| 4 | 700.00 | 700.00 |

## Desarrollo

- [[TP4 - Resolución 3 - Código corregido y granularidades|Correcciones sintácticas, retorno y selección de granularidad]].
- [[TP4 - Resolución 3 - Simulación paso a paso|Seguimiento del mínimo, descuentos y estado después de cada paso]].

En ROW, cada fila insertada dispara antes su propia ejecución; la nueva fila todavía no está en la tabla cuando se aplica ese descuento. En STATEMENT, el cuerpo se ejecuta una sola vez antes de las tres inserciones.

Los resultados se verificaron ejecutando ambas variantes en esquemas temporales separados. Un INSERT SELECT sin orden no permite asegurar ese recorrido de filas; el caso trabajado explicita el orden esperado por la simulación.

Fuente: `Practico 4 - Acciones Procedurales.docx`, TP3 y esquema correspondiente de la cátedra. Se usa `EXECUTE PROCEDURE`.
