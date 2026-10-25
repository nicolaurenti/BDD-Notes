---
tags:
  - tp8
---

# Log, UNDO, REDO, WAL y checkpoints

Registro didáctico de escritura: `<T, ítem, valor_anterior, valor_nuevo>`.

- **UNDO:** restaurar el valor anterior de una escritura no confirmada; recorrer las operaciones que se deshacen en orden inverso.
- **REDO:** reproducir efectos confirmados que podrían no haber llegado al almacenamiento de datos; respetar el orden de las escrituras que se rehacen.
- **WAL:** el registro de log necesario debe persistirse antes de escribir la página de datos correspondiente. La confirmación durable exige que el log pertinente esté asegurado.

No confundir «se escribió un registro en el log» con «la página de datos ya se guardó».

## Seguir las escrituras del log

UNDO restaura el valor anterior en orden inverso; REDO reproduce el nuevo en orden original. Aplicar valores registrados evita repetir cálculos como si fueran operaciones nuevas.

### Ejemplo de UNDO y REDO sobre valores registrados

Dos escrituras sucesivas sobre A permiten comprobar el orden correcto de recuperación.

#### Elegir el valor correcto del registro

Con `<T1, A, 10, 15>`, UNDO escribe 10 y REDO escribe 15. No se vuelve a ejecutar «sumar cinco»: se restaura o reproduce el valor registrado. Esto permite que repetir una acción de recuperación tenga el mismo efecto que aplicarla una vez, propiedad de idempotencia que explican las filminas.

Si una transacción escribió A de 10 a 15 y después de 15 a 20, deshacer en orden inverso restaura primero 15 y luego 10. Hacerlo al revés dejaría 15, que no es el estado anterior a la transacción. Para rehacer se respeta el orden original y se termina en 20.

## Log durable y página de datos son cosas diferentes

Una escritura puede estar registrada en log estable mientras la página modificada sigue en memoria. Después de la falla se perdió esa memoria, pero se puede reconstruir el cambio desde el registro. A la inversa, si se permite escribir una página antes de confirmar, el log debe contener previamente la información necesaria para deshacerla.

WAL establece esa precedencia del registro sobre la página correspondiente. El COMMIT durable necesita que el log pertinente esté asegurado; no exige en todas las estrategias que ya estén escritas todas las páginas de datos.

## Tipo de falla y respaldo

La recuperación del TP supone que el log disponible sobrevive a una falla del sistema. Una falla que destruye almacenamiento puede exigir restaurar un backup y aplicar registros posteriores. Un checkpoint es un punto de control del procesamiento, no una copia de seguridad completa. Las filminas distinguen backups completos e incrementales; no usarlos como sinónimos de UNDO o REDO.

## Checkpoints y transacciones activas

El checkpoint del modelo de la cátedra registra un punto en que se fuerzan a disco los efectos de las transacciones ya confirmadas. Las filminas describen suspender temporalmente el procesamiento, escribir esos efectos y el registro de control, y reanudarlo. Se usa ese modelo para resolver los gráficos y logs del TP8.

### Qué garantiza y qué no

No confirma transacciones activas. Si T1 había empezado y todavía no tenía COMMIT, sigue activa después del checkpoint. Una escritura de T1 anterior al control no queda automáticamente fuera del análisis: la decisión se toma por la situación de **T1**, no solo por la posición de esa escritura.

Para UNDO/REDO, bajo ese modelo:

| Situación | Acción |
|---|---|
| Confirmada antes del checkpoint, efectos asegurados | No necesita recuperación adicional |
| Confirmada después del checkpoint y antes de la falla | REDO de sus efectos que corresponda recuperar |
| No confirmada al fallar, con posibilidad de páginas escritas | UNDO |

La estrategia puede quitar alguna necesidad: en diferida no se hace UNDO de datos persistidos no confirmados; con FORCE al COMMIT no se necesita REDO de confirmadas.

### Ejemplo temporal

El gráfico separa las transacciones que confirmaron antes del control, después de él y las que quedaron activas en la falla.

#### Ejemplo temporal de checkpoint y falla

Línea de tiempo bajo el modelo de checkpoint de la cátedra y la estrategia UNDO/REDO.

```text
T1: START -- escritura -- |CHECKPOINT| -- COMMIT -- |FALLA|
T2: START -- COMMIT ----- |CHECKPOINT| ------------ |FALLA|
T3:                      |CHECKPOINT| -- START --- |FALLA|
```

T2 quedó asegurada por el checkpoint. T1 confirma después y se considera para REDO en UNDO/REDO, incluyendo las escrituras suyas que no quedaron aseguradas por el control. T3 no confirma y se considera para UNDO si pudo escribir páginas.

Para dibujar el gráfico, marcar START y COMMIT de cada transacción sobre una línea de tiempo común, y luego las dos verticales de checkpoint y falla. Una línea abierta en la falla no se transforma en confirmada porque haya comenzado hace mucho.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
