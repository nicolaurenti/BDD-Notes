---
tags:
  - tp4
---

# TP4 - Resolución 3 - Simulación paso a paso

Se supone NUMERIC(10,2) y orden de inserción 700, 300, 700. Antes de comenzar, solo existe id 100 con sueldo 500.

## BEFORE ROW

| Paso | Mínimo antes del trigger | Descuento a cada fila ya presente | Estado después del UPDATE y de insertar la nueva fila |
|---|---:|---:|---|
| Insertar id 2 con 700 | 500 | 25 | 100: 475; 2: 700 |
| Insertar id 3 con 300 | 475 | 23.75 | 100: 451.25; 2: 676.25; 3: 300 |
| Insertar id 4 con 700 | 300 | 15 | 100: 436.25; 2: 661.25; 3: 285; 4: 700 |

Primer paso: 5 % de 500 es 25. Solo se descuenta al empleado inicial porque id 2 todavía no fue insertado.

Segundo paso: el mínimo es 475, no 300. El sueldo 300 pertenece a la fila que se insertará después del BEFORE. El descuento 23.75 afecta a los empleados 100 y 2.

Tercer paso: ahora sí existe el sueldo 300 y es el mínimo. El descuento 15 afecta a 100, 2 y 3. Id 4 se incorpora luego y conserva 700.

## BEFORE STATEMENT

El mínimo inicial es 500. Hay una única ejecución del cuerpo, que descuenta 25 al empleado 100 y lo deja en 475. Después se insertan los tres nuevos con sus valores originales: 700, 300 y 700.

No se recalcula un descuento por cada fila en STATEMENT. Si el tipo de sueldo fuera entero o cambiara el orden de inserción, habría que revisar la simulación; no se trasladan estos resultados decimales sin declarar los supuestos.
