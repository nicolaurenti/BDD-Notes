---
tags:
  - tp4
---

# TP4 - Resolución 1 - Horas de voluntarios y tareas

El mismo cuerpo comprueba 5.b y 5.c. La unión de VOLUNTARIO consigo misma diferencia el rol de coordinado (`v`) y coordinador (`c`). La unión con TAREA obtiene los límites que no están en la fila del voluntario.

```sql
-- TP4, ejercicio 1: TP3 5.b y 5.c.
-- Se comprueba el estado final de la sentencia, incluidas las filas coordinadas.
CREATE OR REPLACE FUNCTION tp4_controlar_horas()
RETURNS trigger AS $$
BEGIN
    IF EXISTS (
        SELECT 1
        FROM voluntario v JOIN voluntario c
          ON c.nro_voluntario = v.id_coordinador
        WHERE v.horas_aportadas > c.horas_aportadas
    ) THEN
        RAISE EXCEPTION 'RI_5B: un voluntario supera las horas de su coordinador';
    END IF;

    IF EXISTS (
        SELECT 1
        FROM voluntario v JOIN tarea t ON t.id_tarea = v.id_tarea
        WHERE v.horas_aportadas < t.min_horas
           OR v.horas_aportadas > t.max_horas
    ) THEN
        RAISE EXCEPTION 'RI_5C: horas fuera de los limites de la tarea';
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER tp4_horas_voluntario
AFTER INSERT OR UPDATE ON voluntario
FOR EACH STATEMENT
EXECUTE PROCEDURE tp4_controlar_horas();

CREATE TRIGGER tp4_horas_tarea
AFTER UPDATE OF min_horas, max_horas ON tarea
FOR EACH STATEMENT
EXECUTE PROCEDURE tp4_controlar_horas();
```

No alcanza con comprobar a quien cambia sus horas: reducir las del coordinador puede invalidar a sus coordinados. Tampoco alcanza con un trigger sobre VOLUNTARIO: reducir el máximo de TAREA puede invalidar personas sin modificar sus filas.

La grilla identifica las columnas críticas. El trigger sobre VOLUNTARIO escucha INSERT y cualquier UPDATE como cobertura simple; puede ejecutar una comprobación innecesaria, pero cubre todos los casos críticos.

Si horas o límites son NULL, la comparación no produce una infracción TRUE. Eso conserva la semántica declarativa vista en TP3; no se agrega una prohibición de nulos que no pide esta regla. Las FK comprueban la existencia de tarea y coordinador.

[[TP4 - Resolución 1 - Pruebas de voluntarios|Pruebas de inserciones, cambios del coordinador y límites de tarea]].

Fuente: `Practico 4 - Acciones Procedurales.docx`, TP3 y esquema correspondiente de la cátedra. Se usa `EXECUTE PROCEDURE`.
