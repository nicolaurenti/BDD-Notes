---
tags:
  - tp8
---

# Log, UNDO, REDO y WAL


Registro didáctico de escritura: `<T, ítem, valor_anterior, valor_nuevo>`.

- **UNDO:** restaurar el valor anterior de una escritura no confirmada; recorrer las operaciones que se deshacen en orden inverso.
- **REDO:** reproducir efectos confirmados que podrían no haber llegado al almacenamiento de datos; respetar el orden de las escrituras que se rehacen.
- **WAL:** el registro de log necesario debe persistirse antes de escribir la página de datos correspondiente. La confirmación durable exige que el log pertinente esté asegurado.

No confundir «se escribió un registro en el log» con «la página de datos ya se guardó».

## Seguir las escrituras del log

UNDO restaura el valor anterior en orden inverso; REDO reproduce el nuevo en orden original. Aplicar valores registrados evita repetir cálculos como si fueran operaciones nuevas.

[[TP8 - Ejemplo de UNDO y REDO sobre valores registrados|Ejemplo de UNDO y REDO sobre valores registrados]]

## Log durable y página de datos son cosas diferentes

Una escritura puede estar registrada en log estable mientras la página modificada sigue en memoria. Después de la falla se perdió esa memoria, pero se puede reconstruir el cambio desde el registro. A la inversa, si se permite escribir una página antes de confirmar, el log debe contener previamente la información necesaria para deshacerla.

WAL establece esa precedencia del registro sobre la página correspondiente. El COMMIT durable necesita que el log pertinente esté asegurado; no exige en todas las estrategias que ya estén escritas todas las páginas de datos.

## Tipo de falla y respaldo

La recuperación del TP supone que el log disponible sobrevive a una falla del sistema. Una falla que destruye almacenamiento puede exigir restaurar un backup y aplicar registros posteriores. Un checkpoint es un punto de control del procesamiento, no una copia de seguridad completa. Las filminas distinguen backups completos e incrementales; no usarlos como sinónimos de UNDO o REDO.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
