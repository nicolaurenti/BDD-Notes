---
tags:
  - tp4
---

# Fundamentos y ejecución de triggers

Un trigger asocia un evento de modificación con código que el motor ejecuta automáticamente. En este práctico se usa para expresar controles que no pudieron implementarse declarativamente en el TP3 y para mantener auditorías e información derivada. Una regla declarativa expresa el estado permitido; el trigger debe decidir **cuándo comprobarlo y qué hacer**.

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

## Evento, momento y granularidad

El **evento** puede ser INSERT, UPDATE o DELETE. El **momento** indica si el cuerpo actúa antes o después de la modificación. La **granularidad** indica cuántas veces se ejecuta.

| Decisión | Cuándo elegirla en estos ejercicios |
|---|---|
| BEFORE ROW | Revisar la fila propuesta o corregir NEW antes de guardarla |
| AFTER ROW | Registrar una modificación realizada o mantener otro dato a partir de esa fila |
| BEFORE STATEMENT | Preparar una acción para toda la sentencia |
| AFTER STATEMENT | Registrar o comprobar el efecto global de la sentencia |

Si un UPDATE afecta tres filas, ROW ejecuta el cuerpo tres veces y STATEMENT una. Si afecta cero, ROW no tiene filas para ejecutar el cuerpo, pero STATEMENT sí se dispara. Por eso «una ejecución del trigger» no significa siempre «una fila modificada».

Para simular, ubicar primero BEFORE STATEMENT; luego las actuaciones por cada fila, considerando los BEFORE ROW antes de la operación de esa fila; y las acciones AFTER correspondientes. AFTER significa posterior a la modificación, no una nueva sentencia del usuario ni una confirmación independiente.

## OLD, NEW y RETURN

| Evento ROW | OLD | NEW |
|---|---|---|
| INSERT | No hay fila anterior | Fila a incorporar |
| UPDATE | Valores anteriores | Valores propuestos |
| DELETE | Fila a eliminar | No hay fila nueva |

En BEFORE INSERT/UPDATE, RETURN NEW permite continuar usando la fila propuesta, incluidas las modificaciones que hizo la función. En BEFORE DELETE se devuelve OLD para permitir la baja. RETURN NULL en BEFORE ROW omite esa fila: **no equivale a rechazar la sentencia con un error**. Para una infracción se usa RAISE EXCEPTION.

En AFTER y en triggers de sentencia el valor devuelto se ignora; se puede escribir RETURN NULL. Un trigger STATEMENT no tiene una única fila OLD o NEW que represente todo el conjunto.

`TG_OP` informa la operación; `TG_TABLE_NAME`, la tabla; `TG_WHEN`, el momento; y `TG_LEVEL`, la granularidad. Sirven para compartir un cuerpo entre varios eventos o identificar qué se está auditando.

## Simulación ROW y STATEMENT del ejercicio de sueldos

El cuerpo del ejercicio resta a los empleados existentes el 5 % del sueldo mínimo. Aunque se llame autoincremento, la operación es una resta. Para interpretar código ejecutable, falta el punto y coma del UPDATE y el retorno debe ajustarse: NEW para BEFORE ROW de INSERT; NULL para STATEMENT.

Suponiendo sueldos decimales y el orden de inserción 700, 300, 700, el estado se sigue así:

| Paso BEFORE ROW | Mínimo consultado | Descuento | Sueldos después del trigger y de insertar la fila |
|---|---:|---:|---|
| Insertar id 2, sueldo 700 | 500 | 25 | id 100: 475; id 2: 700 |
| Insertar id 3, sueldo 300 | 475 | 23.75 | id 100: 451.25; id 2: 676.25; id 3: 300 |
| Insertar id 4, sueldo 700 | 300 | 15 | id 100: 436.25; id 2: 661.25; id 3: 285; id 4: 700 |

La fila nueva todavía no está en la tabla cuando corre su BEFORE ROW, por eso no recibe el descuento de su propio paso. Sí puede recibir los descuentos de pasos posteriores.

En BEFORE STATEMENT el cuerpo se ejecuta una sola vez antes de insertar las tres filas: el inicial pasa de 500 a 475 y los nuevos quedan en 700, 300 y 700. Para el cálculo en papel se declara el orden supuesto; no se lo presenta como un orden garantizado de cualquier INSERT SELECT.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
