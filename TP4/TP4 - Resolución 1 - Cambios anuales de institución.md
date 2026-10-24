---
tags:
  - tp4
---

# TP4 - Resolución 1 - Cambios anuales de institución

Se implementa 5.e con un historial específico: una fila de HISTORICO puede representar un cambio de tarea dentro de la misma institución, por lo que contar sus filas no prueba cuántos cambios de institución hubo.

Se elige año calendario, identificador de voluntario estable y esta política: **cada cambio real de id_institucion cuenta, incluidos los pasos desde o hacia NULL; el alta inicial no cuenta**.

```sql
-- TP4, ejercicio 1: TP3 5.e. Año calendario; nro_voluntario se considera estable.
-- HISTORICO registra tareas/períodos, no necesariamente cambios de institución.
CREATE TABLE cambio_institucion (
    nro_voluntario NUMERIC(6,0) NOT NULL,
    fecha_cambio DATE NOT NULL,
    institucion_anterior NUMERIC(4,0),
    institucion_nueva NUMERIC(4,0)
);

CREATE OR REPLACE FUNCTION tp4_controlar_cambios_institucion()
RETURNS trigger AS $$
DECLARE
    cantidad INTEGER;
BEGIN
    IF NEW.id_institucion IS DISTINCT FROM OLD.id_institucion THEN
        SELECT COUNT(*) INTO cantidad
        FROM cambio_institucion
        WHERE nro_voluntario = OLD.nro_voluntario
          AND EXTRACT(YEAR FROM fecha_cambio)
              = EXTRACT(YEAR FROM CURRENT_DATE);

        IF cantidad >= 3 THEN
            RAISE EXCEPTION 'RI_5E: ya realizo tres cambios este ano';
        END IF;

        INSERT INTO cambio_institucion (
            nro_voluntario, fecha_cambio,
            institucion_anterior, institucion_nueva
        ) VALUES (
            NEW.nro_voluntario, CURRENT_DATE,
            OLD.id_institucion, NEW.id_institucion
        );
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER tp4_cambios_institucion
AFTER UPDATE OF id_institucion ON voluntario
FOR EACH ROW
EXECUTE PROCEDURE tp4_controlar_cambios_institucion();
```

Con cero, uno o dos cambios previos se permite otro y se registra. Con tres previos se rechaza el cuarto. Reasignar el mismo valor no registra nada.

La tabla auxiliar no lleva un identificador generado: cada fila representa un evento y COUNT(*) cuenta eventos, aunque dos tengan la misma fecha. Si una comprobación posterior rechaza la sentencia, su modificación y el registro auxiliar se revierten juntos.

Al incorporar el control a datos antiguos, deben cargarse los cambios reales del año que ya se conozcan. Un historial inicialmente vacío no demuestra que nadie haya cambiado antes; los cambios que nunca se registraron no se pueden reconstruir por instalar el trigger.

[[TP4 - Resolución 1 - Pruebas de cambios de institución|Pruebas de tercero, cuarto, mismo valor y cambio de año]].

Fuente: `Practico 4 - Acciones Procedurales.docx`, TP3 y esquema correspondiente de la cátedra. Se usa `EXECUTE PROCEDURE`.
