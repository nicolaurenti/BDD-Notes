---
tags:
  - tp4
---

# TP4 - Resolución 2 - Contador y funciones auxiliares

Esta parte crea seis contadores: uno por combinación de tabla y operación. La primera función pone a cero el correspondiente; la segunda suma una unidad por fila.

```sql
-- Alternativa STATEMENT. Requiere 02_tabla.sql. No instalar junto a 02_row.sql.
-- Cada tabla/evento tiene un contador. Se prepara, acumula y registra.
-- Supuestos del TP: pruebas secuenciales, sin recursión sobre la misma tabla.
CREATE TABLE contador_his_entrega (
    tabla_origen VARCHAR(40),
    operacion VARCHAR(10),
    cantidad NUMERIC(10,0) NOT NULL,
    PRIMARY KEY (tabla_origen, operacion)
);

INSERT INTO contador_his_entrega VALUES
('entrega', 'INSERT', 0), ('entrega', 'UPDATE', 0), ('entrega', 'DELETE', 0),
('renglon_entrega', 'INSERT', 0), ('renglon_entrega', 'UPDATE', 0),
('renglon_entrega', 'DELETE', 0);

CREATE OR REPLACE FUNCTION tp4_iniciar_conteo()
RETURNS trigger AS $$
BEGIN
    UPDATE contador_his_entrega SET cantidad = 0
    WHERE tabla_origen = TG_TABLE_NAME AND operacion = TG_OP;
    RETURN NULL;
END;
$$ LANGUAGE 'plpgsql';

CREATE OR REPLACE FUNCTION tp4_contar_fila()
RETURNS trigger AS $$
BEGIN
    UPDATE contador_his_entrega SET cantidad = cantidad + 1
    WHERE tabla_origen = TG_TABLE_NAME AND operacion = TG_OP;
    RETURN NULL;
END;
$$ LANGUAGE 'plpgsql';
```

Las dos funciones devuelven NULL porque se usarán en BEFORE STATEMENT y AFTER ROW respectivamente. En esos casos el retorno no reemplaza una fila propuesta. Los contadores se cargan una sola vez al instalar la alternativa.
