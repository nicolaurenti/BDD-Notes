---
tags:
  - tp4
---

# TP4 - Ejemplo de cambio de tarea y variación de horas

Desarrollo del ejercicio 5: reiniciar horas al cambiar tarea y comprobar una variación interpretada como margen de ±10 %. Se explicitan los supuestos.

## Comparar valores anteriores y propuestos

Para un cambio real, se comparan OLD y NEW. `UPDATE OF id_tarea` indica que la columna figura en SET; no asegura que su valor sea distinto. `IS DISTINCT FROM` permite detectar diferencia incluso con NULL: ambos nulos no son distintos; solo uno nulo sí.

En el ejercicio 5, cambiar de tarea debe colocar horas en cero aunque la sentencia proponga otra cantidad. Se modifica NEW en BEFORE: así se guarda directamente la fila corregida, sin emitir otro UPDATE sobre la misma tabla.

```sql
CREATE OR REPLACE FUNCTION controlar_cambio_voluntario()
RETURNS trigger AS $$
BEGIN
    IF NEW.id_tarea IS DISTINCT FROM OLD.id_tarea THEN
        NEW.horas_aportadas := 0;
    ELSIF NEW.horas_aportadas IS DISTINCT FROM OLD.horas_aportadas THEN
        IF NEW.horas_aportadas IS NULL
           OR OLD.horas_aportadas IS NULL THEN
            RAISE EXCEPTION 'No se puede calcular la variacion con horas nulas';
        END IF;
        IF NEW.horas_aportadas < OLD.horas_aportadas * 0.90
           OR NEW.horas_aportadas > OLD.horas_aportadas * 1.10 THEN
            RAISE EXCEPTION 'La variacion supera el 10 por ciento';
        END IF;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER tr_cambio_voluntario
BEFORE UPDATE ON voluntario
FOR EACH ROW
EXECUTE PROCEDURE controlar_cambio_voluntario();
```

El segundo requisito tiene una redacción ambigua. Este ejemplo declara la interpretación «variación máxima de ±10 % respecto del valor anterior», con horas no negativas, y rechaza cambios de horas si no se puede calcular la variación por un nulo. No se atribuye esa política de nulos al enunciado.

Con 100 horas, 90 y 110 pasan; 89 y 111 no. Si cambia la tarea y la sentencia propone 105 horas, se guarda 0: el primer IF tiene prioridad y el ELSIF evita rechazar ese cero obligatorio por la regla porcentual. Con horas anteriores 0, un margen del 10 % solo permite 0.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
