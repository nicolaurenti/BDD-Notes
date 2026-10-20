---
tags:
  - tp4
---

# Triggers, funciones y procedimientos


| Recurso | Cómo se ejecuta | Uso |
|---|---|---|
| Restricción declarativa | La verifica el motor | PK, FK, nulidad y reglas expresables con CHECK |
| Trigger | Automáticamente ante un evento | Validación, auditoría, historial y mantenimiento de derivados |
| Función | Mediante una expresión o desde otro código | Calcular o encapsular lógica; puede modificar datos según su definición |
| Procedimiento | Mediante `CALL` | Carga inicial o proceso invocado explícitamente |

Un procedimiento no garantiza mantenimiento permanente si se puede actualizar la tabla sin llamarlo. Un trigger tampoco reconstruye automáticamente el pasado.
