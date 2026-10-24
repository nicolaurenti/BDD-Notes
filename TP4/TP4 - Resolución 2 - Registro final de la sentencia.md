---
tags:
  - tp4
---

# TP4 - Resolución 2 - Registro final de la sentencia

El conteo ya fue preparado y acumulado por los auxiliares. Esta función no adivina las filas afectadas: lee el contador de la tabla y operación que dispararon el trigger.

```sql
CREATE OR REPLACE FUNCTION tp4_registrar_sentencia()
RETURNS trigger AS $$
DECLARE
    nuevo_nro NUMERIC(10,0);
    cantidad_filas NUMERIC(10,0);
BEGIN
    SELECT cantidad INTO cantidad_filas FROM contador_his_entrega
    WHERE tabla_origen = TG_TABLE_NAME AND operacion = TG_OP;

    SELECT COALESCE(MAX(nro_registro), 0) + 1 INTO nuevo_nro
    FROM his_entrega;

    INSERT INTO his_entrega (
        nro_registro, fecha, operacion,
        cant_reg_afectados, usuario, tabla_origen
    ) VALUES (
        nuevo_nro, CURRENT_TIMESTAMP, TG_OP,
        cantidad_filas, CURRENT_USER, TG_TABLE_NAME
    );
    RETURN NULL;
END;
$$ LANGUAGE 'plpgsql';
```

Si no hubo filas, cantidad_filas vale cero y se registra igualmente la sentencia. MAX + 1 asigna el identificador bajo el supuesto de pruebas secuenciales.
