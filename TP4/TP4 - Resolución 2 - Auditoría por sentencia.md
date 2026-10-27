---
tags:
  - tp4
---

# TP4 - Resolución 2 - Auditoría por sentencia

La entrada de auditoría se genera una vez por sentencia y contiene cuántas filas se modificaron. Para contar sin secuencias ni tablas de transición, se separa el trabajo en tres pasos:

1. BEFORE STATEMENT prepara el contador correspondiente a tabla y operación en cero.
2. Un auxiliar AFTER ROW suma uno por cada fila modificada.
3. AFTER STATEMENT toma el total y escribe la única entrada de auditoría de la sentencia.

El auxiliar ROW **cuenta**, pero no inserta en HIS_ENTREGA. La auditoría visible conserva granularidad STATEMENT.

## Implementación completa, en orden de instalación

### 1 — Tabla de contadores, reinicio y acumulación

Esta parte crea seis contadores: uno por combinación de tabla y operación. La primera función pone a cero el correspondiente; la segunda suma una unidad por fila.

```mysql
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

### 2 — Función que registra el total de la sentencia

El conteo ya fue preparado y acumulado por los auxiliares. Esta función no adivina las filas afectadas: lee el contador de la tabla y operación que dispararon el trigger.

```mysql
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

### 3 — Triggers de ambas tablas

Cada tabla necesita preparar su conteo, acumular sus filas y registrar el resultado. Instalar estas cabeceras después de las tres funciones.

```mysql
CREATE TRIGGER tp4_iniciar_entrega
BEFORE INSERT OR UPDATE OR DELETE ON entrega
FOR EACH STATEMENT EXECUTE PROCEDURE tp4_iniciar_conteo();

CREATE TRIGGER tp4_contar_entrega
AFTER INSERT OR UPDATE OR DELETE ON entrega
FOR EACH ROW EXECUTE PROCEDURE tp4_contar_fila();

CREATE TRIGGER tp4_registrar_entrega
AFTER INSERT OR UPDATE OR DELETE ON entrega
FOR EACH STATEMENT EXECUTE PROCEDURE tp4_registrar_sentencia();

CREATE TRIGGER tp4_iniciar_renglon
BEFORE INSERT OR UPDATE OR DELETE ON renglon_entrega
FOR EACH STATEMENT EXECUTE PROCEDURE tp4_iniciar_conteo();

CREATE TRIGGER tp4_contar_renglon
AFTER INSERT OR UPDATE OR DELETE ON renglon_entrega
FOR EACH ROW EXECUTE PROCEDURE tp4_contar_fila();

CREATE TRIGGER tp4_registrar_renglon
AFTER INSERT OR UPDATE OR DELETE ON renglon_entrega
FOR EACH STATEMENT EXECUTE PROCEDURE tp4_registrar_sentencia();
```

Para una actualización de tres entregas: reinicio una vez, tres incrementos y un registro final con cantidad 3. Para cero entregas: reinicio, ningún incremento y un registro final con cantidad 0.

Se necesita previamente HIS_ENTREGA. El script completo, que reúne estas tres partes, es `02_statement.sql`.

Los contadores distinguen tabla y operación para que una baja de entrega y sus bajas de renglones no se mezclen. Esta implementación se valida para los eventos del TP, con pruebas secuenciales y sin triggers que vuelvan a modificar recursivamente la misma tabla auditada.

Fuente: `Practico 4 - Acciones Procedurales.docx`, TP3 y esquema correspondiente de la cátedra. Se usa `EXECUTE PROCEDURE`.
