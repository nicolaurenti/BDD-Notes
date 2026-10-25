---
tags:
  - tp4
---

# Auditoría, historial y resúmenes con triggers

Estos usos responden preguntas distintas. Una **auditoría** registra quién hizo una operación y cuándo. Un **historial** conserva estados o períodos anteriores. Una **tabla resumen** guarda cantidades o fechas calculadas a partir de otros datos. Elegir el propósito determina qué columnas y eventos hacen falta.

## Auditoría por fila

Una auditoría ROW registra una entrada por fila afectada. Debe identificar operación, fecha, usuario y tabla de origen; cada ejecución representa una fila.

### Ejemplo de auditoría por fila en HIS_ENTREGA

Definición de la tabla y un cuerpo compartido por los triggers de ENTREGA y RENGLON_ENTREGA.

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

## Qué cambia con STATEMENT

STATEMENT registra una actuación por sentencia. La cantidad de filas afectadas requiere un mecanismo de conteo adicional: no se obtiene cambiando la cabecera del trigger ROW.

### Auditoría STATEMENT y conteo de filas afectadas

Desarrollo del conteo global, las filas inexistentes y las bajas que se propagan por referencias.

Si una sentencia borra tres entregas, ROW registra tres filas para ENTREGA. STATEMENT ejecuta el cuerpo una vez para esa sentencia, incluso si no borró nada. Cambiar únicamente FOR EACH ROW por FOR EACH STATEMENT **no convierte el 1 del ejemplo en 3**.

COUNT(*) de ENTREGA después de borrar cuenta las filas que quedan, no las eliminadas. OLD y NEW tampoco representan todas las filas en STATEMENT. Para guardar una cantidad correcta hace falta diseñar un mecanismo adicional de conteo: preparar el conteo al comenzar la sentencia, acumular una unidad por cada fila afectada y emitir un único registro al final. Ese diseño debe distinguir tabla y sentencia; un contador compartido sin distinguirlas puede mezclar operaciones. El cambio de granularidad, por sí solo, solo resuelve la frecuencia de ejecución.

En el DELETE del ejercicio 2, el número concreto requiere conocer las entregas seleccionadas y la acción de la FK de sus renglones. Si hay baja en cascada, también se auditan las bajas efectivamente producidas en RENGLON_ENTREGA. No asumir una única fila de auditoría total para ambas tablas ni una cantidad fija sin revisar las referencias.

## Historial de permanencias

Un historial conserva intervalos y no solo el estado actual. Las duraciones de períodos abiertos crecen aunque no ocurran escrituras, por lo que conviene conservar fechas y definir cómo se calcula el promedio.

### Ejemplo de historial de permanencias

Caso de un empleado que vuelve a un departamento: períodos, tiempo acumulado y promedio por estancia.

Para saber cuánto tiempo estuvo un empleado en cada departamento no alcanza con almacenar su departamento actual. Se necesitan intervalos: empleado, distribuidor, departamento, fecha de inicio y fecha de fin. La clave de departamento es `(id_distribuidor, id_departamento)`.

Al ingresar un empleado se abre un intervalo; al cambiar de departamento se cierra el anterior y se abre otro; al salir se cierra el vigente. Si estuvo 10 días en D1, 20 en D2 y luego 30 en D1, la permanencia acumulada de D1 es 40 y el promedio por estancia allí es 20. «Promedio por departamento» necesita definir si promedia estancias o usa otra unidad.

Un período abierto aumenta con el tiempo aunque no se modifiquen filas. Guardar hoy una duración y esperar que un trigger de UPDATE la incremente mañana no funciona. Conservar fechas permite calcular la duración al consultar, o un procedimiento puede recalcularla cuando se lo invoca. También hay que decidir si el intervalo abierto participa en el promedio.

## Resúmenes y datos preexistentes

La carga inicial reconstruye el resumen de los datos existentes. Su mantenimiento posterior debe contemplar inserciones, bajas y cambios de autor o fecha; instalar un trigger no reconstruye el pasado.

### Ejemplo de carga y mantenimiento de TextosporAutor

Procedimiento de carga, grupos afectados por cada evento y caso de baja de la publicación más reciente.

#### Carga inicial y mantenimiento de TextosporAutor

El procedimiento de carga inicial actúa sobre lo que ya existe; los triggers mantienen los cambios futuros. Por ejemplo, sobre una tabla resumen creada con los tipos apropiados de ARTICULO:

```sql
CREATE OR REPLACE PROCEDURE cargar_textos_por_autor()
LANGUAGE 'plpgsql' AS $$
BEGIN
    DELETE FROM TextosporAutor;
    INSERT INTO TextosporAutor (autor, cant_textos, fecha_ultima_public)
    SELECT autor, COUNT(*), MAX(fecha_pub)
    FROM articulo
    GROUP BY autor;
END;
$$;

-- Ejemplo de invocación para el esquema de práctica:
CALL cargar_textos_por_autor();
```

El DELETE pertenece al algoritmo de reconstrucción de la **tabla derivada**: permite repetir la carga sin duplicarla. Aquí solo se escribe como ejemplo; no se ejecutó. Se supone que autor identifica los grupos y que se definió qué hacer con autores nulos.

Para el mantenimiento, identificar qué grupos cambian:

| Evento en ARTICULO | Grupos que deben recalcularse |
|---|---|
| INSERT | Autor nuevo |
| DELETE | Autor anterior |
| UPDATE de autor | Autor anterior y autor nuevo |
| UPDATE de fecha | Autor de la fila |

Si Ana tiene fechas 2019 y 2022 y se borra su artículo de 2022, el resumen debe pasar a cantidad 1 y fecha 2019. Restar uno al conteo sin revisar MAX dejaría una fecha falsa. Una estrategia sencilla es recalcular COUNT y MAX de los autores afectados; al quedar sin artículos, decidir si se quita su fila del resumen o se conserva con cero y fecha NULL.

El trigger no puede reconstruir fechas de ingresos o cambios que nunca se guardaron. Al incorporar un historial, distinguir datos conocidos de supuestos iniciales; no presentar la fecha de instalación como si fuera la de ingreso real de todos los empleados.

## Fuente de estudio

Enunciado del TP4 en la carpeta `../../Practicos` y filminas de cátedra `BDI_04 (SQL Procedural).pdf` (carpeta de cursada 2023).
