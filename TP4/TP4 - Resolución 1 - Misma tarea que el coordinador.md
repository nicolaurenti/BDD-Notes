---
tags:
  - tp4
---

# TP4 - Resolución 1 - Misma tarea que el coordinador

Se implementa 5.d. Se buscan parejas coordinado–coordinador con tareas diferentes y se exige que no exista ninguna.

```sql
-- TP4, ejercicio 1: TP3 5.d.
CREATE OR REPLACE FUNCTION tp4_controlar_tarea_coordinador()
RETURNS trigger AS $$
BEGIN
    IF EXISTS (
        SELECT 1
        FROM voluntario v JOIN voluntario c
          ON c.nro_voluntario = v.id_coordinador
        WHERE v.id_tarea <> c.id_tarea
    ) THEN
        RAISE EXCEPTION 'RI_5D: voluntario y coordinador realizan tareas diferentes';
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER tp4_tarea_coordinador
AFTER INSERT OR UPDATE OF id_tarea, id_coordinador ON voluntario
FOR EACH STATEMENT
EXECUTE PROCEDURE tp4_controlar_tarea_coordinador();
```

El AFTER STATEMENT comprueba también los coordinados de una persona que cambió de tarea. Si una sola sentencia cambia de T1 a T2 al coordinador y todos sus coordinados, el estado final puede ser válido: no se rechaza por observar una fila intermedia.

`id_tarea` es NOT NULL en el esquema proporcionado. Un voluntario sin coordinador no forma una pareja en la unión; no se le inventa un coordinador ni se rechaza su ausencia por esta regla.

[[TP4 - Resolución 1 - Pruebas de tarea del coordinador|Pruebas de cambio de tarea y reasignación de coordinador]].

Fuente: `Practico 4 - Acciones Procedurales.docx`, TP3 y esquema correspondiente de la cátedra. Se usa `EXECUTE PROCEDURE`.
