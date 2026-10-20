---
tags:
  - tp8
---

# Bloqueos compartidos y exclusivos


| Bloqueos sobre el mismo ítem | Compatible |
|---|---|
| S con S | Sí |
| S con X | No |
| X con X | No |

S permite leer; X permite escribir. En el práctico aparecen `r-lock` y `s-lock` como bloqueo compartido, y `w-lock` como exclusivo.

Un acceso incompatible espera hasta que se libere el bloqueo. Al simular, una transacción bloqueada no puede seguir ejecutando instrucciones posteriores como si hubiera adquirido el recurso.
