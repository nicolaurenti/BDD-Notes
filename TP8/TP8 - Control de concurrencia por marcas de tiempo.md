---
tags:
  - tp8
---

# TP8 - Control de concurrencia por marcas de tiempo

Desarrollo del criterio de ordenamiento, las marcas por dato y los reinicios ante operaciones incompatibles.

## Marcas de tiempo en las filminas

El control por timestamps ordena lógicamente las transacciones por su marca temporal. Para cada dato conserva marcas de lectura y escritura y rechaza una operación que contradiga el orden permitido. Por ejemplo, si una transacción más nueva ya escribió X, una lectura tardía de X por otra más antigua no puede aceptarse como si respetara el orden temporal. Ese enfoque evita la espera circular propia de los bloqueos, pero puede provocar reinicios y falta de progreso si se repiten conflictos. No confundir timestamp de ordenamiento con fecha del dato guardado.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
