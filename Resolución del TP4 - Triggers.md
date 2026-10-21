---
tags:
  - tp4
---
EJ1
EJ2
a)
```mysql

/* Creación de tabla */
CREATE TABLE his_entrega (
    nro_registro NUMERIC(4,0),
    fecha DATE NOT NULL,
    operacion VARCHAR(120) NOT NULL,
    cant_reg_afectados NUMERIC(4,0) NOT NULL,
    usuario VARCHAR(120) NOT NULL,

    CONSTRAINT pk_his_entrega
        PRIMARY KEY (nro_registro)
);

/* Trigger entrega */
CREATE OR REPLACE FUNCTION mantenimiento_his_entrega_entr()
RETURNS trigger AS $$
DECLARE
    nuevo_nro NUMERIC(4,0);
BEGIN
    SELECT COALESCE(MAX(nro_registro), 0) + 1
    INTO nuevo_nro
    FROM his_entrega;

    INSERT INTO his_entrega (
        nro_registro,
        fecha,
        operacion,
        cant_reg_afectados,
        usuario
    )
    VALUES (
        nuevo_nro,
        CURRENT_DATE,
        TG_OP,
        1,
        CURRENT_USER
    );

    RETURN NULL;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER tr_mantenimiento_his_entrega_entr
AFTER INSERT OR UPDATE OR DELETE ON entrega
FOR EACH ROW
EXECUTE PROCEDURE mantenimiento_his_entrega_entr();

/* Trigger renglon_entrega */
CREATE OR REPLACE FUNCTION mantenimiento_his_entrega_renglon_entr()
RETURNS trigger AS $$
DECLARE
    nuevo_nro NUMERIC(4,0);
BEGIN
    SELECT COALESCE(MAX(nro_registro), 0) + 1
    INTO nuevo_nro
    FROM his_entrega;

    INSERT INTO his_entrega (
        nro_registro,
        fecha,
        operacion,
        cant_reg_afectados,
        usuario
    )
    VALUES (
        nuevo_nro,
        CURRENT_DATE,
        TG_OP,
        1,
        CURRENT_USER
    );

    RETURN NULL;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER tr_mantenimiento_his_entrega_renglon_entr
AFTER INSERT OR UPDATE OR DELETE ON entrega
FOR EACH ROW
EXECUTE PROCEDURE mantenimiento_his_entrega_renglon_entr();
```
b)

EJ3
EJ4
EJ5
EJ6
# Resolución del TP4 — Triggers

## Resolución

Pendiente de completar.

## Material

- Enunciado: `../Practicos/Practico 4 - Acciones Procedurales.docx`.
