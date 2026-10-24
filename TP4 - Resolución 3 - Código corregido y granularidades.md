---
tags:
  - tp4
---

# TP4 - Resolución 3 - Código corregido y granularidades

El original necesita un punto y coma al terminar UPDATE y un retorno válido. También se normaliza el identificador del trigger, que aparece con un espacio en la consigna.

```sql
-- TP4 ejercicio 3. Presupone EMPLEADO_1 y EMPLEADO_2 con sueldo NUMERIC(10,2).
CREATE OR REPLACE FUNCTION autoIncremento()
RETURNS trigger AS $$
BEGIN
    UPDATE empleado_1
    SET sueldo = sueldo - (SELECT MIN(sueldo) * 0.05 FROM empleado_1);
    IF TG_LEVEL = 'ROW' THEN
        RETURN NEW;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE 'plpgsql';

-- Elegir UNA granularidad. Para la segunda alternativa, usar FOR EACH STATEMENT.
CREATE TRIGGER trigger_autoincremento
BEFORE INSERT ON empleado_1
FOR EACH ROW
EXECUTE PROCEDURE autoIncremento();

-- Sentencia del caso trabajado; id 2, 3 y 4, en ese orden:
-- INSERT INTO empleado_1 (id_empleado, nombre, apellido, sueldo)
-- SELECT id_empleado, nombre, apellido, sueldo
-- FROM empleado_2 ORDER BY id_empleado;
```

El mismo cuerpo sirve para comparar ambas granularidades porque TG_LEVEL distingue ROW y STATEMENT. En BEFORE ROW de INSERT se devuelve NEW para que continúe la inserción; RETURN NULL la omitiría. En STATEMENT el retorno se ignora.

Para la segunda alternativa, sustituir la línea de la cabecera por:

```sql
FOR EACH STATEMENT
```

No instalar ambos triggers a la vez sobre EMPLEADO_1. Cada comparación parte otra vez del empleado inicial con 500 y de EMPLEADO_2 con los tres candidatos, no del estado que dejó la variante anterior.
