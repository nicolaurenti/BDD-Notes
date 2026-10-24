---
tags:
  - tp8
---

# TP8 - Ejemplo de UNDO y REDO sobre valores registrados

Dos escrituras sucesivas sobre A permiten comprobar el orden correcto de recuperación.

## Elegir el valor correcto del registro

Con `<T1, A, 10, 15>`, UNDO escribe 10 y REDO escribe 15. No se vuelve a ejecutar «sumar cinco»: se restaura o reproduce el valor registrado. Esto permite que repetir una acción de recuperación tenga el mismo efecto que aplicarla una vez, propiedad de idempotencia que explican las filminas.

Si una transacción escribió A de 10 a 15 y después de 15 a 20, deshacer en orden inverso restaura primero 15 y luego 10. Hacerlo al revés dejaría 15, que no es el estado anterior a la transacción. Para rehacer se respeta el orden original y se termina en 20.

## Fuente de estudio

Enunciado del TP8 en la carpeta `../../Practicos` y filminas de cátedra `BDI_08 (Proc_Transacciones).pdf` (carpeta de cursada 2023).
