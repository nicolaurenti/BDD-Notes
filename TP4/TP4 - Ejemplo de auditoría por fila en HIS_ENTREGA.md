---
tags:
  - tp4
---

# TP4 - Ejemplo de auditoría por fila en HIS_ENTREGA

Definición de la tabla y un cuerpo compartido por los triggers de ENTREGA y RENGLON_ENTREGA.

## Auditoría por fila

Para HIS_ENTREGA, el TP pide al menos número de registro, fecha, operación, cantidad de registros afectados y usuario. Agregar la tabla de origen permite distinguir ENTREGA de RENGLON_ENTREGA.

```sql
CREATE TABLE his_entrega (
    nro_registro SERIAL PRIMARY KEY,
    fecha TIMESTAMP NOT NULL,
    operacion VARCHAR(10) NOT NULL,
    cant_reg_afectados INTEGER NOT NULL,
    usuario VARCHAR(100) NOT NULL,
    tabla_origen VARCHAR(40) NOT NULL
);

CREATE OR REPLACE FUNCTION auditar_entrega_fila()
RETURNS trigger AS $$
BEGIN
    INSERT INTO his_entrega
        (fecha, operacion, cant_reg_afectados, usuario, tabla_origen)
    VALUES
        (CURRENT_TIMESTAMP, TG_OP, 1, CURRENT_USER, TG_TABLE_NAME);
    RETURN NULL;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER tr_auditar_entrega
AFTER INSERT OR UPDATE OR DELETE ON entrega
FOR EACH ROW
EXECUTE PROCEDURE auditar_entrega_fila();

CREATE TRIGGER tr_auditar_renglon
AFTER INSERT OR UPDATE OR DELETE ON renglon_entrega
FOR EACH ROW
EXECUTE PROCEDURE auditar_entrega_fila();
```

SERIAL genera el identificador al omitirlo del INSERT; no expresa cuántas filas afectó la operación. Cada ejecución de este cuerpo registra cantidad 1 porque su granularidad es ROW. El retorno se ignora por ser AFTER.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
