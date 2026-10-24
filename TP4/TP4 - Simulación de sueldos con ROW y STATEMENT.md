---
tags:
  - tp4
---

# TP4 - Simulación de sueldos con ROW y STATEMENT

Desarrollo del ejercicio de sueldos, con el orden supuesto y las cantidades resultantes en cada paso.

## Simulación ROW y STATEMENT del ejercicio de sueldos

El cuerpo del ejercicio resta a los empleados existentes el 5 % del sueldo mínimo. Aunque se llame autoincremento, la operación es una resta. Para interpretar código ejecutable, falta el punto y coma del UPDATE y el retorno debe ajustarse: NEW para BEFORE ROW de INSERT; NULL para STATEMENT.

Suponiendo sueldos decimales y el orden de inserción 700, 300, 700, el estado se sigue así:

| Paso BEFORE ROW | Mínimo consultado | Descuento | Sueldos después del trigger y de insertar la fila |
|---|---:|---:|---|
| Insertar id 2, sueldo 700 | 500 | 25 | id 100: 475; id 2: 700 |
| Insertar id 3, sueldo 300 | 475 | 23.75 | id 100: 451.25; id 2: 676.25; id 3: 300 |
| Insertar id 4, sueldo 700 | 300 | 15 | id 100: 436.25; id 2: 661.25; id 3: 285; id 4: 700 |

La fila nueva todavía no está en la tabla cuando corre su BEFORE ROW, por eso no recibe el descuento de su propio paso. Sí puede recibir los descuentos de pasos posteriores.

En BEFORE STATEMENT el cuerpo se ejecuta una sola vez antes de insertar las tres filas: el inicial pasa de 500 a 475 y los nuevos quedan en 700, 300 y 700. Para el cálculo en papel se declara el orden supuesto; no se lo presenta como un orden garantizado de cualquier INSERT SELECT.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
