---
tags:
  - tp3
---

# TP3 - Ejemplo de restricción general entre Voluntario y Tarea

La consulta detecta infractores y la restricción exige que no exista ninguno.

## 4. Restricción general

Relaciona datos de **varias tablas**.

«Las horas aportadas por cada voluntario deben estar entre el mínimo y el máximo de su tarea».

```sql
-- SQL estándar: PostgreSQL NO implementa CREATE ASSERTION.
CREATE ASSERTION horas_dentro_de_la_tarea
CHECK (
    NOT EXISTS (
        SELECT 1
        FROM voluntario v
        JOIN tarea t ON t.id_tarea = v.id_tarea
        WHERE v.horas_aportadas < t.min_horas
           OR v.horas_aportadas > t.max_horas
    )
);
```

La subconsulta busca **infractores**. La restricción exige que no encuentre ninguno.

Si la tarea permite entre 100 y 500 horas:

| Horas del voluntario | Resultado |
|---:|---|
| 100 | Aceptado |
| 350 | Aceptado |
| 500 | Aceptado |
| 600 | Rechazado |

Este ejemplo supone que horas y límites no admiten NULL y que una FK garantiza la tarea referenciada. Si esos supuestos no se cumplen, definir cómo tratar valores ausentes: ni el JOIN ni las comparaciones detectan automáticamente todos esos casos como infracción.

**También hay que controlar cambios en tarea:** si un voluntario aporta 350 horas y se baja el máximo de su tarea a 300, la base quedaría inválida aunque no se modifique la fila del voluntario.

Un CHECK local sobre voluntario no alcanza: los límites están en otra tabla. En el apartado de implementación en PostgreSQL del TP3, indicar esa limitación en lugar de presentar como ejecutable una ASSERTION.

## Fuente de estudio

Enunciado del TP3 en la carpeta `../Practicos` y filminas de cátedra `BDI_03.P1 (Restricciones-1).pdf; BDI_03.P1 (Restricciones-2).pdf` (carpeta de cursada 2023).
