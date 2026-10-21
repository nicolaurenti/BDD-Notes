---
tags:
  - tp4
---

# Auditoría, historial y resúmenes con triggers

Estos usos responden preguntas distintas. Una **auditoría** registra quién hizo una operación y cuándo. Un **historial** conserva estados o períodos anteriores. Una **tabla resumen** guarda cantidades o fechas calculadas a partir de otros datos. Elegir el propósito determina qué columnas y eventos hacen falta.

## Auditoría por fila

Una auditoría ROW registra una entrada por fila afectada. Debe identificar operación, fecha, usuario y tabla de origen; cada ejecución representa una fila.

[[TP4 - Ejemplo de auditoría por fila en HIS_ENTREGA|Ejemplo de auditoría por fila en HIS_ENTREGA]]

## Qué cambia con STATEMENT

STATEMENT registra una actuación por sentencia. La cantidad de filas afectadas requiere un mecanismo de conteo adicional: no se obtiene cambiando la cabecera del trigger ROW.

[[TP4 - Auditoría STATEMENT y conteo de filas afectadas|Auditoría STATEMENT y conteo de filas afectadas]]

## Historial de permanencias

Un historial conserva intervalos y no solo el estado actual. Las duraciones de períodos abiertos crecen aunque no ocurran escrituras, por lo que conviene conservar fechas y definir cómo se calcula el promedio.

[[TP4 - Ejemplo de historial de permanencias|Ejemplo de historial de permanencias]]

## Resúmenes y datos preexistentes

La carga inicial reconstruye el resumen de los datos existentes. Su mantenimiento posterior debe contemplar inserciones, bajas y cambios de autor o fecha; instalar un trigger no reconstruye el pasado.

[[TP4 - Ejemplo de carga y mantenimiento de TextosporAutor|Ejemplo de carga y mantenimiento de TextosporAutor]]

## Fuente de estudio

Enunciado del TP4 en la carpeta `../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
