---
tags:
  - tp5
---

# TP5 - Ejemplo de INSERT rechazado en una vista actualizable

Caso de EMPLEADO_DIST_20 con una columna base obligatoria que la vista no proyecta.

## Una vista actualizable puede rechazar un INSERT

Supongamos EMPLEADO_DIST_20 con id, nombre, apellido, sueldo y nacimiento, pero sin la columna id_distribuidor, y que esta es NOT NULL sin DEFAULT en EMPLEADO. La vista puede permitir modificar el sueldo de una fila existente, pero un INSERT no aporta el distribuidor y se rechaza por esa obligatoriedad. El filtro `id_distribuidor = 20` no asigna automáticamente 20 a una columna omitida.

También puede rechazarse un INSERT porque la PK ya existe o una FK referencia un padre inexistente. La respuesta completa distingue **estructura de la vista** y **validez de la operación concreta**. «Es actualizable» contesta solo la primera parte.

## Fuente de estudio

Enunciado del TP5 en la carpeta `../../Practicos` y filminas de cátedra `BDI_05 (Vistas).pdf` (carpeta de cursada 2023).
