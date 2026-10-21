---
tags:
  - tp4
---

# TP4 - Ejemplo de historial de permanencias

Caso de un empleado que vuelve a un departamento: períodos, tiempo acumulado y promedio por estancia.

## Historial de permanencias

Para saber cuánto tiempo estuvo un empleado en cada departamento no alcanza con almacenar su departamento actual. Se necesitan intervalos: empleado, distribuidor, departamento, fecha de inicio y fecha de fin. La clave de departamento es `(id_distribuidor, id_departamento)`.

Al ingresar un empleado se abre un intervalo; al cambiar de departamento se cierra el anterior y se abre otro; al salir se cierra el vigente. Si estuvo 10 días en D1, 20 en D2 y luego 30 en D1, la permanencia acumulada de D1 es 40 y el promedio por estancia allí es 20. «Promedio por departamento» necesita definir si promedia estancias o usa otra unidad.

Un período abierto aumenta con el tiempo aunque no se modifiquen filas. Guardar hoy una duración y esperar que un trigger de UPDATE la incremente mañana no funciona. Conservar fechas permite calcular la duración al consultar, o un procedimiento puede recalcularla cuando se lo invoca. También hay que decidir si el intervalo abierto participa en el promedio.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
