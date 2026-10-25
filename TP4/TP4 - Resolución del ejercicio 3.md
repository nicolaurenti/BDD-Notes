---
tags:
  - tp4
---

# TP4 - Resolución del ejercicio 3

El nombre autoincremento no describe el cuerpo: el UPDATE **resta el 5 % del sueldo mínimo** a las filas que ya están en EMPLEADO_1.

Se toma el sueldo como NUMERIC(10,2), un empleado inicial de id 100 con sueldo 500 y las inserciones de id 2/700, id 3/300 e id 4/700, en ese orden.

| id_empleado | BEFORE ROW | BEFORE STATEMENT |
|---:|---:|---:|
| 100 | 436.25 | 475.00 |
| 2 | 661.25 | 700.00 |
| 3 | 285.00 | 300.00 |
| 4 | 700.00 | 700.00 |

## Desarrollo

### Correcciones sintácticas, retorno y selección de granularidad

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

### Seguimiento del mínimo, descuentos y estado después de cada paso

Se supone NUMERIC(10,2) y orden de inserción 700, 300, 700. Antes de comenzar, solo existe id 100 con sueldo 500.

#### BEFORE ROW

| Paso | Mínimo antes del trigger | Descuento a cada fila ya presente | Estado después del UPDATE y de insertar la nueva fila |
|---|---:|---:|---|
| Insertar id 2 con 700 | 500 | 25 | 100: 475; 2: 700 |
| Insertar id 3 con 300 | 475 | 23.75 | 100: 451.25; 2: 676.25; 3: 300 |
| Insertar id 4 con 700 | 300 | 15 | 100: 436.25; 2: 661.25; 3: 285; 4: 700 |

Primer paso: 5 % de 500 es 25. Solo se descuenta al empleado inicial porque id 2 todavía no fue insertado.

Segundo paso: el mínimo es 475, no 300. El sueldo 300 pertenece a la fila que se insertará después del BEFORE. El descuento 23.75 afecta a los empleados 100 y 2.

Tercer paso: ahora sí existe el sueldo 300 y es el mínimo. El descuento 15 afecta a 100, 2 y 3. Id 4 se incorpora luego y conserva 700.

#### BEFORE STATEMENT

El mínimo inicial es 500. Hay una única ejecución del cuerpo, que descuenta 25 al empleado 100 y lo deja en 475. Después se insertan los tres nuevos con sus valores originales: 700, 300 y 700.

No se recalcula un descuento por cada fila en STATEMENT. Si el tipo de sueldo fuera entero o cambiara el orden de inserción, habría que revisar la simulación; no se trasladan estos resultados decimales sin declarar los supuestos.

En ROW, cada fila insertada dispara antes su propia ejecución; la nueva fila todavía no está en la tabla cuando se aplica ese descuento. En STATEMENT, el cuerpo se ejecuta una sola vez antes de las tres inserciones.

Los resultados se verificaron ejecutando ambas variantes en esquemas temporales separados. Un INSERT SELECT sin orden no permite asegurar ese recorrido de filas; el caso trabajado explicita el orden esperado por la simulación.

Fuente: `Practico 4 - Acciones Procedurales.docx`, TP3 y esquema correspondiente de la cátedra. Se usa `EXECUTE PROCEDURE`.
