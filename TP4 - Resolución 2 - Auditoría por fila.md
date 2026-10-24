---
tags:
  - tp4
---

# TP4 - Resolución 2 - Auditoría por fila

El AFTER ROW se ejecuta una vez por cada fila efectivamente modificada. Cada entrada registra cantidad 1. La función puede compartirse porque TG_OP identifica la operación y TG_TABLE_NAME la tabla.

```sql
-- Alternativa ROW. Requiere 02_tabla.sql.
-- MAX + 1 se utiliza para las pruebas secuenciales del práctico.
CREATE OR REPLACE FUNCTION tp4_auditar_fila()
RETURNS trigger AS $$
DECLARE
    nuevo_nro NUMERIC(10,0);
BEGIN
    SELECT COALESCE(MAX(nro_registro), 0) + 1
    INTO nuevo_nro FROM his_entrega;

    INSERT INTO his_entrega (
        nro_registro, fecha, operacion,
        cant_reg_afectados, usuario, tabla_origen
    ) VALUES (
        nuevo_nro, CURRENT_TIMESTAMP, TG_OP,
        1, CURRENT_USER, TG_TABLE_NAME
    );
    RETURN NULL;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER tp4_auditar_entrega_row
AFTER INSERT OR UPDATE OR DELETE ON entrega
FOR EACH ROW
EXECUTE PROCEDURE tp4_auditar_fila();

CREATE TRIGGER tp4_auditar_renglon_row
AFTER INSERT OR UPDATE OR DELETE ON renglon_entrega
FOR EACH ROW
EXECUTE PROCEDURE tp4_auditar_fila();
```

El segundo trigger está correctamente asociado a **renglon_entrega**. En el borrador ambos escuchaban ENTREGA, lo que duplicaba su auditoría y no cubría cambios directos de renglones.

RETURN NULL es válido en AFTER: su retorno se ignora. Si una sentencia modifica cinco filas, se obtienen cinco entradas, no una entrada con cantidad cinco. Si no modifica ninguna, el cuerpo no se ejecuta.

Fuente: `Practico 4 - Acciones Procedurales.docx`, TP3 y esquema correspondiente de la cátedra. Se usa `EXECUTE PROCEDURE`.
