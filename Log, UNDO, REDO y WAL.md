---
tags:
  - tp8
---

# Log, UNDO, REDO y WAL


Registro didáctico de escritura: `<T, ítem, valor_anterior, valor_nuevo>`.

- **UNDO:** restaurar el valor anterior de una escritura no confirmada; recorrer las operaciones que se deshacen en orden inverso.
- **REDO:** reproducir efectos confirmados que podrían no haber llegado al almacenamiento de datos; respetar el orden de las escrituras que se rehacen.
- **WAL:** el registro de log necesario debe persistirse antes de escribir la página de datos correspondiente. La confirmación durable exige que el log pertinente esté asegurado.

No confundir «se escribió un registro en el log» con «la página de datos ya se guardó». [Write-Ahead Logging](https://www.postgresql.org/docs/current/wal-intro.html).
