---
tags:
  - tp4
---

# Reinicio de horas y control de variación


Este ejemplo cubre el primer requisito del ejercicio 5:

```sql
CREATE OR REPLACE FUNCTION reiniciar_horas_tarea()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    IF NEW.id_tarea IS DISTINCT FROM OLD.id_tarea THEN
        NEW.horas_aportadas := 0;
    END IF;
    RETURN NEW;
END;
$$;

CREATE TRIGGER tr_reiniciar_horas_tarea
BEFORE UPDATE OF id_tarea ON voluntario
FOR EACH ROW
EXECUTE FUNCTION reiniciar_horas_tarea();
```

`IS DISTINCT FROM` detecta cambios incluso si interviene NULL. `UPDATE OF` se activa cuando la columna figura en el SET, aunque su valor no cambie: el IF verifica el cambio real.

Modificar `NEW` evita ejecutar otro UPDATE sobre la misma tabla, que puede producir recursión. Las filminas usan `EXECUTE PROCEDURE` en cabeceras de trigger; la forma moderna es `EXECUTE FUNCTION`, y el objeto llamado sigue siendo una función `RETURNS trigger`.

El segundo requisito del ejercicio 5 tiene redacción ambigua. Si se interpreta como «la variación máxima es ±10 %», para horas anteriores no negativas:

```sql
-- Fragmento dentro de la función, con política de nulidad definida.
IF NEW.horas_aportadas < OLD.horas_aportadas * 0.90
   OR NEW.horas_aportadas > OLD.horas_aportadas * 1.10 THEN
    RAISE EXCEPTION 'La variación de horas supera el 10 por ciento';
END IF;
```

Explicitar la interpretación. Si cambia tarea, priorizar el reinicio con `IF ... ELSIF ...`: de otro modo el control porcentual podría rechazar el cero obligatorio. Si las horas anteriores son cero, ±10 % solo admite cero. Controlar nulos explícitamente si no se permiten.
