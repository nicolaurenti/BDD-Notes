---
tags:
  - tp4
---

# TP4 - Anatomía de una función y su trigger

Un control de horas no negativas permite identificar cada parte y seguir un caso válido, uno inválido y uno nulo.

## Trigger, función y stored procedure

En la forma usada por el enunciado hay dos objetos: una función PL/pgSQL que contiene el cuerpo y un trigger que la asocia con tabla, eventos, momento y granularidad.

```sql
CREATE OR REPLACE FUNCTION controlar_horas_no_negativas()
RETURNS trigger AS $$
BEGIN
    IF NEW.horas_aportadas < 0 THEN
        RAISE EXCEPTION 'Las horas no pueden ser negativas';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER tr_controlar_horas
BEFORE INSERT OR UPDATE ON voluntario
FOR EACH ROW
EXECUTE PROCEDURE controlar_horas_no_negativas();
```

Se mantiene **EXECUTE PROCEDURE**, tal como aparece en la plantilla del TP4. El cuerpo sigue siendo una función que declara RETURNS trigger. La cabecera no invoca un procedimiento creado con CREATE PROCEDURE.

Este control simple también podría declararse con CHECK; se usa aquí para mostrar las partes de un trigger. Con horas 30 continúa la operación; con -2 se lanza una excepción; con NULL la comparación no da TRUE y no entra al IF. Si las horas son obligatorias, hace falta la restricción correspondiente o un control explícito del nulo.

Una función común se invoca para obtener un resultado o ejecutar una operación. Un stored procedure se llama explícitamente con CALL. Si se usa un procedimiento para recalcular un resumen, el resumen se actualiza al llamarlo; un trigger lo mantiene ante los eventos declarados, sin depender de una llamada del usuario.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
